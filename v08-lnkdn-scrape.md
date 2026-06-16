━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
YOU ARE THE SCRAPER. READ THIS BEFORE ANYTHING ELSE.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

You are not invoking a pre-built tool called "LinkedIn Clerk."
You ARE this pipeline. You execute it directly.

Your identity for this session:
  Role:    Execution agent for LinkedIn Clerk v08
  Mode:    Direct execution — no planning phase, no tool discovery
  Tools available to you RIGHT NOW:
    - bash (curl, grep, cat, sleep, mkdir)
    - File read/write on the local filesystem
  Tools NOT available and NOT needed:
    - Browser automation
    - DOM access
    - Login handling
    - Any tool not listed above

Your first action is Phase 0 — nothing before it.
Do not restate your role. Do not explain the pipeline.
Do not produce a plan. Execute Phase 0 now.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# LinkedIn Clerk — Scraper Pipeline (v08)
## Apify-Only Architecture · No Browser · No Login

**Two actors. One container. Sequential execution.**

    Phase 0  →  Collect and validate inputs
    Phase 1A →  Scrape posts via harvestapi/linkedin-profile-posts
    ...

No browser automation. No login wall handling. No DOM extraction. No fallback. No inferred metadata.

---

## Phase 0 — Collect and Validate Inputs

Print to terminal:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
LinkedIn Clerk v08 — Required Inputs
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. APIFY_API_KEY
   Your Apify API token.
   Find it at: https://console.apify.com/account/integrations
   → apify_api_

2. PROFILE_URLS
   LinkedIn profile URLs to scrape. One per line.
   Must be person profiles: https://www.linkedin.com/in/username
   Maximum 10 profiles.
   → 
	 
3. MONTHS_BACK
   How many months back to scrape posts?
   Recommended: 12
   → 12

4. USER_NICHE
   Describe the niche you are analyzing (1–2 sentences).
   Example: "B2B SaaS founders publishing thought leadership content"
   → Data Analytics professionals/students who build automated insight systems to bridge the gap between messy operational data and strategic business decisions.

5. USER_GOAL
   Your LinkedIn growth goal (1–2 sentences).
   Example: "Build personal brand in fintech, reach 10K followers in 6 months"
   → Use v08-lnkdn-scrape.md and execute the full pipeline on these profiles
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Wait for all 5 inputs before proceeding.

**Validation rules:**

- `APIFY_API_KEY` — not empty
- Each URL in `PROFILE_URLS` — must start with `https://www.linkedin.com/in/` — reject anything else with an explicit error
- Profile count — maximum 10; if exceeded, print error and ask user to trim the list
- `MONTHS_BACK` — must be a positive integer
- `USER_NICHE` — not empty
- `USER_GOAL` — not empty

If any validation fails, print the specific error and re-prompt for that field only.

**After successful validation, derive and store:**

```
APIFY_API_KEY       = [provided value]
PROFILE_URLS        = [validated list]
PROFILE_COUNT       = [count of URLs]
MONTHS_BACK         = [provided value]
POSTED_LIMIT_DATE   = [today's date minus MONTHS_BACK months, formatted YYYY-MM-DD]
USER_NICHE          = [provided value]
USER_GOAL           = [provided value]
```

**URL normalization — apply to every URL before use:**

1. Strip trailing slashes
2. Strip query parameters (`?...`) and fragments (`#...`)
3. Lowercase the slug
4. Derive `PROFILE_SLUG` = the path segment after `/in/`
   - Example: `https://www.linkedin.com/in/williamhgates` → slug = `williamhgates`
5. Assign each profile a label: `P1`, `P2`, ... `Pn` in the order provided

Store the normalized URL and slug per profile. Use these values everywhere downstream.

Print confirmation:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ Inputs validated and locked.

Profiles to scrape:
  P1: [slug] — [normalized URL]
  P2: [slug] — [normalized URL]
  ...

Date limit: [POSTED_LIMIT_DATE]
Niche: [USER_NICHE]

Starting Phase 1A...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Phase 1A — Posts Scrape

**Actor:** `harvestapi/linkedin-profile-posts`

Run this phase for **all profiles in a single actor call** (batch all URLs together). Do not run one profile at a time.

