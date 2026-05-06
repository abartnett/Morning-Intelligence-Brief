# Morning Brief Quality Control

Runs automatically as Step 9.5 of the morning-brief workflow. Re-fetches key sources, checks six failure categories, fixes all issues directly in the brief body, then appends a QC Report summarising what was changed. Always proceeds to send. No human needs to be present.

---

## Step 1 — Identify the Brief

Find the most recently modified file in `output/` matching `YYYY-MM-DD-morning-brief.md`. Read the full file. Note the RUN_DATE from the filename.

---

## Step 2 — Re-Fetch Primary Sources

For each story in the brief, identify the cited source URL. Attempt `WebFetch` on each URL. If WebFetch is blocked, use `WebSearch` with the headline text to retrieve the article snippet. Work through Geopolitics first, then Energy, then Markets, then Emerging Markets.

Keep a list of which fetches succeeded and which failed — this goes in the QC Report.

---

## Step 3 — Run the Six Checks

Work through all six. Collect every finding. Do not stop early.

### CHECK A — Executive Summary vs. Story Body Consistency
**Severity: High**

Read each factual claim in the Executive Summary. Find the matching story in the brief body. Flag any case where the Executive Summary says the opposite, or a materially different version, of what the story body says.

This is the most common failure mode: the Executive Summary is written last and can drift from the story bodies written earlier.

**Flag if:** Exec Summary contradicts the story body on a core fact — who did what, the outcome of an event, or the direction of a trend.

---

### CHECK B — Conflict Party Completeness
**Severity: High**

For any active military conflict referenced in the brief: verify that all named belligerents are correctly and consistently identified across all sections. Check whether any party present in the sources is absent or inconsistent in the brief's framing.

**Flag if:** A named conflict party appears in source material but is absent or inconsistently named in the brief.

---

### CHECK C — Numerical Claims Verification
**Severity: Medium**

Identify every specific number: percentages, dollar amounts, barrel figures, vote counts, index levels, rate decisions. For each:
1. Find it in the cited source or re-fetched content
2. Confirm the number, unit, and direction all match

**Common failure modes:**
- A figure from an opinion piece attributed to a news article
- A USD/EUR figure where one outlet reports in euros and another in dollars — both may be correct, but the brief should not present one as definitive without noting the other
- A rounded figure that differs meaningfully from the source

**Flag if:** Any number cannot be matched to the cited source, or differs beyond rounding.

---

### CHECK D — Causal Framing Accuracy
**Severity: Medium**

For any story where the brief states why something happened, verify the sources support that causal chain and have the causes in the right order of importance.

**Pattern to watch:** Multi-factor events where sources cite Factor A as the trigger and Factor B as background. The brief must not reverse these.

**Flag if:** The brief inverts or omits a primary cause clearly stated in the source.

**Concurrent events — timeline check (Severity: High):**
When two major stories break on the same day, explicitly verify which came first before connecting them causally in the Executive Summary. The Executive Summary is written last and is the most likely place to introduce false causation between events that merely coincided.

Check: does the brief imply Event A caused or contributed to Event B? If so, confirm from the sources that A preceded B and that sources explicitly draw that link. If the events were simultaneous or independently caused, the brief must say so — not imply a causal chain.

**Flag if:** The Executive Summary links two same-day events causally when the sources show they were concurrent and independent.

---

### CHECK E — Unverifiable Claims Presented as Reported Fact
**Severity: Low**

Scan for:
- Sentences with "reportedly," "according to sources," or similar hedges that cannot be traced to the cited article
- Specific factual claims (named factions, timelines, institutional behavior) not in any cited source
- Background statistics inserted without citation (import percentages, GDP figures, displacement counts) presented as current reporting rather than context

**Flag if:** A hedged or specific factual claim cannot be traced to any source in the brief or re-fetched content.

---

### CHECK F — Scope and Attribution Integrity
**Severity: Low**

1. **Date scope:** Flag any cited article published outside the 24-hour lookback window used as a primary source without noting its date.
2. **Superlatives:** For any "largest," "first," "record," or "most" claim — verify the source explicitly makes that claim. If not, flag it.
3. **Article count:** Spot-check the "Articles reviewed: N" header figure against the Source Coverage Table. Flag if materially inconsistent.

---

## Step 4 — Fix Issues Directly in the Brief Body

**Apply fixes before appending the QC Report. Do not leave errors in the body for the reader to find.**

For each finding from Step 3, make the correction in-place in `output/RUN_DATE-morning-brief.md` using the Edit tool. Apply fixes in this order: High severity first, then Medium, then Low.

**Fix rules by check:**
- **CHECK A (Exec Summary vs. body):** Rewrite the Executive Summary sentence to match what the story body says. Do not change the story body.
- **CHECK B (Conflict party):** Insert the missing party name at the first reference to the conflict in the affected section.
- **CHECK C (Numerical claims):** Replace the incorrect figure with the source-verified figure. If unverifiable, add a `[unverified]` tag inline.
- **CHECK D (Causal framing / concurrent events):** Rewrite the offending sentence to accurately reflect the source's causal chain, or to separate concurrent events explicitly (e.g., "separately and concurrently," "independently of").
- **CHECK E (Unverifiable claims):** Either add a hedge ("background context — not from cited source") inline, or remove the claim if it cannot be attributed to any source in the brief.
- **CHECK F (Scope/attribution):** Add the out-of-window date disclosure inline; remove or qualify any unsupported superlative.

**Low-severity issues:** Fix if a clean correction exists. If the fix would require rewriting a full paragraph with uncertain sourcing, add an inline `[unverified — review]` tag instead and note it in the QC Report.

Record every fix made — what the original text said, what it now says, and which check triggered it. This log populates the QC Report in Step 5.

---

## Step 5 — Append QC Report to the Brief

**Always run this step, whether or not issues were found.**

After all fixes are applied, open `output/RUN_DATE-morning-brief.md` and append the following block at the very end of the file, after the Source Coverage Summary:

---

**If no issues found:**

```markdown
---

## QC Report
*Auto-generated by qc-brief | Run at: HH:MM*

All six checks passed. Sources re-fetched: N of M. No corrections made.
```

---

**If issues were found and fixed:**

```markdown
---

## QC Report
*Auto-generated by qc-brief | Run at: HH:MM*

Sources re-fetched: N of M (list any that failed to fetch).
**N correction(s) applied to brief body before send.**

| # | Check | Severity | Location | Action |
|---|-------|----------|----------|--------|
| 1 | [Check name] | High/Medium/Low | [Section > Story] | Fixed / Tagged |
| 2 | ... | ... | ... | ... |

### Correction 1 — [Check Letter]: [Check Name] | Severity: [High/Medium/Low]
**Location:** [Section > Story headline > Paragraph or Exec Summary]
**Issue:** [One sentence.]
**Was:** "[Original text]"
**Now:** "[Corrected text]"

### Correction 2 — ...
[Repeat for each change]
```

---

## Step 6 — Always Proceed to Send

After applying all fixes and appending the QC Report, return to the workflow and continue to Step 10 (email). Do not wait for human review.
