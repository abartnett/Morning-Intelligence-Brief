# Workflow: Morning Intelligence Brief
Version: 1.0  
Schedule: Daily at 06:03 local time (CronCreate cron `3 6 * * *`)  
Output: `output/YYYY-MM-DD-morning-brief.md`  
Email: abartnett@gmail.com

---

## Before You Begin

Load these two resource files into your working context before executing any steps:
- `resources/sources.md` — outlet list with fetch methods and URLs
- `resources/topic-criteria.md` — relevance rules for each topic bucket

---

## Step 1 — Set Up the Run

Record today's date as `RUN_DATE` in YYYY-MM-DD format (e.g., 2026-04-21).  
Set YESTERDAY as the date one day prior (for WebSearch `after:` filters).  
Set the lookback cutoff: any article published more than 24 hours before now is out of scope.  
Set the output file path: `output/RUN_DATE-morning-brief.md`.  
Note the run start time (HH:MM local) — this goes in the report header.

---

## Step 1.5 — Fetch Markets Data

Run the curl commands and WebSearch queries below. Keep all curl calls in one parallel batch and all WebSearch calls in a separate parallel batch — never mix them. Record all results in a working markets snapshot.

**Day-of-week check — run first:**
Check whether RUN_DATE falls on a Saturday or Sunday. If it does, skip Batch 1 entirely and fill all Pre-Market Futures rows with `[N/A — weekend]`. Proceed directly to Batch 2.

**Batch 1 — Live futures data (Bash curl, run sequentially with 3-second delays):**

*Skip on Saturday and Sunday — CME Globex weekend hours do not produce meaningful pre-market data for the brief.*

Run these seven curl calls one at a time, with a 3-second sleep between each. Do NOT run them in parallel — Yahoo Finance rate-limits simultaneous requests. Each returns a JSON price quote. Extract `regularMarketPrice` and `regularMarketChangePercent` from the `chart.result[0].meta` object.

```bash
source ~/.zshrc && curl -s "https://query1.finance.yahoo.com/v8/finance/chart/ES%3DF?interval=1m&range=1d" || (sleep 3 && curl -s "https://query1.finance.yahoo.com/v8/finance/chart/ES%3DF?interval=1m&range=1d")
```
```bash
sleep 3 && source ~/.zshrc && curl -s "https://query1.finance.yahoo.com/v8/finance/chart/NQ%3DF?interval=1m&range=1d" || (sleep 3 && curl -s "https://query1.finance.yahoo.com/v8/finance/chart/NQ%3DF?interval=1m&range=1d")
```
```bash
sleep 3 && source ~/.zshrc && curl -s "https://query1.finance.yahoo.com/v8/finance/chart/YM%3DF?interval=1m&range=1d" || (sleep 3 && curl -s "https://query1.finance.yahoo.com/v8/finance/chart/YM%3DF?interval=1m&range=1d")
```
```bash
sleep 3 && source ~/.zshrc && curl -s "https://query1.finance.yahoo.com/v8/finance/chart/GC%3DF?interval=1m&range=1d" || (sleep 3 && curl -s "https://query1.finance.yahoo.com/v8/finance/chart/GC%3DF?interval=1m&range=1d")
```
```bash
sleep 3 && source ~/.zshrc && curl -s "https://query1.finance.yahoo.com/v8/finance/chart/CL%3DF?interval=1m&range=1d" || (sleep 3 && curl -s "https://query1.finance.yahoo.com/v8/finance/chart/CL%3DF?interval=1m&range=1d")
```
```bash
sleep 3 && source ~/.zshrc && curl -s "https://query1.finance.yahoo.com/v8/finance/chart/%5ETNX?interval=1m&range=1d" || (sleep 3 && curl -s "https://query1.finance.yahoo.com/v8/finance/chart/%5ETNX?interval=1m&range=1d")
```
```bash
sleep 3 && source ~/.zshrc && curl -s "https://query1.finance.yahoo.com/v8/finance/chart/NG%3DF?interval=1m&range=1d" || (sleep 3 && curl -s "https://query1.finance.yahoo.com/v8/finance/chart/NG%3DF?interval=1m&range=1d")
```

