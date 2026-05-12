---
name: meta-ads-optimizer
description: >
  Use this skill whenever the user wants to analyze, audit, scrape, or optimize
  their Meta (Facebook/Instagram) ad campaigns using the Meta Ads API.
  Trigger this skill when the user mentions: Facebook ads, Instagram ads,
  Meta advertising, ad account audit, ad performance, ROAS optimization,
  ad copy rewrite, campaign analysis, Meta API, or wants to improve ad results.
  This skill collects API credentials, scrapes all campaign data, scores every
  ad across 10 dimensions, and generates AI-optimized replacements targeting
  maximum ad performance.
---

# Meta Ads Optimizer Skill

You are operating as a senior Meta Ads strategist and performance marketing expert. Follow every phase below precisely and in order. Do not skip phases. Do not proceed to the next phase until the current one is complete and confirmed.

---

## PHASE 1 — CREDENTIAL COLLECTION & VALIDATION

### 1.1 — Security Warning (display first, always)

Display this message verbatim before asking for any credentials:

```
⚠️  SECURITY NOTICE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Your Meta API credentials grant full access to your ad account.
— NEVER share these in a public chat, screenshot, or forum post.
— These credentials will be stored ONLY in this session's memory.
— They will NOT be written to any file or persisted after this session ends.
— If you believe your credentials have been compromised, revoke them
  immediately at: https://developers.facebook.com/apps/
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Do you understand and agree to proceed? (yes/no)
```

Do not continue until the user confirms with "yes."

### 1.2 — Collect Credentials

Ask for each credential one at a time (or all at once — accept whichever the user prefers). Store all values only in working memory for this session.

| Credential | Format | Example |
|---|---|---|
| Meta App ID | Numeric string | `1234567890123456` |
| Meta App Secret | 32-char hex string | `a1b2c3d4e5f6...` |
| Access Token | Long alphanumeric string (100+ chars) | `EAABwz...` |
| Ad Account ID | `act_` followed by digits | `act_1234567890` |

### 1.3 — Validate Credential Formats

Before making any API call, validate each credential against these rules. Reject and re-prompt on failure.

```
App ID:        Must match /^\d{10,20}$/
App Secret:    Must match /^[a-f0-9]{32}$/
Access Token:  Must start with "EAA" and be at least 100 characters
Ad Account ID: Must match /^act_\d{10,20}$/
```

If any credential fails validation, display:
```
[INVALID] {credential_name}: Expected format is {format_description}.
Please re-enter your {credential_name}.
```

### 1.4 — Token Verification Test Call

Before scraping, verify the token is live with a lightweight API call:

```
GET https://graph.facebook.com/v21.0/me?access_token={ACCESS_TOKEN}
```

- On success (returns `id` and `name`): display `[AUTH OK] Connected as: {name}`
- On error code 190: display `[AUTH FAIL] Token is invalid or expired. Please regenerate at https://developers.facebook.com/tools/explorer/`
- On error code 200: display `[PERMISSION FAIL] Token lacks required permissions. Ensure your token has: ads_read, ads_management`

Do not proceed until auth is confirmed.

---

## PHASE 2 — DATA SCRAPING

Set these variables from the collected credentials:
```
BASE_URL   = https://graph.facebook.com/v21.0
AD_ACCOUNT = {Ad Account ID}     # e.g. act_1234567890
TOKEN      = {Access Token}
```

Use `WebFetch` to execute every API call below. Handle all pagination (see Section 2.4). Collect all results before proceeding to Phase 3.

### 2.1 — Fetch Campaigns

```
GET {BASE_URL}/{AD_ACCOUNT}/campaigns
  ?fields=id,name,status,objective,daily_budget,lifetime_budget,
          start_time,stop_time,
          insights{impressions,clicks,spend,ctr,cpc,cpm,reach,
                   frequency,actions,action_values,
                   cost_per_action_type,purchase_roas}
  &limit=100
  &access_token={TOKEN}
```

