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

## 目录结构

```
incidents/<日期>-<厂商>-<标题>/
├── README.md     中文解读（根因 / 技术要点 / 启示）
├── facts.md      机读提取的核心事实表（未经人工核对）
├── meta.json     结构化元数据
├── sources.md    一手来源与归档文件校验值
└── raw/          厂商原始报告原样归档（PDF / HTML / JSON）
```

## 事故索引（3 条）

| 日期 | 厂商 | 事故 | 时长 | 报告类型 |
|---|---|---|---|---|
| 2026-11-13 | Anthropic (Claude) | [Elevated errors on claude.ai, the Claude API, and Claude Code](incidents/2026-11-13-anthropic-claude-elevated-errors-on-claude-ai-the-claude-api-a/) | 149 min | Anthropic (Claude) 状态页事故记录 |
| 2026-10-20 | Google Cloud | [Multiple services were experiencing issues in the europe-west2 region.](incidents/2026-10-20-google-cloud-multiple-services-were-experiencing-issues-in/) | 368 min | Google Cloud Incident Report (厂商官方) |
| 2026-10-07 | Anthropic (Claude) | [Elevated errors on platform.claude.com](incidents/2026-10-07-anthropic-claude-elevated-errors-on-platform-claude-com/) | 8915 min | Anthropic (Claude) 状态页事故记录 |

_最后更新：2026-11-19 03:04 UTC_