Ticker key: `ES%3DF` = S&P 500 futures, `NQ%3DF` = Nasdaq futures, `YM%3DF` = DOW futures, `GC%3DF` = Gold, `CL%3DF` = WTI crude oil, `%5ETNX` = 10-year Treasury yield, `NG%3DF` = Natural gas.

If any curl returns an error or empty result after retry, note `[Data unavailable]` for that instrument and continue.

**Batch 2 — Index closes, earnings calendar (WebSearch, run in parallel):**

Run these three WebSearch queries simultaneously:

- `S&P 500 Nasdaq Dow Jones closing price YESTERDAY`
- `S&P 500 earnings after market close RUN_DATE`
- `IONQ QBTS RGTI PL VRT earnings RUN_DATE`

Extract: (1) yesterday's closing price and % change for each index, (2) list of S&P 500 companies reporting after close today, (3) whether any watchlist tickers are reporting today and their consensus EPS/revenue estimates.

If any query returns no results or ambiguous data, note `[Data unavailable]` for that field. Do not abort — continue to Step 2.

The Sector Impact Forecast (the final subsection of Markets Update) is written in Step 7 after all news summaries are complete.

---

## Step 2 — Fetch Articles from All Sources

Work through every outlet in `resources/sources.md` in order. Use the fetch method specified for each outlet. Collect all results into a single working list. Each entry in the list must have: headline, URL, outlet name, pubDate (or fetch time if missing), and a body snippet of at least 2–3 sentences.

**Method A — Subscriber RSS (WebFetch):**  
Use `WebFetch` on the feed URL. Parse the XML response to extract all `<item>` entries. For each item record: `<title>`, `<link>`, `<pubDate>`, and `<description>` (or `<content:encoded>` if present — prefer the longer field). If the feed returns an error or is empty, log `[Feed error — skipped]` for that outlet in your working notes and continue.

**Method B — NYT Article Search API (Bash curl):**  
Run four separate API calls — one per topic — using the endpoint and query pattern in `resources/sources.md`. The `NYT_API_KEY` environment variable must be set (see `resources/paywall-setup.md` if not). Use `Bash curl` (not WebFetch — api.nytimes.com is blocked by WebFetch).

**Batch isolation rule:** Run all four NYT Bash curl calls as one parallel batch. Run any WebSearch calls (Methods C/D) in a completely separate parallel batch. Never mix Bash and WebSearch calls in the same parallel batch — a Bash failure will cancel all calls in the same batch, including WebSearch queries.

Run each of the four topic queries using this exact pattern:

```bash
source ~/.zshrc && YESTERDAY_DATE=$(date -v-1d +%Y%m%d) && curl -s "https://api.nytimes.com/svc/search/v2/articlesearch.json?q=[QUERY]&sort=newest&begin_date=$YESTERDAY_DATE&api-key=$NYT_API_KEY" | python3 -c "
import json,sys
raw = sys.stdin.read()
data = json.loads(raw)
if 'response' not in data:
    print('ERROR:', raw[:200])
    sys.exit(1)
docs = (data['response'].get('docs') or [])[:8]
for d in docs:
    print('TITLE:', (d.get('headline') or {}).get('main', '[no title]'))
    print('DATE:', d['pub_date'])
    print('URL:', d['web_url'])
    print('SNIPPET:', d.get('abstract','') or d.get('lead_paragraph',''))
    print('---')
"
```

Replace `[QUERY]` with the topic keyword for each call (e.g., `geopolitics`, `energy`, `economy`, `emerging+markets`). Combine all four query results and deduplicate by URL before adding to the working list. If curl returns `{"fault":...}` or the script prints `ERROR:`, log `[NYT API key not in session — run source ~/.zshrc]` in the Source Coverage Table.

