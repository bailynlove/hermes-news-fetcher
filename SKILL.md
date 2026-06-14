---
name: ai-news-fetcher
description: AI/技术新闻聚合引擎。并发抓取 100+ RSS 源、arXiv 论文、GitHub Trending；支持兴趣评分、跨天去重、日期过滤、统一预取。当用户需要查看 AI 新闻、技术趋势、最新论文、热门项目、AI 公司动态时使用。
---

# ai-news-fetcher — AI/技术新闻高性能聚合引擎

并发抓取 100+ AI/技术资讯源，12 秒完成全量更新；支持 ETag/Last-Modified 缓存、跨天去重、按兴趣评分排序，纯 Python 标准库实现，零外部依赖。

## When to Use

当用户出现以下意图时调用本 skill：

- **新闻聚合**：「AI 新闻」「技术新闻」「今天有什么新闻」「日报」「周报」
- **学术追踪**：「最新论文」「arXiv」「AI 研究进展」
- **趋势项目**：「GitHub 热门」「Trending」「最近火的项目」
- **公司动态**：「OpenAI 动态」「Anthropic 更新」「Google AI」
- **批量预取**：「拉一份每日数据」「为后续摘要准备数据」

## Setup

需要 Python 3.8+，纯标准库，无需 `pip install`。如果需要走代理（境内访问 OpenAI/HuggingFace 等），在调用前设置：

```
HTTP_PROXY=http://127.0.0.1:1087
HTTPS_PROXY=http://127.0.0.1:1087
```

## Architecture

```
ai-news-fetcher/
├── SKILL.md
└── scripts/
    ├── rss_aggregator.py        # 核心 RSS 抓取（并发 + 缓存）
    ├── rss_sources.json         # 100+ RSS 源配置
    ├── build_digest_input.py    # 兴趣评分 + 跨天去重 + per_source 限额
    ├── build_reflection_index.py# 周复盘索引生成
    ├── arxiv_papers.py          # arXiv AI/Agent/Memory 论文搜索
    ├── github_trending.py       # GitHub Trending AI 项目
    ├── summarize_url.py         # 单篇文章正文抓取
    ├── gen_status.py            # 预取状态汇总
    └── prefetch_data.sh         # 一键并行预取入口
```

## Data Sources

| 分类 | 源数 | 内容 |
|------|------|------|
| company | 16 | OpenAI / Anthropic / Google / Meta / NVIDIA / Apple / Mistral 等官方博客 |
| papers | 6 | arXiv AI/ML/NLP/CV / HuggingFace Daily Papers / BAIR |
| media | 16 | MIT Tech Review / TechCrunch / Wired / The Verge / VentureBeat |
| newsletter | 15 | Simon Willison / Lilian Weng / Andrew Ng / Karpathy 等专家 |
| community | 12 | HN / GitHub Trending / Product Hunt / V2EX |
| cn_media | 5 | 机器之心 / 量子位 / 36氪 / 少数派 / InfoQ |
| ai-agent | 5 | LangChain / LlamaIndex / Mem0 / Ollama / vLLM |
| twitter | 10 | Sam Altman / Karpathy / LeCun / Hassabis 等领袖 |

## Core Commands

通过终端工具执行以下命令；路径使用 `.hermes/skills/ai-news-fetcher/scripts/` 前缀（依据当前 host 的 skill root 替换）。

### 1. 统一预取（推荐）
一键并行抓取 RSS + GitHub + arXiv，输出到 `/tmp/ainews_prefetch/`：

```bash
bash .hermes/skills/ai-news-fetcher/scripts/prefetch_data.sh
```

输出文件：
- `digest_latest.json` — 经过兴趣评分与去重的候选集
- `github_trending.json` — GitHub AI 趋势项目
- `arxiv_papers.json` — arXiv 最新论文
- `status.json` — 数据状态汇总

### 2. RSS 聚合（按需查询）