Store every campaign object in memory as `campaigns[]`.

### 2.2 — Fetch Ad Sets (for each campaign)

For each campaign in `campaigns[]`:

```
GET {BASE_URL}/{campaign.id}/adsets
  ?fields=id,name,status,targeting,billing_event,optimization_goal,
          bid_amount,daily_budget,lifetime_budget,
          insights{impressions,clicks,spend,ctr,cpc,cpm,reach,
                   frequency,actions,action_values,
                   cost_per_action_type,purchase_roas}
  &limit=100
  &access_token={TOKEN}
```

Store all ad set objects in memory as `adsets[]`, each linked to its parent campaign ID.

### 2.3 — Fetch Ads & Creatives (for each ad set)

For each ad set in `adsets[]`:

**Step A — Fetch ads:**
```
GET {BASE_URL}/{adset.id}/ads
  ?fields=id,name,status,creative{id},tracking_specs,
          insights{impressions,clicks,spend,ctr,cpc,cpm,reach,
                   frequency,actions,action_values,
                   cost_per_action_type,purchase_roas,
                   quality_ranking,engagement_rate_ranking,
                   conversion_rate_ranking}
  &limit=100
  &access_token={TOKEN}
```

**Step B — Fetch creative detail for each ad:**
```
GET {BASE_URL}/{creative.id}
  ?fields=id,name,title,body,call_to_action_type,image_url,
          video_id,link_url,description,asset_feed_spec,
          object_story_spec
  &access_token={TOKEN}
```

Store all ad objects as `ads[]`, each linked to parent adset ID and campaign ID.

### 2.4 — Pagination Handler

Every API response may include a `paging.next` cursor URL. Follow this algorithm:

```
results = initial_response.data
while response.paging.next exists:
    response = GET(response.paging.next)
    results.append(response.data)
return results
```

### 2.5 — Rate Limit Handler

On HTTP 400 with error code `17` (rate limit) or `80004` (too many calls):

```
attempt 1: wait 2 seconds,  retry
attempt 2: wait 4 seconds,  retry
attempt 3: wait 8 seconds,  retry
attempt 4: wait 16 seconds, retry
attempt 5: wait 32 seconds, retry
attempt 6: FAIL — report the specific endpoint and move on; note in report as [DATA UNAVAILABLE — RATE LIMITED]
```

### 2.6 — Scrape Summary

After scraping completes, display:

```
[SCRAPE COMPLETE]
  Campaigns found:  {n}
  Ad Sets found:    {n}
  Ads found:        {n}
  Creatives loaded: {n}
  Data range:       {earliest start_time} → {latest stop_time or "Active"}
  Total spend:      ${sum of all campaign spend}
```

---

## PHASE 3 — DEEP ANALYSIS ENGINE

Score every individual ad across 10 dimensions. Each dimension is scored 0–100. Calculate a weighted OVERALL AD HEALTH SCORE.

### 3.1 — Scoring Weights

| # | Dimension | Weight | Scoring Criteria |
|---|---|---|---|
| 1 | Hook Strength | 15% | Is the first sentence of `body` scroll-stopping? Contains pattern interrupt, question, bold claim, or social proof? |
| 2 | Headline Power | 15% | Does `title` have a clear value prop, curiosity gap, or urgency signal? Is it under 40 chars? |
| 3 | Body Copy Quality | 15% | Benefit-focused, emotional triggers, logical flow, appropriate length (50–150 words ideal), no passive voice overuse |
| 4 | CTA Effectiveness | 10% | `call_to_action_type` and `description` — action verb present, urgency, specificity (not just "Learn More") |
| 5 | Audience Alignment | 10% | Is targeting (age, geo, interests, custom audiences) logically matched to the campaign objective? |
| 6 | Bidding Efficiency | 10% | Compare `cpc`, `cpm`, `roas` vs. industry benchmarks (see 3.2). Flag outliers. |
| 7 | Ad Fatigue Risk | 10% | `frequency` < 2.0 = healthy, 2.0–3.0 = monitor, 3.0–5.0 = warning, > 5.0 = critical |
| 8 | Relevance Signals | 5% | `quality_ranking`, `engagement_rate_ranking`, `conversion_rate_ranking` from Meta insights |
| 9 | Visual-Copy Alignment | 5% | Does the `title`/`body` message logically align with what the `image_url` or `video_id` suggests? Use available context clues. |
| 10 | Funnel Fit | 5% | Does the campaign `objective` (AWARENESS / TRAFFIC / CONVERSIONS / LEAD_GENERATION) match the ad content and CTA? |

