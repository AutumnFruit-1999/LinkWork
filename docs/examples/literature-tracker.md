# Example Role: Literature Tracker

This document demonstrates how to configure a complete AI worker in LinkWork using the "Literature Tracker" role as an example — enabling it to continuously track cutting-edge developments, automatically collect and archive knowledge, and periodically generate reports.

---

## Role Overview

| Attribute | Value |
|-----------|-------|
| Role Name | Literature Tracker |
| Responsibilities | Track cutting-edge developments, automatically collect and archive external knowledge, periodically generate structured reports |
| Target Users | Researchers, technical managers, knowledge workers |
| Core Value | 24/7 personal knowledge assistant for information noise reduction and knowledge accumulation |

---

## Working Modes

### Mode 1: Build a Personal Knowledge Base by Interest

Users tell the Literature Tracker their areas of interest, and the AI automatically creates a structured knowledge directory. Each person's knowledge base is completely independent and invisible to others.

Using "AI Large Language Models" as an example, the generated directory structure would be:

```
literature/
├── README.md                      # Personal knowledge base user guide
├── daily-news/                    # Daily news digest (archived by date)
├── papers/                        # Paper summaries + in-depth reading notes
├── github-projects/               # GitHub open-source project weekly reports
├── reading-notes/                 # Deep reading notes (long-term retention)
├── industry-reports/              # Industry report and whitepaper summaries
├── tech-blogs/                    # Quality tech blog article commentaries
├── _templates/                    # Document templates
└── _archive/                      # Auto-archive for expired content
```

The directory structure is not fixed — the AI tailors it to the user's interest areas (AI, finance, biomedical, etc.).

### Mode 2: Feed It Anything, Auto-archive

When you come across a great article or paper, just throw the link to the Literature Tracker. The AI will fetch related information via Tavily Search, extract summaries and key insights, archive to the appropriate knowledge base directory, and index in the personal knowledge catalog.

**Example Interaction:**

```
User: https://arxiv.org/abs/2401.12345

AI: Fetched paper "Scaling Laws for Neural Language Models"
    → Archived to papers/2026-03-05-scaling-laws.md
    Summary: This paper proposes... Key findings: ① ... ② ... ③ ...
    Added to your knowledge index.
```

### Mode 3: Scheduled Fetching, Daily Digest

After configuring sources of interest, the Literature Tracker runs automatically every day:

- **Morning**: Fetch latest paper updates, AI news, and GitHub Trending projects via Tavily Search
- **Organize**: Filter noise, screen by areas of interest, generate the daily digest
- **Archive**: Write to the `daily-news/` directory, update the knowledge index

**Example Output:**

```
[Auto-executes daily at 8:00 AM]

Today's AI Digest generated → daily-news/2026-03-05.md

  Highlights
  1. DeepSeek-V4 officially released, reasoning performance up 40%
  2. Google open-sources Gemma 3 model series

  New Papers (7 matched your interests)
  1. "Attention is All You Need... Again" — New Transformer architecture variant
  2. ...

  GitHub Trending
  1. openai/swarm — Multi-Agent orchestration framework (new this week)
  2. ...
```

### Mode 4: Periodic Reports, Distilled Insights

The Literature Tracker periodically performs aggregate analysis on accumulated literature, automatically outputting reports:

- **Weekly Report**: What was collected this week, trending topics, which papers deserve deeper reading
- **Monthly Report**: Full domain landscape for the month, key technology evolution threads, recommended reading list

### Mode 5: Ask Anytime, Knowledge On-demand

All archived literature is indexed in the personal knowledge catalog. Users can ask questions in natural language anytime, and the AI answers based on accumulated knowledge:

```
User: What new developments in MoE architecture have there been in the past month?

AI: Based on 12 relevant documents in your knowledge base, key developments include:
    1. DeepSeek proposed "Fine-Grained Expert Segmentation"...
    2. Google adopted dynamic routing strategy in Gemini 2.5...
    3. Tsinghua's OpenMoE team open-sourced complete training code...

    Detailed summary generated → reading-notes/moe-summary-202603.md
```

---

## Role Configuration Reference

### Skills Configuration

| Skill | Purpose |
|-------|---------|
| Literature Fetching | Fetch full text from URLs, supporting papers, blogs, news sites |
| Summary Extraction | Extract key insights and critical data per template |
| Knowledge Indexing | Maintain personal knowledge index, support semantic retrieval |
| Report Generation | Auto-generate trend analysis reports on weekly/monthly basis |

### MCP Tools

| Tool | Purpose |
|------|---------|
| Tavily Web Search | Search papers, news, GitHub projects, and other public information with AI-optimized structured search results |
| Web Scraping Tool | Access and parse web page content, extract body text |

### Scheduled Tasks

| Task | Cron Expression | Description |
|------|----------------|-------------|
| Daily Digest | `0 8 * * *` | Execute daily at 8:00 AM |
| Weekly Report | `0 9 * * 1` | Execute every Monday at 9:00 AM |
| Monthly Report | `0 10 1 * *` | Execute on the 1st of each month at 10:00 AM |

---

## Key Features

| Feature | Description |
|---------|-------------|
| Personally Exclusive | Independent knowledge base and memory space per person, invisible to others |
| Feed Anything | Just throw a link — AI auto-fetches, organizes, and archives |
| Self-running | Scheduled fetching + periodic reports, no manual intervention needed |
| Gets Smarter Over Time | The more accumulated, the more precise retrieval becomes, and reports gain more depth |
| Flexibly Customizable | Interest areas, sources, and report frequency are all adjustable on demand |

---

## Further Reading

- [Workstation Model](../concepts/workstation.md) — Understand the conceptual design of workstations
- [Skills System](../concepts/skills.md) — Learn how Skills work
- [Extension Guide](../guides/extension.md) — Learn how to create your own roles