**Method C — Bloomberg (WebSearch + cookie curl):**  
First run `WebSearch site:bloomberg.com [topic keywords] after:YESTERDAY` to discover article URLs. Then for each discovered URL, use `Bash curl -s -b resources/cookies/bloomberg.txt "[article_url]"` to fetch the full page. Extract the article body text from the HTML (look for `<article>` tags or the largest contiguous text block). If curl returns a login page or 403, log `[Bloomberg cookies expired — snippet only]` in the Source Coverage Table and use the WebSearch snippet instead.

**Method D — Open WebSearch (snippet only):**  
Use `WebSearch site:[domain] [topic keywords] after:YESTERDAY` for CFR, RAND, and Foreign Affairs. Record the headline, URL, and snippet returned by the search. These will be summarized from the snippet; label them `[Snippet only]` in the Source Coverage Table.

**On any source failure:** Log the failure status in your working notes. Do not abort — move on to the next source immediately.

---

## Step 3 — Apply the 24-Hour Filter

Review the pubDate of every article in the working list. Discard any article whose pubDate is more than 24 hours before the current run time. If pubDate is absent, keep the article and treat the fetch time as its timestamp — note `[Date unknown]` in the Source Coverage Table for that outlet's entries.

---

## Step 4 — Apply Topic Relevance Filter

For each article in the working list, read the headline and body snippet. Using the keyword lists and exclusion rules in `resources/topic-criteria.md`, classify each article into one or more of these buckets:

- `GEOPOLITICS` — Geopolitics & Defense  
- `ENERGY` — Energy & Resources  
- `MARKETS` — Markets & Macro  
- `EM` — Emerging Markets  
- `NOISE` — discard

An article may be placed in multiple buckets if it genuinely spans topics (e.g., a China oil import story is both `ENERGY` and `EM`). Discard all `NOISE` articles from the working list. Keep a count of articles reviewed vs. kept per outlet — this feeds the Source Coverage Table.

---

## Step 5 — Deduplicate

After filtering, identify stories that multiple outlets covered. Follow this algorithm exactly:

**Phase 1 — Group by subject + event:**  
For each article extract: (a) primary subject = the first proper noun in the headline (country, organization, or person name), and (b) event verb cluster = one of: `attack/strike`, `sanction/restrict`, `agree/sign/deal`, `collapse/fail`, `elect/appoint`, `announce/plan`, `report/warn`, `rise/fall/surge/drop`. Group together all articles where both (a) and (b) match.

**Phase 2 — Confirm duplicate within each group:**  
Two articles in the same group are confirmed duplicates if: (i) four or more words appear in both headlines (word-set overlap, not requiring them to be consecutive), OR (ii) both headlines share a named entity AND a specific number (casualty count, dollar figure, percentage) AND a location name.

**Phase 3 — Choose the canonical article:**  
From each confirmed duplicate group, keep one canonical article using this priority: Tier-1 outlet > Tier-2 > Tier-3 (tiers defined in `resources/sources.md`). Break ties by longest body text. Break further ties by earliest pubDate.

**Phase 4 — Record `covered_by`:**  
For the canonical article, record a `covered_by` field listing every other outlet in the group. This is displayed in the report as *Also reported by: [outlets]*.

**Edge case:** If two articles from the same outlet cover different angles of the same event (e.g., one covers the military dimension and one the economic dimension of the same sanction), keep both — they are not duplicates.

---

## Step 6 — Fetch Full Article Text

For each surviving canonical article, fetch the full body text using the method appropriate to its outlet:

- **Subscriber RSS articles:** Full text is already in the feed body from Step 2. Use it directly.
- **NYT articles:** Full text is already in the API response from Step 2. Use it directly.
- **Bloomberg articles:** Full text was fetched via cookie curl in Step 2. Use it directly.
- **Open RSS / WebFetch articles:** Use `WebFetch` on the article URL to get the full page. Extract the article body text.
- **Snippet-only articles (CFR, RAND, Foreign Affairs):** Use `WebFetch` on the URL — these are open-access sites and should return full text. If WebFetch fails, use the search snippet and note `[Full text unavailable — snippet used]`.