```bash
# 抓取所有源最近 3 天的新闻，每源限 10 条
python3 .hermes/skills/ai-news-fetcher/scripts/rss_aggregator.py \
  --category all --days 3 --limit 10

# 只看公司博客（最近 1 天）
python3 .hermes/skills/ai-news-fetcher/scripts/rss_aggregator.py \
  --category company --days 1 --limit 5

# 中文媒体
python3 .hermes/skills/ai-news-fetcher/scripts/rss_aggregator.py \
  --category cn_media --days 3 --limit 10

# 输出 JSON 给下游处理
python3 .hermes/skills/ai-news-fetcher/scripts/rss_aggregator.py \
  --category all --days 1 --json
```

### 3. 摘要输入构建（兴趣评分 + 跨天去重）

```bash
python3 .hermes/skills/ai-news-fetcher/scripts/build_digest_input.py \
  --out /tmp/digest.json --days 2 --max-total 40 --workers 20 --timeout 10
```

### 4. arXiv 论文

```bash
# AI/Agent/Memory 相关最新论文，取前 8 篇
python3 .hermes/skills/ai-news-fetcher/scripts/arxiv_papers.py --top 8 --json

# 自定义关键词
python3 .hermes/skills/ai-news-fetcher/scripts/arxiv_papers.py \
  --query "multi-agent" --top 5
```

### 5. GitHub Trending

```bash
# AI 相关每日热门
python3 .hermes/skills/ai-news-fetcher/scripts/github_trending.py \
  --ai-only --limit 10 --json

# 本周热门
python3 .hermes/skills/ai-news-fetcher/scripts/github_trending.py \
  --since weekly
```

### 6. 单篇文章摘要

```bash
python3 .hermes/skills/ai-news-fetcher/scripts/summarize_url.py \
  --url "https://example.com/article" --json
```

### 7. 周复盘索引

```bash
python3 .hermes/skills/ai-news-fetcher/scripts/build_reflection_index.py
```

## Core Rules

### 1. 优先使用 `--days` 限定窗口
默认窗口 ≤ 7 天，避免过期内容：
- 日报：`--days 1`
- 周报：`--days 7`
- 月度回顾：`--days 30`

### 2. 分类选择策略

| 用户需求 | 推荐分类 |
|----------|----------|
| 公司动态 | `--category company` |
| 技术论文 | `--category papers` |
| 中文资讯 | `--category cn_media` |
| 社区趋势 | `--category community` |
| AI Agent | `--category ai-agent` |
| 行业媒体 | `--category media` |
| 专家通讯 | `--category newsletter` |
| 全部 | `--category all` |

### 3. 输出协议
- 给 agent 后续处理用 `--json`
- 给人类阅读用默认 Markdown
- 大批量数据走 `prefetch_data.sh`（写入 `/tmp/ainews_prefetch/`），下游直接读文件

### 4. 缓存机制
- ETag / Last-Modified 自动协商
- 缓存 TTL 1 小时（`/tmp/.ainews_rss_cache.json`）
- 跨天 URL 去重窗口 7 天

### 5. 失败容错
单源失败不影响整体抓取；`prefetch_data.sh` 中任一脚本失败时落 `[]` 兜底，保证下游可读。

## Configuration

编辑 `scripts/rss_sources.json` 增删 RSS 源：

```json
{
  "name": "OpenAI Blog",
  "url": "https://openai.com/blog/rss.xml",
  "category": "company"
}
```

## Notes for Agent

- 该 skill 仅产出**结构化数据**（JSON / Markdown），不做 LLM 摘要；摘要交由调用方 agent 完成。
- 默认入口是 `prefetch_data.sh`；做单点查询才直接调用对应 `.py`。
- 长任务建议在后台执行（`prefetch_data.sh` 实测约 12–30 秒）。
- 输出文件路径固定 `/tmp/ainews_prefetch/`，多次调用会覆盖。