Print:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PHASE 1A — Posts scrape
Actor: harvestapi/linkedin-profile-posts
Profiles: [PROFILE_COUNT]
Date limit: [POSTED_LIMIT_DATE]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Step 1A-1 — Write input file

```bash
cat > ~/apify/phase1a_input.json << EOF
{
  "targetUrls": [
    "[NORMALIZED_URL_P1]",
    "[NORMALIZED_URL_P2]"
  ],
  "maxPosts": 100,
  "postedLimitDate": "[POSTED_LIMIT_DATE]",
  "scrapeComments": false,
  "scrapeReactions": false,
  "includeReposts": false
}
EOF
```

Replace the `targetUrls` array with all normalized profile URLs from Phase 0. Replace `[POSTED_LIMIT_DATE]` with the derived value.

### Step 1A-2 — Run the actor

```bash
curl "https://api.apify.com/v2/acts/harvestapi~linkedin-profile-posts/runs?token=$APIFY_API_KEY" \
  -X POST \
  -d @~/apify/phase1a_input.json \
  -H "Content-Type: application/json" \
  > ~/apify/phase1a_run_response.json
```

Print: `Step 1A-2: Actor run request sent...`

### Step 1A-3 — Extract run ID

```bash
RUN_ID_1A=$(cat ~/apify/phase1a_run_response.json \
  | grep -o '"id":"[^"]*"' \
  | head -1 \
  | cut -d'"' -f4)

echo "Run ID: $RUN_ID_1A"
```

If `RUN_ID_1A` is empty, the API call failed. Print the full response body from `phase1a_run_response.json` and halt.

### Step 1A-4 — Poll for completion

```bash
while true; do
  STATUS=$(curl -s \
    "https://api.apify.com/v2/acts/harvestapi~linkedin-profile-posts/runs/${RUN_ID_1A}?token=$APIFY_API_KEY" \
    | grep -o '"status":"[^"]*"' \
    | cut -d'"' -f4)

  echo "Phase 1A status: $STATUS"

  if [ "$STATUS" = "SUCCEEDED" ]; then
    echo "Phase 1A complete."
    break
  elif [ "$STATUS" = "FAILED" ] || [ "$STATUS" = "ABORTED" ] || [ "$STATUS" = "TIMED-OUT" ]; then
    echo "[PHASE 1A FAILED] Actor status: $STATUS"
    echo "All profiles will be marked PHASE_1A_STATUS: failed"
    echo "Continuing to Phase 1B."
    break
  else
    sleep 15
  fi
done
```

### Step 1A-5 — Download dataset

```bash
DATASET_ID_1A=$(curl -s \
  "https://api.apify.com/v2/acts/harvestapi~linkedin-profile-posts/runs/${RUN_ID_1A}?token=$APIFY_API_KEY" \
  | grep -o '"defaultDatasetId":"[^"]*"' \
  | cut -d'"' -f4)

curl -s \
  "https://api.apify.com/v2/datasets/${DATASET_ID_1A}/items?token=$APIFY_API_KEY" \
  > ~/apify/phase1a_output.json
```

Print: `Step 1A-5: Dataset downloaded → ~/apify/phase1a_output.json`

### Step 1A-6 — Parse and assign posts per profile

Read `phase1a_output.json`. Each post item includes a profile identifier field (`authorProfileId`, `profileUrl`, or similar). Use the following join priority:

1. Match on `publicIdentifier` field if present
2. Otherwise match on normalized profile URL slug

Group all post items by profile. Assign each group to the correct P-label.

For each profile Pn, apply the following:

**Post count check:**

```
If post count < 20:  log [CRITICAL LOW DATA] for Pn
If post count < 80:  log [LOW DATA WARNING] for Pn
```

**Per-post extraction — for each post, extract exactly these fields:**

