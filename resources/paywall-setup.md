# Paywall & Credentials Setup

One-time setup required before the morning-brief workflow can run. Complete all four sections below, then run the test procedure at the bottom to confirm everything works.

---

## §1 — Subscriber RSS URLs (WSJ, FT, The Economist)

These outlets provide subscriber-only RSS feeds with full article text. You must be logged in on their website to access the authenticated feed URL.

### Wall Street Journal
1. Log in at wsj.com.
2. Navigate to: wsj.com/news/rss-news-and-feeds
3. On that page, find the subscriber RSS feeds. Look for "World News" and "Business" under the subscriber section (they differ from the public feeds in that they include full article text).
4. Copy each feed URL.
5. Open `resources/sources.md` and paste the URLs into the WSJ World News and WSJ Business rows under Method A.

### Financial Times
1. Log in at ft.com.
2. Navigate to: ft.com/myft/following
3. Look for an RSS icon in your browser's address bar, or find the RSS link on the myFT page.
4. The subscriber feed URL format is: `https://www.ft.com/myft/following/[your-token]/rss`
5. Copy the full URL and paste it into the FT row in `resources/sources.md`.

**Alternative for FT:** If the myFT RSS does not include full article text, use the cookie method (§3 below) for FT instead, and leave the FT subscriber RSS row blank.

### The Economist
1. Log in at economist.com.
2. Navigate to your account settings page.
3. Look for an "RSS" or "Feeds" section. Copy the subscriber feed URL.
4. Paste it into the Economist row in `resources/sources.md`.

**Note:** If any of these outlets no longer offer subscriber RSS, leave the URL field as-is (with the placeholder text). The workflow will fall back to WebSearch for that outlet and log `[Subscriber RSS not configured]` in the Source Coverage Table.

---

## §2 — NYT Article Search API Key (Free)

The NYT API provides full article text for New York Times content with no paywall. The free tier allows 10 requests per minute and 4,000 per day — more than sufficient for this workflow (which makes 4 calls per run).

### Setup steps:
1. Go to: developer.nytimes.com
2. Create an account (or sign in with your existing NYT account).
3. Navigate to: My Apps → + New App
4. Give the app a name (e.g., `morning-brief`).
5. Under "APIs," enable **Article Search API**.
6. Click Save. Your API key is displayed on the app detail page — copy it.
7. Open your shell profile: `nano ~/.zshrc`
8. Add this line at the bottom: `export NYT_API_KEY="your-api-key-here"`
9. Save and close. Then run: `source ~/.zshrc`
10. Confirm it's set: `echo $NYT_API_KEY` — you should see the key printed.

### Test the API:
Run this command (replace the key value or just ensure the env var is set):
```
curl -s "https://api.nytimes.com/svc/search/v2/articlesearch.json?q=geopolitics&sort=newest&api-key=$NYT_API_KEY" | python3 -c "import sys,json; d=json.load(sys.stdin); print(d['response']['metadata']['hits'], 'results')"
```
You should see a number like `12847 results`. If you see an authentication error, re-check the key.

---

## §3 — Bloomberg Access Note

Bloomberg runs Cloudflare bot detection that blocks automated HTTP requests (including curl with valid cookies), because it checks for JavaScript execution and browser fingerprinting that only a real browser can provide. There is no simple workaround.

**What this means for the workflow:** Bloomberg articles will be fetched via WebSearch, which returns a headline and 2–3 sentence snippet per article. The workflow uses this snippet for classification, deduplication, and summarization, and notes `[Snippet only — bot detection]` in the Source Coverage Table. This is sufficient for the brief's purposes.

**If you want Bloomberg full text in the future**, the options are: (1) Bloomberg Terminal API (expensive, enterprise), or (2) a headless browser setup using Playwright (complex — requires additional software). Neither is needed to run the workflow.

---

## §4 — Gmail App Password (Email Delivery)

Gmail requires an "app password" (a separate 16-character credential) for SMTP access from scripts. This is different from your regular Google password.

### Prerequisites:
- 2-Step Verification must be enabled on your Google account. If it isn't, enable it first at myaccount.google.com → Security → 2-Step Verification.

### Setup steps:
1. Go to: myaccount.google.com → Security.
2. In the search bar at the top of the Security page, type **App passwords** and click the result (or navigate directly if you can find it in the menu).
3. Click **Create app password**.
4. In the "App name" field, type `morning-brief`. Click **Create**.
5. Google displays a 16-character password (shown only once). Copy it immediately — you cannot retrieve it later. If you lose it, you must create a new one.
6. Open your shell profile: `nano ~/.zshrc`
7. Add this line: `export GMAIL_APP_PASSWORD="xxxx xxxx xxxx xxxx"` (paste the 16-char password with spaces exactly as shown, or without spaces — both work).
8. Save and close. Run: `source ~/.zshrc`
9. Confirm: `echo $GMAIL_APP_PASSWORD` — you should see the password.

---

## Full Pre-Run Test Checklist

Run these checks in order before activating the daily schedule.

**[ ] 1. Directory structure exists**
```
ls -la "workflows/" "resources/" "resources/cookies/" "output/"
```
All four should exist.

**[ ] 2. NYT API key works**
```
curl -s "https://api.nytimes.com/svc/search/v2/articlesearch.json?q=geopolitics&api-key=$NYT_API_KEY" | python3 -c "import sys,json; print(json.load(sys.stdin)['response']['metadata']['hits'])"
```
Should print a number.

**[ ] 3. Bloomberg cookies work**
```
curl -s -L -b resources/cookies/bloomberg.txt "https://www.bloomberg.com" | grep -c "<article"
```
Should print 1 or more (meaning the page loaded with article elements, not a login wall).

**[ ] 4. Email script works**
```
echo "# Test Brief\nThis is a test." > output/test-brief.md
python3 ~/scripts/send_morning_brief.py output/test-brief.md
```
Check abartnett@gmail.com for the test email within 2 minutes.

**[ ] 5. Al Jazeera RSS feed test (partial workflow)**
Ask Claude: "Execute only Steps 1–4 of workflows/morning-brief.md using only the Al Jazeera RSS feed. Show me the working article list after the topic filter."
Verify that articles are fetched, filtered, and classified correctly.

**[ ] 6. Full dry run**
Ask Claude: "Execute workflows/morning-brief.md completely for today's date."
Review the output file. Check:
- All four topic sections have at least 2 articles
- No obvious duplicate stories survived deduplication
- Executive Summary is 250–350 words
- Source Coverage Table lists all outlets with their statuses
- Email was received at abartnett@gmail.com

**[ ] 7. Activate the schedule**
Once the dry run passes, ask Claude: "Set up the morning brief daily cron at 6:03 AM using CronCreate with durable:true and recurring:true."

**[ ] 8. Verify the schedule**
Ask Claude: "List all scheduled tasks (CronList)." Confirm the 6:03 AM job appears with the correct prompt. Note the expiry date — set a calendar reminder for 6 days from today to renew.

---

## Renewing the Cron Schedule

CronCreate jobs expire after 7 days. To renew:
1. Ask Claude: "List scheduled tasks (CronList)" — confirm the morning brief job is still listed.
2. If expired or about to expire, ask Claude: "Set up the morning brief daily cron at 6:03 AM using CronCreate with durable:true and recurring:true."

Recommended: Set a recurring weekly calendar reminder (every Sunday morning) to renew the cron.
