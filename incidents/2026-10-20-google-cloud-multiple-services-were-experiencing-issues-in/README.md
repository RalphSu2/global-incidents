# Google Cloud — europe-west2（伦敦）单可用区断电，38 个产品受影响

> **分级：A 级** · Google Cloud 官方 Incident Report，含完整根因、分钟级时间线、整改清单
> 原始报告归档：[`raw/official-report.html`](raw/official-report.html)

## 一句话结论

一次外部电缆故障打掉了一路市电，而 PLC 的欠压传感点被接在**断路器上游**，导致它误判"还有电"、拒绝启动柴油发电机 —— 冗余 2N 架构因一次"临时 1N 维护姿态"彻底失效，UPS 5 分钟耗尽后整个可用区断电。

---

## 时间线（UTC，2026 年）

| UTC | 事件 |
|---|---|
| 10-20 **05:53** | 市电 feeder A 因**数据中心外部**地下电缆故障停电；PLC 故障，发电机未接管负载，UPS 电池开始放电 |
| **06:00** | **影响开始** — 电池耗尽，`europe-west2-b` 的主机、本地存储、网络汇聚设备断电；实例、数据库副本、worker 节点立即不可达 |
| **06:04** | 自动监控与探测器告警：API 错误率上升、作业提交超时、基础设施不可达 |
| **06:09** | 区域流量路由与客户端触发自动 failover 到同区域其它 AZ；跨区 RPC 超时，其它 AZ 容量被打满，全区间歇性 500/503 |
| **06:54** | 备用市电（standby utility source）恢复供电 |
| **07:20** | 数据厅市电恢复，开始重置并测试发电机自动化控制器 |
| **07:40** | 集群开始恢复，集群管理器重新调度计算作业 |
| **07:45** | 网络设备全部重新上线 |
| **07:50** | 存储设备因非正常关机开始恢复。**响应团队主动选择不恢复部分服务**，以免情况恶化 |
| **10:13** | 电源冗余验证通过，存储完整性达标，准备受控回流量 |
| **10:45** | 执行渐进式受控流量回注，清理积压作业 |
| **11:11** | GCP 服务与 API 全量可用；部分 GCE 虚拟机仍未恢复 |
| **11:29** | **核心平台级中断结束**（区域服务与后端计算管理完全排空） |
| **12:08** | **99.5% GCE 虚拟机恢复**，绝大多数客户工作负载恢复正常 |

**总时长 6 小时 08 分**（06:00 → 12:08）；其中核心平台中断 **5 小时 29 分**（06:00 → 11:29）。

---

## 根因（厂商口径）

### 三层叠加，缺一不成灾

**第一层 —— 临时 1N 维护姿态（前置条件）**

> 原文：*"Prior to the utility disruption, the facility had been placed in a temporary single-feed (1N) bridged maintenance posture on the primary utility. Circuit breaker feeder B was physically opened to allow for corrective repairs following issues identified during a quarterly generator load test on 14 Oct 2026."*

10 月 14 日季度发电机带载测试发现异常，为做修复，**物理断开了 feeder B 的断路器**。设施因此从 2N（全冗余双路）退化为 **1N（单路）**，feeder A 独自承担全部关键负载。

**第二层 —— 外部电缆故障（触发条件）**

> 原文：*"the primary utility provider experienced an outage on primary utility feed A, caused by an external underground cable fault on the utility network outside the data center facility."*

唯一在役的 feeder A 被外部电缆故障打掉。这本身不在 Google 责任范围内，但冗余已被第一层消解。

**第三层 —— PLC 传感点接错，这才是真正的缺陷**

> 原文：*"The investigation of the PLC configuration identified the root cause as a misconfiguration of the PLC logic that did not verify both an undervoltage condition and physical breaker position for both utility feeders. Instead, the PLC undervoltage sensing inputs were **wired upstream (line side) of the isolated utility feed breakers**."*

PLC 的判断逻辑要求同时确认"欠压"**和**"物理断路器位置"，但**欠压传感输入被接在断路器的上游（line side）**。B 的断路器虽然断开，上游依然带电 → **PLC 看到"有电"，判定无需启动发电机**。

> 原文：*"the system inhibited the action due to the live upstream voltage reading on feed B."*

更糟的是，现场团队尝试远程带载切换时，**PLC 同样以"feed B 上游有电"为由拒绝了操作**。团队判断 UPS 撑不到远程操作解决，转而把负载切到备用市电，才止住失电。

**因果链完整版：**

```
10/14 发电机测试异常 → 断开 feeder B 断路器维修 → 设施降为 1N
        ↓
10/20 05:53 feeder A 外部电缆故障 → 唯一在役市电失效
        ↓
PLC 传感点在上游 → 误判"B 路有电" → 不发发电机启动信号
        ↓
UPS 撑约 5 分钟 → 06:00 电池耗尽 → 整个 -b 区断电
        ↓
自动 failover → 其它 AZ 容量饱和 → 全区 500/503
        ↓
存储非正常关机 → 恢复慢且不可加速 → 12:08 才恢复 99.5%
```