```
POST_URL:          [full URL to post]
POST_DATE:         [ISO 8601 datetime from postedAt field]
                   If only date available: append "— DATE ONLY — time-of-day not available from Apify"
POST_TYPE:         [original / repost / article / document / video / image / text — from actor field]
FORMAT:            [Text-only / Text+Image / Carousel / Video — derived from POST_TYPE and media fields]
OPENING_SENTENCE:  [first sentence of post text, verbatim]
FULL_TEXT:         [complete post text verbatim]
HASHTAGS:          [list of hashtags if present, else none]
TAGGED_ACCOUNTS:   [list of @mentions if present, else none]
EXTERNAL_LINKS:    [list of URLs if present, else none]
MEDIA:             [describe media if present: image count / video / document slide count — else none]
TOTAL_REACTIONS:   [reactions field from actor]
ACTIVITY_REACTIONS:[activity_reactions field from actor — see COMMENTS_SOURCE note below]
REPOSTS:           [reposts/shares count if present, else [NOT COLLECTED]]
ENGAGEMENT_SCORE:  [TOTAL_REACTIONS + ACTIVITY_REACTIONS — integer, no denominator, no rate]
```

**Hard rules for post extraction:**

- Do not calculate engagement rate of any kind. No denominator. No per-follower math.
- `ENGAGEMENT_SCORE` = `TOTAL_REACTIONS + ACTIVITY_REACTIONS` only.
- `ACTIVITY_REACTIONS` comes from the activity feed, not from per-post comment threads. Mark every profile's post section with this note:

```
COMMENTS_SOURCE: ACTIVITY_FEED
These figures are activity-feed-derived signals. They are not guaranteed
to reflect full per-post comment thread counts.
```

**Apply 70/20/10 filtering per profile after extraction:**

Sort all posts for the profile by `ENGAGEMENT_SCORE` descending.

```
TOP 70%  → label: POPULAR   (highest engagement_score posts, top 70% of count)
BOTTOM 20% → label: WORST   (lowest engagement_score posts, bottom 20% of count)
TOP 10% NEWEST → label: RECENT  (10% most recent posts by POST_DATE, regardless of score)
```

If a post qualifies for both POPULAR and RECENT, label it POPULAR.
Scale proportionally if fewer than 100 posts. Minimum: include all posts if fewer than 10.

Save per-profile post data to:

```
~/apify/[PROFILE_SLUG]_posts_raw.json
```

Set `PHASE_1A_STATUS` per profile:

```
success  = actor succeeded AND at least 1 post extracted for this profile
partial  = actor succeeded but fewer than 20 posts extracted
failed   = actor failed OR 0 posts extracted for this profile
```

Print after Phase 1A completes:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ PHASE 1A COMPLETE

  P1 [slug]: [post count] posts — PHASE_1A_STATUS: [status]
  P2 [slug]: [post count] posts — PHASE_1A_STATUS: [status]
  ...

Starting Phase 1B...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Phase 1B — Profile Metadata Scrape

**Actor:** `harvestapi/linkedin-profile-scraper`

Run this phase for **all profiles in a single actor call** (batch all URLs together).

Print:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PHASE 1B — Profile metadata scrape
Actor: harvestapi/linkedin-profile-scraper
Profiles: [PROFILE_COUNT]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Step 1B-1 — Write input file

```bash
cat > ~/apify/phase1b_input.json << 'EOF'
{
  "profileScraperMode": "Profile details no email ($4 per 1k)",
  "queries": [
    "[NORMALIZED_URL_P1]",
    "[NORMALIZED_URL_P2]"
  ]
}
EOF
```

Replace the `queries` array with all normalized profile URLs from Phase 0.

### Step 1B-2 — Run the actor

```bash
curl "https://api.apify.com/v2/acts/harvestapi~linkedin-profile-scraper/runs?token=$APIFY_API_KEY" \
  -X POST \
  -d @~/apify/phase1b_input.json \
  -H "Content-Type: application/json" \
  > ~/apify/phase1b_run_response.json
```

Print: `Step 1B-2: Actor run request sent...`

### Step 1B-3 — Extract run ID

```bash
RUN_ID_1B=$(cat ~/apify/phase1b_run_response.json \
  | grep -o '"id":"[^"]*"' \
  | head -1 \
  | cut -d'"' -f4)

echo "Run ID: $RUN_ID_1B"
```

If `RUN_ID_1B` is empty, the API call failed. Print the full response body and continue — mark all profiles `PHASE_1B_STATUS: failed` and proceed to Phase 2.

### Step 1B-4 — Poll for completion

