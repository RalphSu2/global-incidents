# AWS — us-east-1 认证失败引发 11 项服务连锁降级

> **分级：C 级** · **厂商未发布技术报告**。AWS 的 Post-Event Summary (PES) 索引页上，**最新一条仍停留在 2025-10-19**，2026 年至今一条都没有。

## 一句话结论

us-east-1 的认证链路失效约 2 小时 15 分且**没有提供任何 workaround**，连带打掉控制台、Kiro、Connect、CloudWatch 等 11 项服务；随后又有 1 小时的单 cell 负载均衡残留问题。**AWS 至今未披露原因。**

---

## 时间线（UTC，2026-11-15）

| UTC | 阶段 | 内容 |
|---|---|---|
| **11:05** | 阶段一 | 认证失败开始，约 **2 小时 15 分**，**无 workaround** |
| ~13:20 | 阶段二 | Network Load Balancer 在 `use1-az1` 单个 cell 出现残留连接问题，约 **1 小时 01 分** |
| **14:21** | 结束 | 全部恢复（累计 196 分钟） |

## 受影响服务（11 项）

IAM Identity Center、Sign Up、Kiro、Amazon Connect、CloudWatch、NAT Gateway、Elastic Load Balancing、VPC Lattice、OpenSearch、Step Functions、IoT Device Management

## 根因（官方口径）

**厂商未披露原因，未发布 PES。**

AWS 的 PES 页面在采集时（见 `raw/aws-pes-index-snapshot.html`）最新条目为 **2025-10-19 DynamoDB 事件**，2026 全年零条。也就是说：**AWS 今年所有重大故障都没有出报告。**

## 技术要点

> 解读：以下为我的分析，**厂商未确认**。

**1. 认证是云上的单点**
IAM Identity Center 一挂，控制台、Kiro、Connect 全部不可用 —— 因为它们都通过同一认证网关。**能干活的人和监控系统一起下线**，这是最危险的一类故障。

**2. "无 workaround" 是最刺眼的表述**
持续 2 小时 15 分没有任何绕过方案，意味着连 root 账号应急通道都没生效，或客户根本不知道该怎么办。**灾备方案里"认证不可用"这一格通常是空的。**

**3. 阶段二暴露了第二个独立缺陷**
认证恢复后 NLB 在单个 cell 留下残留连接问题，说明存在两个互不相关的故障点，加剧了恢复时长。

**4. 零报告本身就是结论**
AWS 2026 年零 PES，监管与客户都无法做技术复盘。**把它当作采购决策中的一个风险项是合理的。**

## 与我们相关的启示

> 解读：以下为我的分析。

**① 不要让 IAM 成为唯一入口** —— 关键运维路径应有独立的、不依赖主认证链路的应急通道（break-glass 账号、独立 MFA 路径）。

**② 监控要能脱离被监控对象工作** —— CloudWatch 自己也在受影响列表里。

**③ 多 cell / 多 AZ 不等于无单点** —— NLB 单 cell 残留问题说明控制面与数据面的可用性不是一回事。

## 原始报告

- **PES 索引**：https://aws.amazon.com/premiumsupport/technology/pes/（最新条目 2025-10-19，无 2026 年报告）
- **健康面板**：https://status.aws.amazon.com/rss/all.rss
- **本地归档**：`raw/aws-pes-index-snapshot.html`（采集时点快照，用于证明"无 PES"这一事实）
- **结论**：❌ **无厂商技术报告，根因未披露**