If full text fetch fails for any article, use whatever snippet is available and mark it `[Full text unavailable]`. Do not skip the article.

---

## Step 7 — Write Summaries

For each canonical article, write the following. Use the full article text from Step 6 as your source. Do not editorialize or add opinions — stay factual and analytical.

**Paragraph 1 — What happened:**  
State the core event in 3–5 sentences. Name the key actors, the action taken, the location, and the date. Include any specific figures quoted (troop numbers, dollar amounts, vote tallies, price levels).

**Paragraph 2 — Why it matters:**  
2–4 sentences on strategic, economic, or geopolitical significance. Address: Who benefits? Who is harmed? What precedent does this set? How does it connect to existing trends or conflicts?

**Paragraph 3 — What to watch (optional):**  
Include only if there is a concrete, actionable signal to monitor — an upcoming vote, a decision deadline, a specific indicator. Skip this paragraph if there is no clear forward-looking hook. Keep to 1–3 sentences.

**Key Takeaways — 3 to 5 bullet points:**  
Each bullet is one sentence, factual, and self-contained. Together they should be scannable in under 30 seconds. Do not repeat verbatim from the paragraphs — distill.

**Sector Impact Forecast (write after all article summaries are complete):**  
Write one sentence per affected sector assessing how the day's top stories are likely to impact it. Cover these five sectors: Defense & Aerospace, Energy, Technology, Financials, Emerging Markets / EM equities. Use only content from the articles summarized above — do not introduce new facts. For each sentence, state: (1) which story creates the exposure, (2) the direction of likely impact (positive/negative/mixed), and (3) the mechanism. Skip any sector with no relevant story in the brief and write "No significant story today." This block populates the Sector Impact Forecast subsection of Markets Update.

---

## Step 8 — Assemble the Report

Assemble sections in this order. Write the Executive Summary last.

**Section order:**
1. Executive Summary (write last, insert at top)
2. Markets Update (data from Step 1.5; Sector Impact Forecast from Step 7)
3. Geopolitics & Defense
4. Energy & Resources
5. Markets & Macro
6. Emerging Markets
7. Source Coverage Summary

**Within each topic section:**  
Order articles by significance: the most consequential story (widest geopolitical or market impact, most outlets covering it) goes first. Supporting stories follow.

**Executive Summary (write after all sections are drafted):**  
250–350 words. Open with the single most consequential story across all sections — 2–3 sentences. Then write 2–3 sentences on the lead story from each remaining section. Close with one sentence listing the top 2–3 things to watch in the next 24–48 hours. Tone: direct, analytical, declarative. No hedging phrases ("it remains to be seen," "could potentially," "may or may not").

Use the exact report template specified below in the Report Format section.

---

## Step 9 — Save the Report

Write the completed report to `output/RUN_DATE-morning-brief.md`. Replace `RUN_DATE` with the actual date string from Step 1 (e.g., `output/2026-04-21-morning-brief.md`). Confirm the file was written successfully before proceeding to Step 9.5.

---

## Step 9.5 — Quality Control

Execute the QC protocol defined in `.claude/commands/qc-brief.md` directly — read that file and follow each step inline. Do not use the Skill tool. The protocol re-fetches key sources, checks the brief across six failure categories, appends a QC FLAGS section to the saved report file, and then always proceeds. No human needs to be present — the brief sends regardless of findings. Any issues are appended to the report so the reader can see them when the email arrives.

After the QC protocol completes, continue to Step 10.

---

## Step 10 — Send Email

Run the following command exactly:

```
python3 ~/scripts/send_morning_brief.py "output/RUN_DATE-morning-brief.md"
```

Replace `RUN_DATE` with the actual date string. If the script exits with a non-zero code, append the following to the bottom of the saved report:

```
---
*Email delivery failed. Error: [paste error message here]. Report available at output/RUN_DATE-morning-brief.md.*
```

If the script succeeds, no additional action is needed.

---

## Step 11 — Confirm Completion

Output this line:

```
Morning brief for RUN_DATE complete. Report saved to output/RUN_DATE-morning-brief.md. Email sent to abartnett@gmail.com.
```

---

## Report Format Template

Use this exact Markdown structure. Do not add extra headers or reorder sections.

```markdown
# Morning Intelligence Brief — [Full weekday, Month DD, YYYY]
*Generated by Claude Code | Run time: HH:MM | Articles reviewed: N | Stories included: M*

---

## Executive Summary

[250–350 words.]

---

## Markets Update
*Data as of pre-market [RUN_DATE] | Prices approximate — verify before trading*

### Yesterday's Close ([YESTERDAY])
| Index | Close | Change |
|-------|-------|--------|
| S&P 500 | [price] | [+/-X.XX%] |
| Nasdaq | [price] | [+/-X.XX%] |
| DOW | [price] | [+/-X.XX%] |

### Pre-Market Futures
| Instrument | Level / Price | Change |
|------------|--------------|--------|
| S&P 500 Futures | | |
| Nasdaq Futures | | |
| DOW Futures | | |
| Gold | | |
| 10-Year Treasury Yield | | |
| WTI Crude Oil | | |
| Natural Gas | | |

### Earnings After Close Today ([RUN_DATE])
**S&P 500 companies reporting:** [list, or "None scheduled"]
**Watchlist:** IONQ — [reporting / not reporting]; QBTS — [reporting / not reporting]; RGTI — [reporting / not reporting]; PL — [reporting / not reporting]; VRT — [reporting / not reporting]

### Sector Impact Forecast
*Based on today's top stories — see full brief below*

- **Defense & Aerospace:** [one sentence, or "No significant story today"]
- **Energy:** [one sentence, or "No significant story today"]
- **Technology:** [one sentence, or "No significant story today"]
- **Financials:** [one sentence, or "No significant story today"]
- **Emerging Markets / EM equities:** [one sentence, or "No significant story today"]

---

## Geopolitics & Defense

### [Story Headline]
**Source:** [Outlet] | **Date:** [Month DD, YYYY] | **Link:** [URL]
*Also reported by: [Outlet A], [Outlet B]* ← omit this line if no duplicates

[Paragraph 1: What happened.]

[Paragraph 2: Why it matters.]

[Paragraph 3: What to watch.] ← omit if no forward signal

**Key Takeaways:**
- [Bullet]
- [Bullet]
- [Bullet]

---

[Repeat story block for each story in this section]

---

## Energy & Resources

[Same structure]

---

## Markets & Macro

[Same structure]

---

## Emerging Markets

[Same structure]

---

## Source Coverage Summary

| Outlet | Method | Articles Reviewed | Stories Included | Status |
|--------|--------|------------------|-----------------|--------|
| Al Jazeera | Open RSS | N | M | OK |
| Bloomberg | Cookie curl | N | M | OK / Cookies expired |
| NYT | API | N | M | OK / API key missing |
| ... | | | | |

---
*End of brief. Next scheduled run: [Tomorrow's full date] at 06:03.*
```

---

## Troubleshooting Quick Reference

| Problem | What to do |
|---|---|
| RSS feed returns empty or error | Log `[Feed error]` in Source Coverage Table, continue |
| Bloomberg curl returns login page | Log `[Cookies expired]`, use search snippet, re-export cookies at next opportunity |
| NYT API returns 401 | Log `[API key invalid]`, skip NYT for this run, check `NYT_API_KEY` env var |
| Fewer than 2 articles in a topic section | Run the Catch-All WebSearch query for that topic from `resources/sources.md` |
| Email script fails | Append failure note to report footer, check `GMAIL_APP_PASSWORD` env var |
| Article pubDate missing | Keep article, use fetch time, mark `[Date unknown]` in Source Coverage Table |