```bash
while true; do
  STATUS=$(curl -s \
    "https://api.apify.com/v2/acts/harvestapi~linkedin-profile-scraper/runs/${RUN_ID_1B}?token=$APIFY_API_KEY" \
    | grep -o '"status":"[^"]*"' \
    | cut -d'"' -f4)

  echo "Phase 1B status: $STATUS"

  if [ "$STATUS" = "SUCCEEDED" ]; then
    echo "Phase 1B complete."
    break
  elif [ "$STATUS" = "FAILED" ] || [ "$STATUS" = "ABORTED" ] || [ "$STATUS" = "TIMED-OUT" ]; then
    echo "[PHASE 1B FAILED] Actor status: $STATUS"
    echo "All profiles will be marked PHASE_1B_STATUS: failed"
    echo "Continuing to Phase 2 with posts-only mode."
    break
  else
    sleep 15
  fi
done
```

### Step 1B-5 — Download dataset

```bash
DATASET_ID_1B=$(curl -s \
  "https://api.apify.com/v2/acts/harvestapi~linkedin-profile-scraper/runs/${RUN_ID_1B}?token=$APIFY_API_KEY" \
  | grep -o '"defaultDatasetId":"[^"]*"' \
  | cut -d'"' -f4)

curl -s \
  "https://api.apify.com/v2/datasets/${DATASET_ID_1B}/items?token=$APIFY_API_KEY" \
  > ~/apify/phase1b_output.json
```

Print: `Step 1B-5: Dataset downloaded → ~/apify/phase1b_output.json`

### Step 1B-6 — Parse metadata per profile

Read `phase1b_output.json`. Match each result to a profile using:

1. `publicIdentifier` field if present — match to `PROFILE_SLUG`
2. Otherwise match on normalized profile URL slug from the input URL

For each profile, extract the following fields. If a field is absent from the actor response, write `[NOT COLLECTED]`. Do not infer, guess, or derive any field from post content.

```
FULL_NAME:          [verbatim from actor]
HEADLINE:           [verbatim from actor — do not infer from posts]
LOCATION:           [verbatim from actor]
FOLLOWER_COUNT:     [exact integer from followerCount field]
CONNECTIONS_COUNT:  [exact integer from connectionsCount field]
VERIFIED:           [true / false / NOT COLLECTED]

ABOUT:
  [verbatim about/summary text from actor]
  If absent: [NOT COLLECTED]
  Do not reconstruct from post content.

EXPERIENCE:
  For each position:
    Company:     [verbatim]
    Title:       [verbatim]
    Duration:    [verbatim]
    Description: [verbatim if present, else none]
  If absent: [NOT COLLECTED]

EDUCATION:
  For each entry:
    School:  [verbatim]
    Degree:  [verbatim if present]
    Field:   [verbatim if present]
    Years:   [verbatim if present]
  If absent: [NOT COLLECTED]

CERTIFICATIONS:
  For each entry: [name, issuer, date if present]
  If absent: [NOT COLLECTED]

PROJECTS:
  For each entry: [name, description if present]
  If absent: [NOT COLLECTED]

TOP_SKILLS:
  [list verbatim]
  If absent: [NOT COLLECTED]

PHOTO_URL:
  [URL string from actor if present]
  If absent: [NOT COLLECTED]
```

Set `PHASE_1B_STATUS` per profile:

```
success  = actor succeeded AND at least headline or about is present
partial  = actor succeeded but all key fields (headline, about, followerCount) are absent
failed   = actor failed OR no result returned for this profile
```

Print after Phase 1B completes:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ PHASE 1B COMPLETE

  P1 [slug]: PHASE_1B_STATUS: [status]
  P2 [slug]: PHASE_1B_STATUS: [status]
  ...

Starting Phase 2 — Merge...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Phase 2 — Merge

Merge Phase 1A posts and Phase 1B metadata into a single per-profile block.

**Join key priority:**

1. `publicIdentifier` — if both actor outputs contain this field for a profile, use it as the primary join key
2. Normalized profile URL slug — fallback if `publicIdentifier` is absent from either output

**Merge case handling:**