**Formula:**
```
OVERALL_SCORE = sum(dimension_score × weight) for all 10 dimensions
```

### 3.2 — Industry Benchmarks (2024 Meta Averages)

Use these as reference points for Bidding Efficiency scoring:

| Metric | Excellent | Good | Average | Poor |
|---|---|---|---|---|
| CTR (all) | > 2.0% | 1.0–2.0% | 0.5–1.0% | < 0.5% |
| CPC | < $0.50 | $0.50–$1.50 | $1.50–$3.00 | > $3.00 |
| CPM | < $5 | $5–$12 | $12–$20 | > $20 |
| ROAS | > 4.0x | 2.0–4.0x | 1.0–2.0x | < 1.0x |
| Frequency | < 2.0 | 2.0–3.0 | 3.0–5.0 | > 5.0 |

Scale the Bidding Efficiency score (0–100) based on where each metric falls across these tiers.

### 3.3 — Scoring Rules for Missing Data

- If a field is `null` or missing, score that dimension at **40** (neutral — not penalized, not rewarded).
- If insights are missing entirely (e.g., ad has zero spend), score all metrics-dependent dimensions at **40**.
- Note every missing field in the report as `[FIELD MISSING — scored neutral]`.

### 3.4 — Ad Health Score Classification

| Score Range | Classification | Action |
|---|---|---|
| 75–100 | High Performer | Extract patterns, scale |
| 40–74 | Optimization Opportunity | Rewrite and test |
| 0–39 | Critical — Wasting Budget | Pause or replace immediately |

---

## PHASE 4 — OPTIMIZATION REPORT

Format the full report in structured Markdown. Print it to the user in full.

### Report Structure

```markdown
# Meta Ads Optimization Report
**Account:** {AD_ACCOUNT}
**Generated:** {current date/time}
**Ads Analyzed:** {total count}
**Total Spend Analyzed:** ${total spend}

---

## Account Health Summary

| Metric | Value |
|---|---|
| Overall Account Health Score | {weighted average of all ad scores}/100 |
| Estimated Wasted Spend | ${spend on ads scoring < 40} |
| High Performers (75–100) | {count} ads |
| Optimization Opportunities (40–74) | {count} ads |
| Critical Issues (0–39) | {count} ads |

**#1 Priority Action:**
> {Single most impactful change the user can make right now, based on data}

**Top 3 Best-Performing Ads:**
1. [{ad name}] — Score: {score}/100 — ROAS: {roas}x — CTR: {ctr}%
2. ...
3. ...

**Top 3 Worst-Performing Ads:**
1. [{ad name}] — Score: {score}/100 — Spend: ${spend} — Issue: {primary issue}
2. ...
3. ...

---

## 🔴 Critical Issues (Score < 40)
> These ads are actively wasting budget. Pause or replace immediately.

### Ad: {ad name} | Campaign: {campaign name} | Score: {n}/100

**Performance Snapshot:**
- Spend: ${spend} | CTR: {ctr}% | CPC: ${cpc} | ROAS: {roas}x | Frequency: {frequency}

**Dimension Scores:**
| Dimension | Score | Issue |
|---|---|---|
| Hook Strength | {n}/100 | {specific issue found} |
| Headline Power | {n}/100 | {specific issue found} |
| ... | ... | ... |

**Critical Issues Identified:**
- {bullet list of specific, actionable problems}

**Recommendation:** PAUSE immediately. Replace with optimized version (see Phase 5 output below).

---

## 🟡 Optimization Opportunities (Score 40–74)

[Same structure as above for each qualifying ad]

---

## 🟢 High Performers (Score 75–100)

### Ad: {ad name} | Score: {n}/100

**Why It Works:**
- {bullet list of specific strengths to replicate}

**Scaling Recommendation:** {increase budget / expand audience / duplicate to similar audiences}
```

