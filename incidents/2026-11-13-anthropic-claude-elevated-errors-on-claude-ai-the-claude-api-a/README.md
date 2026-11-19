# Anthropic (Claude) — claude.ai / Claude API / Claude Code 大面积报错

> **分级：B 级** · 仅厂商状态页事故记录，**厂商未发布技术复盘**
> 原始报告归档：[`raw/official-report.html`](raw/official-report.html)

## 一句话结论

claude.ai（含移动端）、Claude API、Claude Code 与连接器、Managed Agents、Claude Security 同时降级，持续 **2 小时 29 分**；受影响面横跨推理端点、代码工具和第三方连接器，指向共享的依赖层而非单一服务，但 **Anthropic 全程未披露任何根因**。

---

## 时间线（UTC，2026-11-13）

依据状态页更新序列还原。**厂商未给出每次更新的精确时间戳**，起点与终点取自状态页首末条更新。

| UTC | 状态 | 内容 |
|---|---|---|
| **13:09** | Investigating | 开始调查 claude.ai、Claude API、Claude Code 的性能降级 |
| — | Investigating（扩大） | 范围扩大到连接器（Claude Docs、Claude Sheets）、Claude Security、Claude Managed Agents |
| — | 部分恢复 | claude.ai 与 API 出现恢复迹象，但"错误可能反复"；连接器、Claude in Meetings、Managed Agents、Claude Security 仍降级 |
| — | Resolved | claude.ai（含移动端）、API、Claude Code 恢复正常水平，继续监控；Docs / Sheets 连接器恢复；Managed Agents 与 Claude in Meetings 恢复 |
| **15:38** | Resolved（终态） | 影响多界面的问题已解决 |

**总时长 2 小时 29 分。**

> ⚠️ 时长口径说明：该数值是状态页首条（Investigating）到末条（Resolved）的跨度，反映的是**厂商对外通告的窗口**，不等于每一个受影响面都连续不可用 2.5 小时。

---

## 根因（厂商口径）

**厂商未披露根因。**

状态页全部四条更新均为"正在调查 / 正在缓解 / 已恢复"的操作性描述，**没有任何一条提及错误类型、故障组件、触发变更或技术机制**。原文仅止于：

> *"We are investigating reports of degraded performance affecting a variety of surfaces including claude.ai, the Claude API, Claude Code, connectors (including Claude Docs and Claude Sheets), Claude Security and Claude Managed Agents."*

> *"Errors on claude.ai, including the mobile apps, the Claude API and Claude Code have returned to normal levels, and we are continuing to monitor."*

按本仓库纪律：**此处不推测、不填补。** 如果 Anthropic 后续发布 RCA，应在下一轮采集中作为新 item 补入。

---

## 技术要点

> 解读：以下是基于状态页文本的推断性分析，**属于我的判断，非厂商陈述**。

**1. 受影响面横跨三层，说明故障点不在应用层**

同时被点名的有：
- **推理端点**：claude.ai、Claude API（api.anthropic.com）、Claude Code、Claude Cowork
- **第三方连接器**：Claude Docs、Claude Sheets
- **平台能力**：Claude Managed Agents、Claude Security、Claude in Meetings

这三层的共同依赖通常是**认证/鉴权层、内部 API 网关、或统一的模型路由**。单一推理服务故障不会同时打掉 OAuth 连接器和 Agent 编排。

**2. "错误可能反复"是关键信号**

> 原文：*"though errors may return while we continue working on this"*

这句话通常意味着缓解手段是**压制症状而非消除根因** —— 比如限流、切流、扩容。真正的根因修复不会出现"恢复后又反复"。

**3. 连接器恢复晚于主服务**

主链路（claude.ai / API / Code）先恢复，连接器（Docs、Sheets）和 Agent 类能力明显滞后。这说明连接器链路有**独立的、在本次事件中被连带拖垮的下游依赖**，或者其恢复需要额外的凭证/授权校验恢复。

**4. 与 11/12 的 Microsoft 365 故障在时间上相邻，但没有证据表明相关**

同一周内 M365 也发生了大规模故障。**状态页中没有任何交叉引用**，因此不能把两件事关联起来 —— 记录这一点是为了防止事后误归因。

---

## 与我们相关的启示

> 解读：以下是我的分析，非厂商原文。

**① B 级来源的信息天花板就在这里**

这条 item 的价值不在于技术细节（根本没有），而在于**证明了监控管道在工作**：状态页的降级信号被正确捕获、分级、归档。但它也暴露了硬限制 —— **厂商不出 RCA，我们就永远拿不到根因**。

对这类事件，正确的做法是把它当作**线索**而非结论：如果后续看到同厂商的正式复盘、工程博客或会议演讲，应当回填到本 item。

**② 把"受影响面清单"当作依赖图的免费情报**

状态页列出"哪些表面同时挂了"，本身就是一份**厂商内部依赖拓扑的侧面证据**。多次事件累积下来，可以看出哪一层是真正的单点。

**③ 对外通告窗口 ≠ 实际故障窗口**

2 小时 29 分是对外通告的跨度。真实故障可能更早开始（监控延迟）、也可能更晚结束（长尾）。**用状态页时长做 SLO 计算会系统性低估影响。**

**④ 连接器类集成的容错必须独立设计**

Docs / Sheets 这类连接器常被当作"锦上添花"，但它们在本次事件中独立降级、且恢复更慢。如果你的产品依赖第三方连接器做关键流程，**必须有独立的降级路径和超时兜底**，不能假定主服务恢复它就恢复。

---

## 原始报告

- **官方链接**：https://status.claude.com/incidents/34wq7gxnxvvm
- **报告类型**：Anthropic 状态页事故记录（Incident History）—— **无 RCA，无根因披露**
- **本地归档**：`raw/official-report.html`（SHA256 `cb346edaacb8695b…`，完整校验值见 [`sources.md`](sources.md)）
- **受影响组件**（取自状态页 "This incident affected"）：claude.ai、Claude API (api.anthropic.com)、Claude Code、Claude Cowork；更新中另提及 Claude Docs、Claude Sheets、Claude in Meetings、Claude Managed Agents、Claude Security

**后续跟进建议**：监控 Anthropic 官方博客与工程渠道，若发布本次事件的正式复盘，应作为新 item 补入并在本文顶部加注交叉链接。