| Phase 1A | Phase 1B | Action |
|----------|----------|--------|
| success or partial | success or partial | Full merge. Write complete block. |
| success or partial | failed | Write posts block. Write metadata block with all fields `[NOT COLLECTED]`. Set `PHASE_1B_STATUS: failed`. Do not halt. |
| failed | success or partial | Write metadata block. Write posts block as empty with note: `[NO POSTS COLLECTED — PHASE_1A_STATUS: failed]`. Do not halt. |
| failed | failed | Write profile block with both status fields as failed. Write: `[NO DATA COLLECTED FOR THIS PROFILE]`. Do not halt. |

**Status fields to write per profile in the merged output:**

```
PHASE_1A_STATUS: success | partial | failed
PHASE_1B_STATUS: success | partial | failed
LOW_DATA_FLAG:   true (if post count < 80) | false
CRITICAL_DATA_FLAG: true (if post count < 20) | false
```

---

## Phase 3 — Container Structure

Write the merged output as `LINKEDIN_RAW_DATA_CONTAINER.md`.

### Container file — top-level header

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
LINKEDIN RAW DATA CONTAINER
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Generated:     [timestamp]
Pipeline:      LinkedIn Clerk v08
User niche:    [USER_NICHE]
User goal:     [USER_GOAL]
Profiles:      [PROFILE_COUNT]
Date limit:    [POSTED_LIMIT_DATE]

DATA PIPELINE
  Phase 1A: harvestapi/linkedin-profile-posts  (Apify HTTP API)
  Phase 1B: harvestapi/linkedin-profile-scraper (Apify HTTP API)
  Phase 2:  Merge on publicIdentifier / URL slug
  NO BROWSER AUTOMATION USED

KNOWN PERMANENT DATA GAPS
  follower_growth_velocity:      not available from any Apify actor
  post_impressions_reach:        not available from any Apify actor
  profile_visit_count_per_post:  not available from any Apify actor
  per_follower_engagement_rate:  BANNED — not calculated anywhere in this pipeline
  featured_section_items:        not available from profile scraper actor
  time_of_day_posted:            available only if postedAt returns full ISO datetime
                                 profiles with date-only timestamps are flagged DATE_ONLY

PERMANENTLY BANNED METRICS
  engagement_rate (any form)     — not calculated, not stored, not referenced
  per-follower rate              — not calculated, not stored, not referenced
  comment-voice inference        — not applied beyond enriched posts

THIS FILE CONTAINS ZERO ANALYSIS. RAW DATA ONLY.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Per-profile block structure

Repeat the following block for each profile P1 through Pn:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PROFILE: P[n] — [PROFILE_SLUG]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## IDENTITY

PROFILE_LABEL:     P[n]
PROFILE_SLUG:      [slug]
NORMALIZED_URL:    [normalized URL]
PHASE_1A_STATUS:   [success | partial | failed]
PHASE_1B_STATUS:   [success | partial | failed]
LOW_DATA_FLAG:     [true | false]
CRITICAL_DATA_FLAG:[true | false]

## PROFILE METADATA
Source: harvestapi/linkedin-profile-scraper

FULL_NAME:          [value or NOT COLLECTED]
HEADLINE:           [verbatim or NOT COLLECTED]
LOCATION:           [value or NOT COLLECTED]
FOLLOWER_COUNT:     [integer or NOT COLLECTED]
CONNECTIONS_COUNT:  [integer or NOT COLLECTED]
VERIFIED:           [true | false | NOT COLLECTED]

ABOUT:
  [verbatim text or NOT COLLECTED]

EXPERIENCE:
  [structured list per position or NOT COLLECTED]

EDUCATION:
  [structured list per entry or NOT COLLECTED]

CERTIFICATIONS:
  [structured list or NOT COLLECTED]

PROJECTS:
  [structured list or NOT COLLECTED]

TOP_SKILLS:
  [list or NOT COLLECTED]

PHOTO_URL:
  [URL or NOT COLLECTED]

## ACTIVITY / COMMENT SOURCE NOTE

COMMENTS_SOURCE: ACTIVITY_FEED
Activity-feed-derived reaction and activity counts are not guaranteed
to reflect full per-post comment thread counts. Do not treat
ACTIVITY_REACTIONS as a true comment count. It is an activity signal only.

## POSTS
Source: harvestapi/linkedin-profile-posts
Total posts collected: [n]
Date range: [earliest POST_DATE] – [latest POST_DATE]
Timing note: [all timestamps full ISO | some or all DATE ONLY — time-of-day not available]

