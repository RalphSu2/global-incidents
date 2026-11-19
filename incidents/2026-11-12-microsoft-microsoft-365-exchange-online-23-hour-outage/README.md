# Microsoft — Microsoft 365 / Exchange Online 级联故障（23 小时 50 分）

> **分级：A 级** · 厂商发布了《Preliminary Post Incident Report》
> ⚠️ **PIR 原文只存在于 Microsoft 365 管理中心的租户内**，公开渠道仅有 Copilot/Agents/Platform 执行副总裁 Charles Lamanna 的摘要帖。本条目 `raw/` 归档的 PDF 是**第三方公开镜像**，非厂商原始发布渠道，引用时请注意这一点。

## 一句话结论

一个 7 月就已埋入、全球跑了三个多月都没暴露的 Windows 锁竞争缺陷，因为 Teams 自动 failover 把流量压到共用网关、叠加 14% 网关容量当时已下线，在不到两小时内把 **80% 以上的服务器打成无响应** —— 而厂商的隔离机制因另一个独立问题未能阻止级联。

---

## 时间线（UTC，2026 年）

| UTC | 事件 |
|---|---|
| 11-11 **01:07** | Teams 自动 failover 到与 Exchange Online 共用的 primary gateway，流量模式改变（故障前约 15 小时） |
| 11-12 **16:03** | 监控开始告警 |
| **16:20** | **故障开始**，事件 `EX1550543` / `MO1550821` |
| ~17:00 | 缺陷在机群内扩散，**超过 80% 服务器无响应** |
| **20:00** | 绕过健康检查，恢复部分流量接纳能力 |
| 11-13 **01:53** | 关闭自动重定向 |
| **07:20** | 回滚应用更新 |
| **16:10** | **事件结束**（累计 23 小时 50 分） |
| 11-14 **04:15** | 最终客户通知 |

## 根因（厂商口径）

三因素叠加：

**① 潜伏缺陷（2026 年 7 月埋入）**
> *"a latent defect caused Windows servers to become overwhelmed and unresponsive under certain load conditions. This was due to **lock contention inside an internal Windows process** which caused the machine's resources (CPU & memory) to be exhausted."*

关键细节 —— **测试环境比生产环境弱，因此漏掉了**：
> *"The contention issue is **worsened on machines with more cores and newer hardware**; whereas our load testing tended to focus on **weaker hardware with fewer cores**."*

**② 负载激增** — Teams 的自动 failover 把流量导向共用网关，改变了流量模式。

**③ 容量缺口** — 约 **14% 的网关服务器容量**因前一天的无关问题已下线。

**级联机制（原文）**
> *"the redirected traffic increased the likelihood for the same problem to occur on the remaining endpoints. This caused a **cascading failure from server to server** as the same amount of load was spread across fewer servers. In less than two hours, the previously unknown defect was encountered across our fleet... **more than 80% of servers were unresponsive.**"*

**隔离机制失效（厂商承认但未解释）**
> *"due to **a separate issue**, the normal bulkheads between these different partitions did not stop the cascading failure."*

**为什么恢复这么慢** —— 三点都值得记：
1. 服务器一旦重新接入流量就再次被打爆
2. 自动化恢复流程**不是为"大面积无响应"设计的**
3. 缺陷是几个月前埋入的，**rollback、flighting、feature flag 全部回不去**

## 技术要点

> 解读：以下为我的分析，非厂商原文。

**1. "测试机比生产机弱"是最经典的漏网姿势**
报告自己写明了：缺陷在核数更多、硬件更新的机器上更严重，而负载测试集中在核数更少的老弱硬件上。**容量与缺陷都要在"最强"而非"最弱"的硬件上验证。**

**2. 弹性机制本身成为故障通路**
Teams 的 failover 本是保护手段，却把冲击转移到了共用组件的薄弱面。这是"correlated fragility"：**被保护对象没事，保护它的那层倒了。**

**3. 隔离机制失效却没有解释**
报告承认 bulkhead 因"另一个独立问题"未生效，但**没有说明那个问题是什么**。这是本 PIR 最大的信息缺口。

**4. 幽灵缺陷的不可回滚性**
当缺陷已在生产环境静默运行数月，所有常规止血手段（回滚、开关、灰度和 flighting）全部失效。**唯一选择是硬扛到修复。**

## 与我们相关的启示

> 解读：以下为我的分析。

**① 长潜伏期缺陷要有独立的探测手段** —— 不能只依赖"变更后观察"。可考虑在高峰负载下做合成压测，主动触发核数敏感类缺陷。

**② 容量水位要按"最坏同时发生"算** —— 14% 容量下线 + 突发 failover 是可以在计划内预演的组合。

**③ 恢复流程要为"大面积无响应"单独设计** —— 常规自动化恢复假定服务器"活着但慢"，不适用于服务器直接不回话的场景。

**④ 要求厂商补全 bulkhead 失效说明** —— 这是本次唯一没交代清楚的环节。

## 原始报告

- **公开摘要**：https://x.com/clamanna/status/2122209697226948729
- **PIR 正文**：Microsoft 365 Admin Center → Service health → `EX1550543`（需租户权限）
- **本地归档**：`raw/preliminary-pir-thirdparty-mirror.pdf`（第三方镜像，SHA256 见 `sources.md`）
- **最终版 PIR**：厂商承诺 2026-11-20 发布，届时应复查补充
