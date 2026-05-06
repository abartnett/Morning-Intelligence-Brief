# Morning Intelligence Brief

An autonomous daily intelligence workflow powered by Claude Code. Each morning at 6:03 AM, the agent fetches from 15+ news outlets, deduplicates cross-outlet coverage, produces detailed summaries organized by topic, runs automated quality control, and delivers a structured Markdown report by email.

## Coverage Areas

- **Geopolitics & Defense** — conflicts, alliances, sanctions, defense procurement
- **Energy & Resources** — OPEC decisions, LNG, critical minerals, commodity moves
- **Markets & Macro** — Fed decisions, macro data, pre-market futures, earnings calendar, sector impact forecast
- **Emerging Markets** — EM political economy, China, India, Southeast Asia, Gulf SWF activity

## How It Works

Claude Code executes `workflows/morning-brief.md` — a plain-English recipe that handles:

1. Markets data fetch (pre-market futures, yesterday's close, earnings calendar)
2. Source fetch from 15+ outlets via RSS, WebSearch, and NYT API
3. 24-hour filter + topic relevance classification
4. Deduplication (entity + event type grouping, tier-ranked canonical selection)
5. Full-text summarization (2–3 paragraphs + key takeaways per story)
6. Report assembly with Executive Summary written last
7. Automated QC (6 checks: consistency, conflict parties, numerical claims, causal framing, unverifiable claims, scope)
8. Email delivery

## Structure

```
workflows/          Executable workflow recipe (morning-brief.md)
resources/          Source registry, topic criteria, setup guides
output/             Generated daily briefs (YYYY-MM-DD-morning-brief.md)
.claude/commands/   QC skill (qc-brief.md)
```

## Setup

See `resources/paywall-setup.md` for one-time credential setup (NYT API key, Gmail app password, subscriber RSS URLs).

Required environment variables (set in `~/.zshrc`, never committed):
- `GMAIL_APP_PASSWORD` — Gmail app password for email delivery
- `NYT_API_KEY` — New York Times Article Search API key
