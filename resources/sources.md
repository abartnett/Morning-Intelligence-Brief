# News Sources Registry

Reference for the morning-brief workflow. Each outlet lists its fetch method, URL or query, and tier ranking used for deduplication.

**Tier ranking (for deduplication canonical selection):**
- **Tier 1:** FT, WSJ, Bloomberg, NYT, The Economist, Reuters, Foreign Affairs
- **Tier 2:** CFR, RAND, Brookings, CSIS, BBC, Al Jazeera, Foreign Policy
- **Tier 3:** SCMP, Nikkei Asia, Spiegel International, AP

---

## Method A — Subscriber RSS (WebFetch on feed URL)

These outlets provide subscriber-authenticated RSS feeds with full or extended article text. After logging in, paste your personalized feed URL in the table below. Instructions for finding each URL are in `resources/paywall-setup.md`.

| Outlet | Tier | Feed URL | Notes |
|--------|------|----------|-------|
| WSJ World News | 1 | `https://feeds.content.dowjones.io/public/rss/RSSWorldNews` | wsj.com/news/rss-news-and-feeds |
| WSJ Business | 1 | `https://feeds.content.dowjones.io/public/rss/WSJcomUSBusiness` | wsj.com/news/rss-news-and-feeds |
| Brookings (Intl Affairs) | 2 | `https://www.brookings.edu/topic/international-affairs/feed/` | Confirmed working Apr 2026 |
| BBC World | 2 | `https://feeds.bbci.co.uk/news/world/rss.xml` | Open RSS |
| Al Jazeera | 2 | `https://www.aljazeera.com/xml/rss/all.xml` | Open RSS |
| SCMP | 3 | `https://www.scmp.com/rss/91/feed` | Open RSS |
| Nikkei Asia | 3 | `https://asia.nikkei.com/rss/feed/nar` | Open RSS; pubDate sometimes absent — use fetch time |

--
## Method B — NYT Article Search API

Use `Bash curl` (not WebFetch — api.nytimes.com is blocked by WebFetch) with the URL below, substituting `[TOPIC]` and the API key env var. Run once per topic bucket. Combine and deduplicate results by `web_url` before adding to working list.

**Endpoint:** `https://api.nytimes.com/svc/search/v2/articlesearch.json`  
**Env var required:** `NYT_API_KEY` (set in ~/.zshrc — see paywall-setup.md §2)  
**Note:** The `fq=section_name` filter is broken as of May 2026 (returns 0 results regardless of syntax). Use `begin_date` for 24-hour scoping instead.

| Topic bucket | Query string |
|---|---|
| GEOPOLITICS | `q=geopolitics+defense+military+sanctions+conflict&sort=newest&begin_date=YESTERDAY_YYYYMMDD` |
| ENERGY | `q=oil+gas+energy+minerals+OPEC+LNG+pipeline&sort=newest&begin_date=YESTERDAY_YYYYMMDD` |
| MARKETS | `q=Federal+Reserve+central+bank+inflation+GDP+markets&sort=newest&begin_date=YESTERDAY_YYYYMMDD` |
| EM | `q=China+India+Brazil+emerging+markets+sovereign&sort=newest&begin_date=YESTERDAY_YYYYMMDD` |

Replace `YESTERDAY_YYYYMMDD` with yesterday's date in YYYYMMDD format (e.g. `20260505`).  
Append `&api-key=$NYT_API_KEY` to each query. Extract from `response.docs[]`: `headline.main`, `web_url`, `pub_date`, `abstract`, `lead_paragraph`.

---

## Method C — WebSearch + WebFetch (Bloomberg, Reuters, Spiegel)

Bloomberg uses Cloudflare bot detection that blocks automated requests — treat as snippet-only. Reuters discontinued their native RSS in early 2026 and WebFetch to reuters.com is blocked; use WebSearch fallback (see Method D). Spiegel is open access and works with WebFetch.

| Outlet | Tier | Discovery query | Full text method |
|--------|------|-----------------|-----------------|
| Bloomberg | 1 | `site:bloomberg.com markets OR macro OR geopolitics OR energy OR "emerging markets" after:YESTERDAY` | Snippet only — log `[Snippet only — bot detection]` |
| Spiegel Int'l | 3 | `site:spiegel.de/international geopolitics OR Europe OR NATO OR defense OR energy after:YESTERDAY` | WebFetch (open access) |

---

## Method D — WebSearch Only (headline + snippet)

For open-access outlets where RSS is broken or unavailable. Use `WebFetch` on the article URL for full text (these sites are unpaywalled); fall back to snippet if WebFetch fails.

| Outlet | Tier | Discovery query | Notes |
|--------|------|-----------------|-------|
| CFR | 2 | `site:cfr.org geopolitics OR defense OR energy OR "emerging markets" after:YESTERDAY` | Open access |
| RAND Corporation | 2 | `site:rand.org defense OR security OR geopolitics OR energy after:YESTERDAY` | Open access |
| Foreign Affairs | 2 | `site:foreignaffairs.com geopolitics OR defense OR international relations after:YESTERDAY` | Open access |
| CSIS | 2 | `site:csis.org geopolitics OR defense OR energy OR Iran after:YESTERDAY` | Open access; RSS dead (returns 2016 content) |
| Foreign Policy | 2 | `site:foreignpolicy.com geopolitics OR defense OR energy OR sanctions after:YESTERDAY` | Open access; RSS returns 403 |
| Reuters | 1 | `reuters geopolitics OR energy OR markets OR defense after:YESTERDAY` | No site: operator — RSS discontinued Mar 2026, WebFetch blocked |

---

## Catch-All Queries (per topic — use if a section has fewer than 3 articles)

Run these only if the section is thin after all sources above are processed.

| Topic | Catch-all WebSearch query |
|---|---|
| GEOPOLITICS | `geopolitics defense military conflict sanctions after:YESTERDAY` |
| ENERGY | `oil gas energy minerals OPEC pipeline after:YESTERDAY` |
| MARKETS | `central bank inflation GDP recession markets after:YESTERDAY` |
| EM | `"emerging markets" China India Southeast Asia after:YESTERDAY` |