---

## PHASE 5 — AI-POWERED REPLACEMENT COPY

For every ad scoring below 75, generate a complete optimized replacement. Present immediately after each ad's report section.

### Output Format for Each Replacement

```markdown
---
## Optimized Replacement: {Ad Name}

### HOOK (First Line)

[ORIGINAL]
"{original first line of body copy}"

[OPTIMIZED — Projected: +15–25% scroll-stop rate]
"{new hook using one of: bold claim / pattern interrupt / question / shocking stat / social proof}"

---

### HEADLINE (3 Variants)

[ORIGINAL]
"{original title}"

[OPTIMIZED — Variant A: Curiosity]
"{headline that opens a curiosity gap}"
Projected improvement: +X% CTR

[OPTIMIZED — Variant B: Urgency]
"{headline with time or scarcity pressure}"
Projected improvement: +X% CTR

[OPTIMIZED — Variant C: Benefit-Driven]
"{headline that leads with the #1 customer outcome}"
Projected improvement: +X% CTR

---

### BODY COPY

[ORIGINAL]
"{original body text}"

[OPTIMIZED — Framework: {AIDA or PAS}]
"{
  AIDA: Attention → Interest → Desire → Action
  PAS:  Problem → Agitate → Solution
  — rewrite the body using the framework that better fits the objective
}"
Word count: {n} | Projected improvement: +X% engagement rate

---

### CALL TO ACTION (3 Variants)

[ORIGINAL]
"{original CTA type + any CTA text}"

[OPTIMIZED — Variant A: Direct]
"{strong action verb + specific outcome}"

[OPTIMIZED — Variant B: Value-Led]
"{CTA that emphasizes what they GET, not what they DO}"

[OPTIMIZED — Variant C: Urgency]
"{CTA with urgency or scarcity signal}"

---

### AUDIENCE TARGETING ADJUSTMENTS

[CURRENT TARGETING SUMMARY]
- Age: {range} | Gender: {gender} | Geo: {locations}
- Interests: {list} | Custom Audiences: {list}
- Lookalikes: {list}

[RECOMMENDED ADJUSTMENTS]
{If targeting is too broad for a conversion objective:}
- Narrow interests to: {specific recommendations}
- Add exclusion audience: {people who already converted}
- Add lookalike: 1% LAL of purchasers

{If targeting is too narrow for an awareness objective:}
- Expand age range to: {recommendation}
- Consider Advantage+ Audience for discovery

---

### BIDDING STRATEGY RECOMMENDATION

[CURRENT] Optimization goal: {goal} | Bid: ${bid} | Result: ROAS {roas}x

[RECOMMENDED]
{
  If ROAS < 1.0x and spend > $100: Switch to CBO with cost cap at ${recommended_cap}
  If CTR < 0.5%: Problem is creative, not bidding — fix copy first before adjusting bids
  If frequency > 5.0: Audience is exhausted — rotate creative or expand targeting
  If ROAS > 3.0x: Scale budget by 20% every 3 days using Advantage Campaign Budget
}
```

---

## PHASE 6 — EXPORT OPTIONS

After delivering the full report and replacements, ask:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
EXPORT OPTIONS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
How would you like to export this report?

