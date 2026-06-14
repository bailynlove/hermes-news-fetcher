# ai-news-fetcher

Hermes-agent 兼容版 AI 新闻聚合 skill，从 [`ai-news-aggregator`](../ai-news-aggregator/) 包装而来。

## What's different from the original?

| 项 | ai-news-aggregator | ai-news-fetcher (本目录) |
|----|--------------------|--------------------------|
| 目标宿主 | OpenClaw / Claude Code | Hermes Agent |
| frontmatter | 完整字段 (slug/version/changelog/metadata...) | 仅 `name` + `description`（Hermes allowlist） |
| 路径前缀 | `skills/ai-news-aggregator/` | `.hermes/skills/ai-news-fetcher/` |
| 工具术语 | Bash / Read / Write | terminal / read_file / patch |
| 文档命名 | CLAUDE.md / SKILL.md | AGENTS.md / SKILL.md |
| 核心脚本 | 同源（已修复一处硬编码 cwd） | 同上 |

## Files

```
ai-news-fetcher/
├── README.md                  # 本文件
├── SKILL.md                   # Hermes skill 描述（已 allowlist frontmatter）
└── scripts/
    ├── arxiv_papers.py
    ├── build_digest_input.py  # 已修复 cwd 路径假设
    ├── build_reflection_index.py
    ├── gen_status.py
    ├── github_trending.py
    ├── prefetch_data.sh
    ├── rss_aggregator.py
    ├── rss_sources.json
    └── summarize_url.py
```

## Install (Hermes)

```bash
# 复制到 hermes skill root
cp -r ai-news-fetcher ~/.hermes/skills/

# 或在项目本地安装
mkdir -p .hermes/skills && cp -r ai-news-fetcher .hermes/skills/
```

## Quick test

```bash
# 直接跑预取
bash ~/.hermes/skills/ai-news-fetcher/scripts/prefetch_data.sh
ls -la /tmp/ainews_prefetch/
cat /tmp/ainews_prefetch/status.json
```

## License

MIT (随上游 ai-news-aggregator)。