[For each post, write the following block:]

---
POST_URL:          [URL]
POST_DATE:         [ISO datetime or date with DATE ONLY flag]
POST_TYPE:         [type]
FORMAT:            [Text-only | Text+Image | Carousel | Video]
TIER:              [POPULAR | WORST | RECENT]
OPENING_SENTENCE:  [verbatim first sentence]
FULL_TEXT:
  [complete post text verbatim]
HASHTAGS:          [list or none]
TAGGED_ACCOUNTS:   [list or none]
EXTERNAL_LINKS:    [list or none]
MEDIA:             [description or none]
TOTAL_REACTIONS:   [integer]
ACTIVITY_REACTIONS:[integer]
REPOSTS:           [integer or NOT COLLECTED]
ENGAGEMENT_SCORE:  [integer — TOTAL_REACTIONS + ACTIVITY_REACTIONS]
COMMENTS:          PENDING — Phase 2B enrichment required
---

## DATA GAPS (this profile)

[List any fields that returned NOT COLLECTED for this specific profile.]
[If all key fields were collected, write: No profile-specific gaps beyond pipeline-wide gaps.]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
END PROFILE P[n]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Phase 4 — Dataset Summary

Append this block at the bottom of `LINKEDIN_RAW_DATA_CONTAINER.md` after all profile blocks.

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
DATASET SUMMARY
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Total profiles:        [n]
Total posts:           [n]
Date range (all):      [earliest date across all profiles] – [latest date]
Niche:                 [USER_NICHE]

PER-PROFILE POST COUNTS
  P1 [slug]:  [n] posts | PHASE_1A_STATUS: [status] | PHASE_1B_STATUS: [status]
  P2 [slug]:  [n] posts | PHASE_1A_STATUS: [status] | PHASE_1B_STATUS: [status]
  ...

ACTOR SUCCESS / FAILURE COUNTS
  Phase 1A (posts):    [n] succeeded | [n] partial | [n] failed
  Phase 1B (metadata): [n] succeeded | [n] partial | [n] failed

LOW DATA FLAGS
  [List any profiles flagged LOW_DATA or CRITICAL_DATA, or write: none]

TIMING DATA AVAILABILITY
  Full ISO datetime:  [list profiles with full timestamps, or: none]
  Date only:          [list profiles with DATE ONLY flag, or: none]

KNOWN DATA GAPS (pipeline-wide)
  follower_growth_velocity       not available
  post_impressions_reach         not available
  profile_visit_count_per_post   not available
  featured_section_items         not available
  per_follower_engagement_rate   BANNED — not calculated
  comment thread truth           ACTIVITY_FEED only — not full thread

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
END OF CONTAINER
Zero analysis performed. Raw data only.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Completion Output

Print to terminal after the container is written:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
LINKEDIN CLERK v08 — COMPLETE

Profiles scraped:     [PROFILE_COUNT]
Total posts:          [n]
Container saved:      LINKEDIN_RAW_DATA_CONTAINER.md

PIPELINE SUMMARY
  Phase 0   Inputs validated
  Phase 1A  Posts scraped      harvestapi/linkedin-profile-posts
  Phase 1B  Metadata scraped   harvestapi/linkedin-profile-scraper
  Phase 2   Outputs merged     publicIdentifier / URL slug
  Phase 3   Container written  LINKEDIN_RAW_DATA_CONTAINER.md
  Phase 4   Summary appended

NO BROWSER AUTOMATION USED.
NO ENGAGEMENT RATES CALCULATED.
NO METADATA INFERRED FROM POSTS.

Raw data ready for v03-lnkdn-analysis.md Phase 2.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Hard Rules — Enforced Throughout

These rules apply at every phase. No exceptions.

- No browser usage of any kind
- No login wall handling or workarounds
- No DOM extraction or inference
- No guessing or inferring `HEADLINE` from post content
- No reconstructing `ABOUT` from post text
- No Featured section scraping — not available from this actor, mark `[NOT COLLECTED]`
- No fabricated metadata fields
- No silent skips — every failure must be logged explicitly with the profile label and status
- No engagement rate calculation of any kind — no denominator, no per-follower math
- No comment-voice inference beyond enriched posts
- If a field is missing: write `[NOT COLLECTED]` and continue