---

## 技术要点

**1. 传感点与被控对象不一致，是设施自动化里的经典致命缺陷**

PLC 测的是**线侧电压**，控制的却是**母线供电**。中间隔着一个断开的断路器，测量值就完全失真。这类缺陷不会在日常自检中暴露 —— 只要双路都在，读数就是对的。

> 解读：这是"监控指标正确但语义错误"的极端案例。仪表没坏、读数没假，但**它回答的不是那个问题**。

**2. 维护姿态把 2N 架构的保护静默抹掉了**

2N 意味着每条关键负载有两条独立供电路径 —— 但这个保证在"为维修而断开一路"时不成立。Google 的整改项专门有一条印证了这层反思：

> *"Freeze planned maintenance activities at other sites with similar topologies, and implement elevated approvals and reviews for data center configuration changes which are outside their standard designs."* `[已完成 10-30]`

**3. 自动化保护机制反向阻断了人工干预**

PLC 不但没启动发电机，还拒绝了人工的 load transfer。两层保护叠加成双重锁死 —— 现场的"最后一道人工兜底"被自动化本身吃掉。

**4. 存储恢复时间是硬约束，压不动**

Google 明确写了为什么慢：

> *"Responders recognized this and chose **not to restore services that would otherwise make this worse**. Because of the large volume of storage devices involved, this process required a substantial period to finish."*

非正常关机会在存储层留下未完成事务与元数据不一致。**贸然拉起服务会造成数据损坏**，所以只能等。这决定了 RTO 的下限。

**5. 单 AZ 故障如何变成全区故障**

- 跨区 RPC 指向已下线 AZ 的元数据/目录组件 → 超时
- 重定向流量涌入 → 其它 AZ 计算槽位与数据库 failover 能力饱和
- 排队积压 → 数据库 failover 完成延迟 → 间歇 500/503

Google 的处置顺序很关键：先**流量隔离**（把路由从 -b 挪走，阻止请求继续打向离线端点），再**工作负载重新配额**（优先保障存量客户预留）。不先隔离，重试风暴会持续消耗其它 AZ 的容量。

---

## 与我们相关的启示

> 解读：以下是我的分析，非厂商原文。

**① "临时"配置必须当正式变更管**

1N 维护姿态是"临时的"，为期 6 天，却没有任何机制强制提醒"你正在无冗余运行"。建议给自建系统的降级姿态打上**带 TTL 的强制标记**，超时自动升级告警。

**② 灾备演练要在"降级姿态"下做**

如果 Google 在这次 1N 窗口内做过一次发电机启动测试，这个缺陷当场就会暴露。常规演练默认满配，恰恰绕开了最危险的时段。

**③ 传感/探活点必须与被控对象同层**

这条适用于所有自动化系统：探活探的是 VIP 还是后端实例？健康检查查的是进程存活还是真实业务链路？**测量层与控制层不一致的系统，在故障时一定给你错误答案。**

**④ RTO 承诺要给存储恢复留出空间**

如果对外承诺"30 分钟恢复"，但存储层非正常关机需要 4 小时，那承诺本身就是错的。恢复时间要按**最慢的那一层**算，不是最快的。

**⑤ 人工兜底路径要独立于自动化**

当 PLC 拒绝人工操作时，现场没有任何 bypass。**任何自动化保护都必须存在一条手动旁路**，并且这条旁路要定期演练。

---

## 原始报告

- **官方链接**：https://status.cloud.google.com/incidents/Xs2XTHzAVC9YnafnPyzj
- **报告类型**：Google Cloud Incident Report（厂商官方，含 Root Cause / Remediation / Detailed Impact / Action Items）
- **本地归档**：`raw/official-report.html`（SHA256 `a80c249fd713b492…`，完整校验值见 [`sources.md`](sources.md)）
- **受影响产品**：38 个，含 Compute Engine、Persistent Disk / Hyperdisk、GKE、Cloud SQL、AlloyDB、Bigtable、Cloud Batch、Cloud Build、Cloud Storage、Cloud Workstations、Composer、Dataproc、Filestore、Managed Lustre、Migrate to VMs、NetApp Volumes、Pub/Sub Lite、Vertex AI Workbench、VMware Engine 等（完整清单见 [`facts.md`](facts.md)）

**整改项（部分已完成）**

| 项目 | 状态 |
|---|---|
| 修复电缆故障点（低频雷达探测） | 已完成 10-26 |
| 修复 feeder A 电缆 | 预计 11-13 |
| PLC 配置修复与验证 | 已完成 10-26 |
| 全球同类拓扑站点 PLC 配置校验 | 预计 11-15 |
| EOP（应急操作规程）更新，每班演练两次 | 已完成 10-26 |
| 冻结同类站点计划性维护 + 变更审批升级 | 已完成 10-30 |
| PLC 代码评审纳入投运流程（FR 前 + 投运后一季度复核） | 预计 12-18 |