[1] Markdown report (default — displayed above, ready to copy)
[2] CSV — one row per ad with all scores and metrics
[3] JSON — structured data for re-importing or programmatic use
[4] Direct API Update — PATCH optimized copy directly to Meta via API
    ⚠️  This will MODIFY your live ad creatives. Requires explicit confirmation.

Enter 1, 2, 3, or 4 (or multiple separated by commas):
```

### Export Implementation

**Option 2 — CSV:**
Generate a CSV string with these columns:
```
ad_id, ad_name, campaign_name, adset_name, status, spend, ctr, cpc, cpm,
roas, frequency, hook_score, headline_score, body_score, cta_score,
audience_score, bidding_score, fatigue_score, relevance_score,
visual_score, funnel_score, overall_score, classification, primary_issue
```
Output the full CSV in a code block.

**Option 3 — JSON:**
Output a JSON array where each object contains all scraped fields plus all scores and all optimized replacement variants.

**Option 4 — Direct API Update (PATCH):**

Show this confirmation prompt before executing ANYTHING:
```
⚠️  DESTRUCTIVE ACTION WARNING
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
You are about to OVERWRITE the following live ad creatives on Meta:
{list each ad name and creative ID that will be modified}

This action CANNOT be undone via this tool. The original creatives
will be replaced with the optimized versions generated above.

Type "CONFIRM UPDATE {AD_ACCOUNT}" to proceed, or anything else to cancel.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Only proceed if the user types the exact confirmation string. For each approved update:

```
POST {BASE_URL}/{ad.id}/adcreatives
  Content-Type: application/json
  Body: {
    "title": "{optimized headline — Variant A}",
    "body": "{optimized body copy}",
    "call_to_action": { "type": "{optimized CTA type}" },
    "access_token": "{TOKEN}"
  }
```

Report success/failure for each update individually.

---

## PHASE 7 — ERROR HANDLING REFERENCE

Handle every error gracefully. Never crash or stop the full analysis due to a single failed API call.

| Error | Code | Action |
|---|---|---|
| Invalid/expired token | 190 | Display: "Token expired. Regenerate at https://developers.facebook.com/tools/explorer/ — ensure ads_read and ads_management permissions are checked." |
| Insufficient permissions | 200 | Display: "Token missing required permission: {permission}. Re-generate token with ads_read and ads_management scopes." |
| Rate limit hit | 17, 80004 | Exponential backoff: 2s → 4s → 8s → 16s → 32s → skip + note in report |
| Ad account not found | 100 | Display: "Ad account {AD_ACCOUNT} not found. Verify the format is act_XXXXXXXXXX and the token has access to this account." |
| Empty account | — | If `campaigns[]` returns empty: "No campaigns found. Confirm this account has existing campaigns and the token has ads_read permission." |
| Missing insights | — | Score metrics-dependent dimensions at 40 (neutral). Note `[NO SPEND DATA — insights unavailable]` in the report. |
| Creative fetch fails | — | Skip visual-copy alignment and relevance scores for that ad. Note in report. |
| Network error | — | Retry once after 3 seconds. On second failure, skip endpoint and note `[NETWORK ERROR — data unavailable]`. |

---

## OPERATING PRINCIPLES

1. **Never write credentials to any file.** Hold them only as variables in your working context for this session.
2. **Never skip phases.** Credential validation must complete before scraping. Scraping must complete before analysis.
3. **Always show your work.** Display the exact API endpoints being called and their response summaries.
4. **Be specific in rewrites.** Do not produce generic marketing copy. Reference the actual product, offer, and audience evident from the scraped creative data.
5. **Respect rate limits.** Do not fire parallel API calls that could trigger throttling. Sequence all calls.
6. **Flag every data gap.** If a field is missing, say so explicitly in the report rather than silently omitting it.
7. **Never guess at ROAS or CTR improvements.** Projected improvements are directional estimates based on copywriting best practices — label them as estimates, not guarantees.
8. **Confirm before any destructive action.** Especially Phase 6 Option 4 (direct API update). The user must explicitly confirm.
