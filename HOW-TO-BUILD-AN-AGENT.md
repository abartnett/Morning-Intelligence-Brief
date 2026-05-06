# How to Build a Claude Code Agent
## A Step-by-Step Guide Based on Agent 1: The Morning Intelligence Brief

---

## What This Guide Covers

This document explains how to build a fully automated Claude Code agent from scratch — one that runs on a schedule, fetches data from multiple sources, applies structured logic, produces a formatted output file, and delivers it without human intervention.

Agent 1 is a morning intelligence brief that runs at 6:03 AM daily. It scans 14+ news outlets, deduplicates cross-outlet coverage, summarizes stories by topic, fetches live market data, runs quality control, and emails a structured Markdown report. Everything in this guide is derived from how that agent was actually built, including the mistakes made and how they were fixed.

---

## Table of Contents

1. [The Core Concept: What Is a Claude Code Agent?](#1-the-core-concept)
2. [Project Structure](#2-project-structure)
3. [CLAUDE.md: Teaching Claude About Your Project](#3-claudemd)
4. [Plan Mode: Design Before You Build](#4-plan-mode)
5. [Workflow Files: The Agent's Brain](#5-workflow-files)
6. [Resource Files: Reference Data the Agent Reads](#6-resource-files)
7. [Skill Files: Custom Commands](#7-skill-files)
8. [Settings.json: Pre-Authorizing Tools for Unattended Runs](#8-settingsjson)
9. [API Integration](#9-api-integration)
10. [RSS Feed Fetching](#10-rss-feed-fetching)
11. [External Scripts: Keeping Credentials Out of the Project](#11-external-scripts)
12. [Scheduling with CronCreate](#12-scheduling-with-croncreate)
13. [Iterative Training: How to Improve the Agent Over Time](#13-iterative-training)
14. [Debugging: What Broke and Why](#14-debugging)
15. [What to Do Differently: Efficiency and Performance Improvements](#15-what-to-do-differently)
16. [Full Build Checklist](#16-full-build-checklist)

---

## 1. The Core Concept

### What is a Claude Code agent?

A Claude Code agent is a set of plain-English instruction files that Claude reads and executes autonomously — fetching data, processing it, making decisions, writing output, and taking actions like sending emails or running scripts. You are not writing code in the traditional sense. You are writing **recipes** that Claude follows.

The key insight: Claude is the runtime. Your files are the program.

### What makes a good agent candidate?

A task is a good fit for a Claude Code agent if it:

- Runs on a repeating schedule
- Pulls from multiple data sources using different methods
- Requires judgment calls that are too complex to encode in rigid code (e.g., "is this article relevant to geopolitics?")
- Produces structured output that humans consume
- Has a clear success condition you can check

### What Claude Code provides

| Capability | Tool |
|---|---|
| Read and write files | `Read`, `Write`, `Edit` |
| Fetch web pages and RSS feeds | `WebFetch` |
| Search the web | `WebSearch` |
| Run terminal commands, curl, Python | `Bash` |
| Schedule recurring tasks | `CronCreate` |
| Run quality-control passes | Custom skill files |
| Send email, interact with APIs | `Bash` + external scripts |

---

## 2. Project Structure

Create this directory structure before writing a single line of content. The structure is the architecture.

```
Agent 1/
├── CLAUDE.md                    ← Project context Claude reads at session start
├── HOW-TO-BUILD-AN-AGENT.md    ← This file
├── .claude/
│   ├── settings.json            ← Tool permissions for unattended runs
│   └── commands/
│       └── qc-brief.md          ← Custom slash command (quality control)
├── workflows/
│   └── morning-brief.md         ← The main recipe Claude follows
├── resources/
│   ├── sources.md               ← Outlet registry with fetch methods and URLs
│   ├── topic-criteria.md        ← Relevance filter rules per topic
│   └── paywall-setup.md         ← One-time credential setup instructions
└── output/
    └── (generated daily)        ← YYYY-MM-DD-morning-brief.md

~/scripts/                       ← Lives OUTSIDE the project (credentials)
└── send_morning_brief.py        ← Email delivery script
```

### Why this structure?

- **`workflows/`** contains the executable recipes. Claude reads these and follows them step by step. Keeping them separate from resources makes the logic easy to find and edit.
- **`resources/`** contains reference data — things Claude looks up while executing, not things it follows. A clear separation between "instructions" and "reference data" prevents confusion during execution.
- **`.claude/commands/`** holds custom slash commands (skill files). These are modular quality-control or utility routines Claude can invoke during a workflow.
- **`output/`** is the only directory Claude writes to during a run. Everything else is read-only from Claude's perspective.
- **`~/scripts/`** is deliberately outside the project. Scripts that handle credentials (email passwords, API keys) should never live inside a project directory that could be version-controlled or shared.

---

## 3. CLAUDE.md

`CLAUDE.md` is the most important file in your project. Claude reads it at the start of every session. It establishes who you are, what the project is, and what rules Claude must follow.

### What to put in CLAUDE.md

```markdown
# Project Context

[One paragraph describing what this project does and why it exists.]

# About Me

[Who you are, your background, what level of detail and terminology you expect.]

# Rules

- [Behavioral rules Claude must follow in this project]
- [e.g., "Always ask clarifying questions before starting a complex task"]
- [e.g., "Save all output files to the output/ folder"]
- [e.g., "Cite sources when doing research"]

# Project Structure

- workflows/ - [what's in here]
- output/ - [what's in here]
- resources/ - [what's in here]
```

### Agent 1's CLAUDE.md

```markdown
# Project Context

This is my AI agent workspace. I use it for research, productivity, and education workflows.

# About Me

I am an MBA student with a background in military intelligence. I am working to learn about
how business, investment, and geopolitics overlap to drive business outcomes that are
profitable and create geopolitical security around the globe. I prefer fact based, clear,
and detailed output.

# Rules

- Always ask clarifying questions before starting a complex task
- Show your plan and steps before executing
- Save all output files to the output folder
- Cite sources when doing research

# Project Structure

- workflows/ - Workflow instruction files (plain English recipes the agent follows)
- output/ - Finished deliverables (reports, drafts, analysis)
- resources/ - Reference docs and templates
```

### Key principles for CLAUDE.md

1. **Be specific about your background.** Claude tailors explanations, depth of analysis, and terminology to who you say you are. An MBA student with military intelligence background will get different output than a software engineer.
2. **Keep rules short and imperative.** One rule per bullet. No ambiguity.
3. **Do not put workflow instructions here.** CLAUDE.md sets context; it does not direct execution. Execution logic goes in `workflows/`.

---

## 4. Plan Mode

Plan Mode is Claude's architecture design mode. Use it before writing any workflow or resource file. In Plan Mode, Claude will design the full implementation — step by step — and ask for your approval before touching any files.

### How to enter Plan Mode

Type `/plan` in Claude Code, or frame your request as: *"I want to build X. Show me the plan before doing anything."*

### What Plan Mode produces

A structured implementation document covering:
- What needs to be built and why
- Which files need to be created or modified
- Exact content for each change
- Architectural tradeoffs and decisions

### How Agent 1 used Plan Mode

The initial architecture was designed in Plan Mode. The plan covered:
- The full directory structure
- The workflow step sequence (11 steps)
- The deduplication algorithm
- The source registry format
- The topic classification criteria
- The email delivery approach
- The scheduling method

The plan was reviewed, a change was requested (sector-level impact forecast instead of stock-level), the plan was revised, and only then was execution approved.

### Plan Mode best practices

- **Use Plan Mode for anything that touches more than 2 files.** If Claude will be creating a new workflow, adding a major section, or restructuring resources, make it show you the plan first.
- **Request changes to the plan before approving.** This is the lowest-cost moment to change direction. Revising a plan takes seconds; revising already-written workflow files takes minutes and risks introducing inconsistencies.
- **Save the plan.** Claude Code can save plan files that persist across sessions. Use this for complex multi-session builds.
- **Approve the plan explicitly.** Say "yes" or "proceed" to exit Plan Mode. Claude will not execute without approval.

---

## 5. Workflow Files

The workflow file is the agent's brain. It is a plain-English step-by-step recipe that Claude reads and executes autonomously. It is not code — it is structured prose that Claude interprets as instructions.

### Anatomy of a workflow file

```markdown
# Workflow: [Name]
Version: [X.X]
Schedule: [When it runs]
Output: [Where output goes]

---

## Before You Begin

[List any resource files Claude must load before starting.]

---

## Step 1 — [Step Name]

[Plain English instructions for this step. Be explicit about:]
- What Claude should do
- What tools to use (WebFetch, Bash curl, WebSearch)
- What to do if something fails
- What output to keep and in what format

---

## Step 2 — [Next Step]
...
```

### Writing effective workflow steps

**Be explicit about tool choice.** Claude will pick the most convenient tool if you don't specify. Sometimes that's wrong. Example:

```
Bad:  "Fetch the NYT article search API."
Good: "Use Bash curl (not WebFetch — api.nytimes.com is blocked by WebFetch) with this 
       exact command: curl -s 'https://api.nytimes.com/...'
```

**Specify parallel vs. sequential explicitly.** Claude can run multiple tool calls simultaneously, which is faster but can cause cascade failures if one call crashes. Be explicit:

```
Bad:  "Run these queries."
Good: "Run these four WebSearch queries in parallel in a single batch. 
       Run all NYT Bash curl calls in a separate parallel batch. 
       Never mix Bash and WebSearch in the same parallel batch."
```

**Define failure handling per step.** An unattended agent will hit errors. Tell Claude exactly what to do:

```
If any curl returns an error or empty result, note [Data unavailable] for that 
instrument and continue. Do not abort the run.
```

**Use a "Before You Begin" section.** List every resource file Claude needs to load. Claude reads these into its context before executing, preventing mid-run failures when it reaches a step that references a resource it hasn't seen.

### Agent 1's workflow structure

```
Step 1    — Set up run (date variables, output path, run time)
Step 1.5  — Fetch markets data (Yahoo Finance curl, WebSearch)
Step 2    — Fetch articles from all sources (RSS, NYT API, WebSearch)
Step 3    — Apply 24-hour filter
Step 4    — Apply topic relevance filter
Step 5    — Deduplicate articles
Step 6    — Fetch full article text
Step 7    — Write summaries + sector impact forecast
Step 8    — Assemble report
Step 9    — Save report
Step 9.5  — Quality control (reads .claude/commands/qc-brief.md inline)
Step 10   — Send email
Step 11   — Confirm completion
```

### The report format template

Include a complete Markdown template at the bottom of your workflow file. This is the exact structure Claude must produce. Specify every section, every header level, every table column. Example:

```markdown
## Report Format Template

Use this exact Markdown structure. Do not add extra headers or reorder sections.

\`\`\`markdown
# Morning Intelligence Brief — [Full weekday, Month DD, YYYY]
*Generated by Claude Code | Run time: HH:MM | Articles reviewed: N | Stories included: M*

---

## Executive Summary
[250–350 words.]

---

## Markets Update
...
\`\`\`
```

Explicit templates eliminate formatting drift between runs.

### Editing workflow files

Follow these rules when modifying a workflow:

1. **Make surgical edits.** Change only the specific step or section that needs updating. Do not rewrite surrounding steps.
2. **Do not add explanatory commentary to the file.** The workflow is a recipe Claude executes, not documentation for humans. Extra commentary can confuse execution.
3. **Preserve step numbering and section order.** Claude references steps by number. Renumbering breaks cross-references.
4. **Test one step at a time when debugging.** Tell Claude to execute only Steps 1–3, for example, and confirm the output before proceeding.

---

## 6. Resource Files

Resource files are reference documents the agent reads during execution. They contain data, rules, and configuration — not instructions.

### sources.md

The outlet registry. For each news source, specify:
- **Outlet name** and tier ranking (used for deduplication priority)
- **Fetch method** (RSS WebFetch, API curl, WebSearch snippet)
- **URL or query pattern**
- **Known failure modes** (e.g., "bloomberg.com blocks WebFetch — snippet only")

```markdown
## Method A — Subscriber RSS (WebFetch on feed URL)

| Outlet | Tier | Feed URL | Notes |
|--------|------|----------|-------|
| WSJ World News | 1 | https://feeds.content.dowjones.io/public/rss/RSSWorldNews | |
| BBC World | 2 | https://feeds.bbci.co.uk/news/world/rss.xml | Open RSS |
| Al Jazeera | 2 | https://www.aljazeera.com/xml/rss/all.xml | Open RSS |

## Method B — API (Bash curl)

| Outlet | Endpoint | Query patterns |
|--------|----------|----------------|
| NYT | https://api.nytimes.com/svc/search/v2/articlesearch.json | q=geopolitics... |

## Method C — WebSearch (snippet only)

| Outlet | Discovery query |
|--------|-----------------|
| Bloomberg | site:bloomberg.com markets OR geopolitics after:YESTERDAY |
```

**Critical: document every known limitation.** If an outlet blocks WebFetch, blocks curl, requires cookies, or needs a special search operator, document it here. An agent running unattended at 6 AM cannot ask you for help.

### topic-criteria.md

The relevance classification rules. For each topic bucket, specify:
- **Keep if:** specific keywords, event types, actor categories
- **Discard (NOISE) if:** specific exclusions
- **Borderline cases:** decision rules for ambiguous articles

```markdown
## GEOPOLITICS — Keep if:
- Military operations: ground offensives, airstrikes, naval maneuvers
- Defense procurement: weapons contracts, arms sales
- Alliance activity: NATO decisions, bilateral security agreements
...

## GEOPOLITICS — Discard (NOISE) if:
- Domestic crime with no foreign policy dimension
- Sports, entertainment, celebrity
...
```

The more specific these rules, the more consistent the agent's classification will be across runs.

### paywall-setup.md

A one-time setup guide for credentials and authenticated access. This file is read by humans, not executed by Claude. It should cover:
- How to find subscriber RSS feed URLs for each outlet
- How to get an NYT API key (free, takes 5 minutes)
- How to export browser cookies for paywalled sites
- How to set environment variables in `~/.zshrc`
- How to test each credential before scheduling

---

## 7. Skill Files

Skill files are modular routines stored in `.claude/commands/`. They are the equivalent of functions or subroutines — self-contained procedures Claude can be told to execute during a workflow.

### Location and naming

```
.claude/
└── commands/
    └── qc-brief.md        ← The QC skill for Agent 1
    └── your-skill.md      ← Any other custom skill
```

### Structure of a skill file

```markdown
# [Skill Name]

[One paragraph description of what this skill does and when it runs.]

---

## Step 1 — [First action]
[Instructions]

## Step 2 — [Second action]
[Instructions]

...

## Step N — Always Proceed
[If this skill is called in an automated context, end with an explicit 
instruction to continue regardless of findings — never block execution.]
```

### Agent 1's QC skill (qc-brief.md)

The QC skill runs as Step 9.5 — after the report is written but before the email is sent. It:

1. Re-fetches primary sources to verify the brief is accurate
2. Runs six checks (executive summary consistency, conflict party completeness, numerical claim verification, causal framing accuracy, unverifiable claims, scope integrity)
3. Appends a QC FLAGS section to the report file with any findings
4. Always proceeds to send — never blocks delivery

The "always proceed" design is critical for unattended execution. A skill that blocks and waits for human review defeats the purpose of automation. Instead, it surfaces findings in the output so the reader can see them when the email arrives.

### How to invoke a skill in a workflow

**Important:** The Skill tool (`/skill-name`) only recognizes built-in Claude Code skills. Custom `.claude/commands/` files are NOT invokable via the Skill tool.

The correct way to invoke a custom skill in a workflow is:

```
## Step 9.5 — Quality Control

Execute the QC protocol defined in `.claude/commands/qc-brief.md` directly — 
read that file and follow each step inline. Do not use the Skill tool.
```

This is a non-obvious but critical distinction. If your workflow says `Run /qc-brief`, it will fail with "Unknown skill" in an unattended run. Always use the "read and execute inline" pattern for custom skills.

### When to create a skill vs. putting logic in the workflow

Create a skill when:
- The procedure is reusable across multiple workflows
- The procedure is long enough to clutter the main workflow
- The procedure has a clear, testable success condition
- You want to invoke it manually outside the scheduled run

Keep logic in the workflow when:
- It only applies to one workflow
- It is sequential and tightly coupled to surrounding steps
- It accesses variables set earlier in the workflow

---

## 8. Settings.json

`settings.json` is the permission registry for unattended execution. Every tool call that Claude makes during an automated run must be pre-authorized here — or the run will pause and wait for a human to approve it.

### Location

```
.claude/
└── settings.json
```

### Format

```json
{
  "permissions": {
    "allow": [
      "Write",
      "Edit",
      "WebFetch",
      "WebSearch",
      "ToolSearch",
      "Bash(source ~/.zshrc*)",
      "Bash(curl -s *)",
      "Bash(python3 ~/scripts/send_morning_brief.py *)"
    ]
  }
}
```

### Permission pattern syntax

| Pattern | What it allows |
|---------|---------------|
| `"Write"` | All Write tool calls |
| `"WebFetch"` | All WebFetch calls |
| `"Bash(curl -s *)"` | Any Bash command starting with `curl -s` |
| `"Bash(python3 ~/scripts/send_morning_brief.py *)"` | Only that specific script |
| `"ToolSearch"` | The meta-tool that loads deferred tool schemas |

**Use the narrowest pattern that covers your actual usage.** A broad pattern like `"Bash(*)"` would authorize arbitrary shell execution — never do this.

### The ToolSearch entry is mandatory

`WebSearch` and `WebFetch` are "deferred" tools in Claude Code — their parameter schemas are not loaded at session start. Before Claude can call them, it must invoke `ToolSearch` to load their schemas. If `ToolSearch` is not in the allowlist, this step will pause for approval and the unattended run will hang.

Add `"ToolSearch"` to your allowlist from day one.

### The Write and Edit entries are mandatory

If your workflow saves a report file (Step 9) or appends a QC section to an existing file (Step 9.5), both `Write` and `Edit` must be authorized. These are easy to forget because they seem obvious — but Claude will pause and ask for approval if they're absent.

### How to audit your allowlist

After building the workflow, read through every step and ask: *"What tool will Claude call here?"* For every tool call:
- Is it in the allowlist? If not, add it.
- Is the Bash pattern specific enough to cover the actual command? If not, adjust it.

Also run the `/fewer-permission-prompts` skill in Claude Code — it analyzes your session transcripts and suggests allowlist entries based on actual tool usage patterns.

---

## 9. API Integration

### The NYT Article Search API

The NYT API provides full article text, structured metadata, and no paywall — free to use with an API key from developer.nytimes.com.

**Setup:**
1. Go to developer.nytimes.com → Create Account → My Apps → New App → enable "Article Search API"
2. Copy the API key
3. Add to `~/.zshrc`: `export NYT_API_KEY="your-key-here"`
4. Run `source ~/.zshrc`

**Key implementation lessons:**

Use Bash curl, not WebFetch. `api.nytimes.com` is blocked by WebFetch in Claude Code. Always use:

```bash
source ~/.zshrc && curl -s "https://api.nytimes.com/svc/search/v2/articlesearch.json?q=geopolitics&sort=newest&api-key=$NYT_API_KEY"
```

Use safe Python parsing. The API returns documents where `headline` can be `None`. A naive parser will crash:

```python
# WRONG — crashes if headline is None
print('TITLE:', d['headline']['main'])

# CORRECT — safe at both levels
print('TITLE:', (d.get('headline') or {}).get('main', '[no title]'))
```

Run topic queries individually, not in parallel if failures cascade. If one Bash call in a parallel batch crashes, Claude cancels all other calls in that batch. The safe approach:

```
Run each of the four topic queries sequentially, one at a time. 
Do not run them in parallel.
```

**Query structure for the brief:**
- GEOPOLITICS: `q=geopolitics+defense+military+sanctions&sort=newest`
- ENERGY: `q=oil+gas+energy+OPEC+LNG&sort=newest`
- MARKETS: `q=Federal+Reserve+central+bank+inflation+GDP&sort=newest`
- EM: `q=China+India+emerging+markets&sort=newest`

### The Yahoo Finance Unofficial Quote API

Yahoo Finance provides live price data through an unofficial (undocumented) endpoint. No API key required.

**Endpoint pattern:**
```bash
curl -s "https://query1.finance.yahoo.com/v8/finance/chart/[TICKER]?interval=1m&range=1d"
```

**Ticker encoding:**

| Instrument | Ticker (URL-encoded) |
|---|---|
| S&P 500 Futures | `ES%3DF` |
| Nasdaq Futures | `NQ%3DF` |
| DOW Futures | `YM%3DF` |
| Gold | `GC%3DF` |
| WTI Crude Oil | `CL%3DF` |
| 10-Year Treasury | `%5ETNX` |
| Natural Gas | `NG%3DF` |

**Response parsing:**
```python
import json, sys
data = json.loads(sys.stdin.read())
meta = data['chart']['result'][0]['meta']
price = meta['regularMarketPrice']
change_pct = meta.get('regularMarketChangePercent', 'N/A')
```

**Critical: run sequentially with delays.** Yahoo Finance rate-limits simultaneous requests. Running 7 calls in parallel returns "Too Many Requests" for all of them. The correct approach:

```bash
source ~/.zshrc && curl -s "[URL_1]" || (sleep 3 && curl -s "[URL_1]")
sleep 3 && source ~/.zshrc && curl -s "[URL_2]" || (sleep 3 && curl -s "[URL_2]")
# ... and so on
```

The `||` retry pattern handles transient failures. The 3-second sleep between calls avoids rate limiting. Total time for 7 calls: approximately 25 seconds.

### Environment variables and credentials

Never put API keys or passwords directly in workflow files or resource files. Always:
1. Store credentials in `~/.zshrc` as environment variables
2. Load them with `source ~/.zshrc &&` at the start of each Bash command
3. Reference them as `$VARIABLE_NAME` in curl commands

```bash
# In ~/.zshrc:
export NYT_API_KEY="your-key-here"
export GMAIL_APP_PASSWORD="xxxx xxxx xxxx xxxx"

# In workflow:
source ~/.zshrc && curl -s "...?api-key=$NYT_API_KEY"
```

---

## 10. RSS Feed Fetching

RSS feeds are the cleanest data source for news agents. They return structured XML with titles, URLs, publication dates, and often full article text — no HTML parsing required.

### Fetching a feed

Use `WebFetch` on the feed URL with a parsing prompt:

```
WebFetch(
  url: "https://feeds.bbci.co.uk/news/world/rss.xml",
  prompt: "Extract all article items from this RSS feed. For each item return: 
           title, link/URL, pubDate, and description/snippet (first 3 sentences). 
           Return as a numbered list."
)
```

### Finding RSS feed URLs

- **Open RSS feeds:** Usually at `/rss.xml`, `/feed`, `/feed.xml`, or discoverable via the site's footer or `/about` page
- **Subscriber RSS feeds:** Log in to your account, look for an RSS icon or link in account settings. WSJ, FT, The Economist all provide subscriber-authenticated feeds with full article text
- **Broken feeds:** Test every URL before including it in sources.md. Some outlets (CSIS, Foreign Policy) have RSS feeds that return old content or 403 errors — document this and use WebSearch as the fallback

### Handling common RSS failures

| Problem | Solution |
|---------|----------|
| Feed returns 403 | Switch to WebSearch discovery with `site:domain.com` |
| Feed returns empty XML | Log `[Feed error]` and continue — do not abort |
| pubDate absent | Keep the article, use fetch time, note `[Date unknown]` |
| Feed returns full HTML articles | Use the full text directly — no need to re-fetch |
| Feed returns only headlines | Must re-fetch article URLs for full text |

### Subscriber vs. open RSS

Subscriber RSS feeds (WSJ, FT, Economist) require you to be logged in to generate the personalized feed URL. These URLs contain a token unique to your account. Treat them like passwords — do not commit them to version control. Store them in `resources/sources.md` which should be in your `.gitignore`.

Open RSS feeds (BBC, Al Jazeera, SCMP) are public and require no authentication.

---

## 11. External Scripts

Some agent tasks — particularly those involving credentials like email passwords — should be handled by external scripts living outside the project directory.

### Why external?

- Scripts in `~/scripts/` are not in the project repository
- Credentials stay out of version control
- The script can be tested independently of the agent
- The workflow simply calls the script with a file path argument

### The email delivery script

Agent 1's email script (`~/scripts/send_morning_brief.py`) uses Gmail SMTP with an app password. This is the simplest reliable email delivery method that works from a terminal script.

**Gmail App Password setup (one-time):**
1. `myaccount.google.com` → Security → enable 2-Step Verification
2. Search "App passwords" → create one named `morning-brief` → copy the 16-character password
3. Add to `~/.zshrc`: `export GMAIL_APP_PASSWORD="xxxx xxxx xxxx xxxx"`
4. `source ~/.zshrc`

**Script structure:**

```python
import smtplib, os, sys
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from datetime import date

def send_brief(filepath):
    with open(filepath, 'r') as f:
        body = f.read()
    
    msg = MIMEMultipart()
    msg['From'] = 'your.email@gmail.com'
    msg['To'] = 'your.email@gmail.com'
    msg['Subject'] = f'Morning Intelligence Brief — {date.today()}'
    msg.attach(MIMEText(body, 'plain'))
    
    password = os.environ.get('GMAIL_APP_PASSWORD')
    if not password:
        print("ERROR: GMAIL_APP_PASSWORD not set")
        sys.exit(1)
    
    with smtplib.SMTP('smtp.gmail.com', 587) as server:
        server.starttls()
        server.login('your.email@gmail.com', password)
        server.send_message(msg)
    
    print(f"Email sent: {msg['Subject']} → {msg['To']}")

if __name__ == '__main__':
    send_brief(sys.argv[1])
```

**Calling it from the workflow:**
```
python3 ~/scripts/send_morning_brief.py "output/RUN_DATE-morning-brief.md"
```

The full path in `settings.json` (`"Bash(python3 ~/scripts/send_morning_brief.py *)"`) ensures only this specific script is pre-authorized — not arbitrary Python execution.

---

## 12. Scheduling with CronCreate

CronCreate is Claude Code's built-in scheduler. It fires Claude at a specified cron time and passes a prompt for Claude to execute.

### Setting up the schedule

Tell Claude:

```
Set up a daily cron job using CronCreate to run the morning brief workflow. 
Cron: 3 6 * * * (6:03 AM daily — off the hour to avoid fleet congestion).
Recurring: true. Durable: true.
The prompt should be: "Execute workflows/morning-brief.md completely for today's date. 
Save the report to output/ and send email when done."
```

### Critical limitations

**CronCreate jobs are session-scoped.** Despite the `durable: true` flag, scheduled jobs expire when Claude Code's session infrastructure resets — typically every 7 days. You must recreate the cron job periodically. Set a calendar reminder for day 6 to renew it.

**The computer must be awake.** Claude Code processes run on your local machine. macOS auto-wake (System Settings → Battery → Schedule) will wake the machine at the specified time, and the cron will fire correctly even if the screen is locked. But if the machine is powered off or Claude Code is not running, the job will be skipped.

**Claude Code must already be open.** The cron fires within an existing Claude Code session — it does not launch Claude Code from scratch. Make sure Claude Code is running before your computer goes to sleep.

### Verifying the schedule

After setting up the cron, ask Claude to run `CronList` and confirm the job appears with the correct schedule and is active.

### Why 6:03 and not 6:00?

Off-the-minute scheduling avoids fleet congestion on Claude's infrastructure. Round-minute times (6:00, 7:00) are the most popular cron times and can experience latency when many jobs fire simultaneously. 6:03 is a cheap way to improve reliability.

---

## 13. Iterative Training

An agent is not built in one session. It is built through repeated runs, observation, and refinement. Here is how to approach each iteration.

### The build sequence

```
Session 1: Design in Plan Mode → build skeleton → partial test
Session 2: Fix failures → add missing source handling → full dry run
Session 3: Add features (Markets Update, QC skill) → test → fix
Session 4: Automate remaining manual steps → schedule
Ongoing:   Fix source failures as they occur, tune criteria, add sources
```

### After each run, ask these questions

1. **Were there any steps where I had to approve something?** These are automation blockers. Fix each one before the next run.
2. **Were there sources that returned no articles?** Diagnose whether it's a timing issue, a feed error, or a method problem (WebFetch blocked, cookie expired, API key invalid).
3. **Were any classified topic buckets wrong?** Update `topic-criteria.md` to be more specific.
4. **Were any summaries inaccurate or shallow?** Improve the Step 7 instructions for what good summaries look like.
5. **Was the deduplication too aggressive or not aggressive enough?** Tune the Phase 2 confirmation thresholds.

### Improving source coverage over time

Start with the easiest sources (open RSS feeds) and confirm they work before adding harder ones (subscriber RSS, API calls, cookie-authenticated curl). The order:

1. Open RSS (BBC, Al Jazeera) — works immediately
2. NYT API — 10-minute setup, then reliable
3. Subscriber RSS (WSJ, FT) — requires logged-in account URLs
4. WebSearch supplemental (CFR, RAND, Foreign Policy) — works but snippet-only
5. Cookie-authenticated (Bloomberg) — requires periodic re-export, snippet-only if Cloudflare blocks

### Tracking what works

Keep a running log in `resources/sources.md` of which sources work reliably, which need attention, and which are permanently degraded. Examples from Agent 1:

- Bloomberg: snippet-only permanently — Cloudflare blocks all automated curl
- Reuters: no `site:` operator in WebSearch (returns zero results) — use broad query instead
- CSIS: RSS feed returns 2016 content — use WebSearch instead
- Foreign Policy: RSS returns 403 — use WebSearch instead

### Using memory across sessions

Claude Code has a persistent memory system at `~/.claude/projects/[project-path]/memory/`. Across sessions, Claude can recall:
- **User profile** — your background, preferences, watchlist tickers
- **Project state** — current workflow version, known source failures
- **Feedback** — what editing patterns you prefer, what behaviors to avoid

Build memory by telling Claude things like: *"Remember that Bloomberg is snippet-only — Cloudflare blocks all curl."* Claude will write this to memory and recall it in future sessions without you having to explain it again.

---

## 14. Debugging

### The parallel batch cascade failure

**Problem:** When running multiple Bash calls in a parallel batch, if the first call fails (Python crash, network error, malformed response), Claude cancels all remaining calls in the batch.

**Why it happens:** Claude's parallel execution model treats a batch as a unit. An error in one branch terminates the batch.

**Fix:**
- Separate Bash and WebSearch into different parallel batches — never mix them
- Run long Bash sequences that are interdependent sequentially, not in parallel
- Add error handling in Python scripts so a single bad document doesn't crash the whole parser
- Use `|| (sleep 3 && retry-command)` patterns in Bash for automatic retry

### The deferred tool problem

**Problem:** WebSearch and WebFetch don't work until their schemas are loaded via ToolSearch. If ToolSearch isn't in `settings.json`, an unattended run hangs at the first WebSearch or WebFetch call.

**Fix:** Add `"ToolSearch"` to `permissions.allow` in `.claude/settings.json`.

### The custom skill invocation failure

**Problem:** Workflow says `Run /qc-brief` — Claude responds "Unknown skill: qc-brief" and either fails or skips the step.

**Why:** The Skill tool only recognizes built-in Claude Code skills. `.claude/commands/` files are not registered with the Skill tool.

**Fix:** Change the workflow instruction to: *"Read `.claude/commands/qc-brief.md` and execute each step inline. Do not use the Skill tool."*

### API key not in session

**Problem:** `curl` to NYT API returns `{"fault":{"faultstring":"Failed to resolve API Key variable..."}}`.

**Why:** `$NYT_API_KEY` is defined in `~/.zshrc` but not loaded in the current shell session.

**Fix:** Always prefix Bash commands with `source ~/.zshrc &&`. This ensures environment variables are available regardless of how the shell session was started.

### RSS feed returns old content

**Problem:** An outlet's RSS feed consistently returns articles from weeks or months ago.

**Why:** Some outlets (CSIS, Brookings for certain feeds) have RSS feeds that are infrequently updated or broken.

**Fix:** Document the outlet as "RSS dead" in `resources/sources.md` and switch to `WebSearch site:[domain]` for that outlet.

### Rate limiting on financial APIs

**Problem:** Running multiple simultaneous `curl` calls to Yahoo Finance returns "Too Many Requests" for all of them.

**Fix:** Run calls sequentially with 3-second sleep delays between each. Add a `|| (sleep 3 && retry)` pattern for automatic retry on transient failures.

---

## 15. What to Do Differently

If building this agent from scratch today, here is what should be done differently for better reliability and performance.

### Do from day one

**Add ToolSearch to settings.json immediately.** This is not obvious but critical. Do it before your first run.

**Test email delivery before building anything else.** Run `python3 ~/scripts/send_morning_brief.py output/test-brief.md` with a dummy file before writing a single workflow step. If email doesn't work, the entire agent is useless. Fix the hardest dependency first.

**Test each source individually before combining.** Run the workflow for a single source (just BBC RSS) and confirm the fetch → filter → summarize pipeline works before adding all 14 sources. Combining sources before the pipeline is validated makes debugging much harder.

**Set up a dry-run mode.** Add a flag or a separate workflow file (`workflows/morning-brief-test.md`) that runs the full pipeline but skips the email step. Use this for testing without sending to your inbox.

### Structural improvements

**Use a config section at the top of the workflow** for variables that change often:

```markdown
## Configuration

- RUN_DATE: [Set dynamically in Step 1]
- YESTERDAY: [Set dynamically in Step 1]  
- OUTPUT_DIR: output/
- EMAIL_TO: abartnett@gmail.com
- WATCHLIST_TICKERS: IONQ, QBTS, RGTI, PL, VRT
- LOOKBACK_HOURS: 24
```

This makes it easy to change the email address, add a ticker, or adjust the lookback window without hunting through the workflow prose.

**Add a source health log.** After each run, have the agent append a one-line status entry to `resources/source-health-log.md`:

```
2026-04-25 | WSJ:OK | BBC:OK | Bloomberg:SNIPPET | NYT:OK | Nikkei:DATES_MISSING
```

After a few weeks this log will show you which sources are unreliable and need attention.

**Use a dedicated search API for WebSearch.** Claude's built-in WebSearch tool is convenient but returns unpredictable results — different queries return different result counts, and some queries return nothing even for major outlets. A dedicated search API (Serper, SerpAPI, or Brave Search API) provides consistent JSON responses with reliable result counts and `after:` date filtering. This would significantly improve article discovery for Methods C and D.

### Quality improvements

**Add a minimum article threshold check.** If a section has fewer than 2 articles after filtering, the agent should automatically run the catch-all WebSearch query for that topic before assembling the report. This is in the workflow's troubleshooting section but should be an explicit automatic step, not a troubleshooting fallback.

**Add source diversity tracking.** Track what percentage of included stories come from each outlet. If one outlet (WSJ) is providing 70% of stories because others are failing, that's a signal that source coverage is degraded.

**Improve the deduplication algorithm.** The current word-overlap approach (4+ words in common) is effective but coarse. A better approach: compute headline similarity using a simple TF-IDF or embedding approach via a Python script. This would catch more cross-outlet duplicates, especially when outlets use different phrasing for the same event.

**Store previous briefs for trend detection.** Add a step that reads the previous 3 briefs and flags any story that appeared in all three — these are ongoing situations that warrant a "Day N of..." context note rather than a fresh summary.

---

## 16. Full Build Checklist

Use this as your build sequence for a new agent.

### Phase 1: Setup (30 minutes)

- [ ] Create project directory structure (`workflows/`, `output/`, `resources/`, `.claude/commands/`)
- [ ] Write `CLAUDE.md` with project context, user profile, and rules
- [ ] Create `settings.json` with `ToolSearch`, `Write`, `Edit`, `WebFetch`, `WebSearch` in the allowlist
- [ ] Set environment variables in `~/.zshrc` (API keys, email password)
- [ ] Test email delivery with a dummy file before anything else

### Phase 2: Design (1-2 hours)

- [ ] Enter Plan Mode and describe the full agent behavior
- [ ] Review the plan — request any changes before approving
- [ ] Approve the plan and let Claude create all files

### Phase 3: Resource files (30 minutes)

- [ ] Write `resources/sources.md` — start with 3-4 easy sources (open RSS)
- [ ] Write `resources/topic-criteria.md` — define each topic bucket with keep/discard rules
- [ ] Write `resources/paywall-setup.md` — document every credential setup step
- [ ] Test each source individually: run `WebFetch` on each RSS URL and confirm you get articles

### Phase 4: Workflow file (varies)

- [ ] Write `workflows/[agent-name].md` with full step-by-step instructions
- [ ] Include a "Before You Begin" section listing resource files to load
- [ ] Include explicit tool specifications for every step (WebFetch vs. Bash curl vs. WebSearch)
- [ ] Include explicit failure handling for every step
- [ ] Include the complete report format template at the bottom
- [ ] Include a troubleshooting quick reference table

### Phase 5: Skills (30 minutes)

- [ ] Write `.claude/commands/qc-brief.md` or equivalent QC skill
- [ ] Ensure the skill ends with "always proceed — never block" instruction
- [ ] Update the workflow to invoke the skill by reading the file inline (not via Skill tool)

### Phase 6: Testing

- [ ] Run Steps 1-4 only with a single source — confirm fetch → filter works
- [ ] Run the full workflow for the first time — note every approval step
- [ ] Fix every approval step (add to `settings.json` or change the workflow instruction)
- [ ] Run the full workflow again — no approval steps should appear
- [ ] Check the email — confirm format, content, and delivery
- [ ] Run a second time the next day — confirm the run date logic works correctly

### Phase 7: Scheduling

- [ ] Ask Claude to set up the CronCreate schedule
- [ ] Verify with `CronList`
- [ ] Set a calendar reminder for Day 6 to renew the cron job
- [ ] Configure macOS auto-wake (System Settings → Battery → Schedule) for the run time
- [ ] Ensure Claude Code is running before your computer goes to sleep the night before

### Phase 8: Ongoing maintenance

- [ ] After each run, review for any steps that required approval or produced errors
- [ ] Fix automation blockers immediately — they will recur every run
- [ ] Update `resources/sources.md` as feeds break or move
- [ ] Tune `resources/topic-criteria.md` as you notice misclassifications
- [ ] Renew CronCreate job every 6-7 days
- [ ] Re-export browser cookies for paywalled sites every 30-60 days

---

*Built from the ground up through Agent 1: Morning Intelligence Brief | April 2026*
