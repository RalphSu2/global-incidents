# 全球互联网可靠性事故 · 原始报告库

每一条 = 一起重大事故的**原始厂商报告归档 + 中文技术解读**。

## 收录标准

**A 级 · 厂商正式技术报告**（优先收录）

- Microsoft Azure `Post Incident Review (PIR)`（仅 Scenario 1 公开）
- AWS `Post-Event Summary (PES)`
- Google Cloud `Incident Report`（状态页 Incident 页含完整 RCA）
- Cloudflare 官方 Post-Mortem 博客

**B 级 · 厂商状态页事故记录**（需达影响门槛才收录）

- 影响评分 ≥ 3.5（outage / down / unavailable / elevated errors 等关键词 + 受影响面加权）
- 连接器故障、单区域小幅降级、计划性维护不收录

**C 级 · 厂商未发布技术报告**（仍保留条目）

- 厂商确认事件存在，但**没有发布** PIR / PES / Post-Mortem 等技术报告
- 条目内容限于：官方声明原文、状态页状态、公开表态、权威报道
- **保留目的**：把「谁不给报告」变成可统计的事实，避免故障从公共记录中消失
- 此类条目的 `README.md` 会明确写「厂商未披露根因」，不做任何推测填补

## 目录结构

```
incidents/<日期>-<厂商>-<标题>/
├── README.md     中文解读（根因 / 技术要点 / 启示）
├── facts.md      机读提取的核心事实表（未经人工核对）
├── meta.json     结构化元数据
├── sources.md    一手来源与归档文件校验值
└── raw/          厂商原始报告原样归档（PDF / HTML / JSON）
```

## 事故索引（12 条）

| 日期 | 分级 | 厂商 | 事故 | 时长 | 报告类型 |
|---|---|---|---|---|---|
| 2026-11-17 | C | NSA | [NSANet 机密内网临时降级（例行维护期间）](incidents/2026-11-17-nsa-nsanet-classified-network-temporary-degradation/) | — | NSA 官方新闻稿声明（短声明，无技术报告） |
| 2026-11-17 | C | Apple | [iCloud 登录与 App Store 故障：仅一句「内部更新」](incidents/2026-11-17-apple-icloud-sign-in-and-app-store-internal-update/) | 270 min | Apple 系统状态页（仅状态记录，无任何说明） |
| 2026-11-16 | C | United Airlines | [配重平衡系统故障导致航班延误](incidents/2026-11-16-united-airlines-weight-and-balance-system-outage/) | 90 min | 工会与公司发言人表述（无书面报告） |
| 2026-11-16 | C | JPMorgan Chase | [本地部署系统软件缺陷导致网银与转账大面积失败](incidents/2026-11-16-jpmorgan-chase-on-premise-software-bug-banking-outage/) | 420 min | Chase 官方 X 账号声明（一段话，无技术报告） |
| 2026-11-15 | C | AWS | [us-east-1 认证失败引发 11 项服务连锁降级](incidents/2026-11-15-aws-us-east-1-iam-identity-center-authentication-failure/) | 196 min | AWS 未发布 Post-Event Summary（该页面最新条目仍为 2025-10-19） |
| 2026-11-13 | C | Workday | [灾备演练把 DR 环境连到了生产数据库](incidents/2026-11-13-workday-disaster-recovery-test-connected-to-production-db/) | 1056 min | Workday 客户状态页通知（无公开永久链接） |
| 2026-11-13 | C | FAA | [全美航班停飞：铜缆转光纤路由器打爆老旧设备](incidents/2026-11-13-faa-national-ground-stop-l3harris-router-copper-to-fiber/) | 240 min | FAA 新闻室与交通部长声明（无书面技术报告） |
| 2026-11-13 | B | Anthropic (Claude) | [Elevated errors on claude.ai, the Claude API, and Claude Code](incidents/2026-11-13-anthropic-claude-elevated-errors-on-claude-ai-the-claude-api-a/) | 149 min | Anthropic (Claude) 状态页事故记录 |
| 2026-11-12 | A | Microsoft | [Microsoft 365 / Exchange Online 级联故障（近 24 小时）](incidents/2026-11-12-microsoft-microsoft-365-exchange-online-23-hour-outage/) | 1430 min | Microsoft 365 Preliminary Post Incident Report（正文在 M365 管理中心内，公开仅有执行摘要 + 第三方 PDF 镜像） |
| 2026-11-09 | C | Microsoft Azure | [Azure North Europe 多台网络设备断电](incidents/2026-11-09-microsoft-azure-north-europe-power-loss-network-devices/) | 219 min | Azure 门户内 Service Issue 通知（未达 Scenario 1，不公开 PIR） |
| 2026-10-20 | A | Google Cloud | [Multiple services were experiencing issues in the europe-west2 region.](incidents/2026-10-20-google-cloud-multiple-services-were-experiencing-issues-in/) | 368 min | Google Cloud Incident Report (厂商官方) |
| 2026-10-07 | B | Anthropic (Claude) | [Elevated errors on platform.claude.com](incidents/2026-10-07-anthropic-claude-elevated-errors-on-platform-claude-com/) | 8915 min | Anthropic (Claude) 状态页事故记录 |

_最后更新：2026-11-19 03:27 UTC_
