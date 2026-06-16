# LinkedIn Clerk — Phase 2: Analysis Pipeline (v05)
## COMPETITOR DATA → STRATEGIC SOP + CONTENT IDEAS

This protocol reads LINKEDIN_RAW_DATA_CONTAINER.md produced by v08-lnkdn-scrape.md
and outputs REPORT_1.md, VALIDATION_LAYER.md, and REPORT_2.md.

Zero scraping. Zero data collection. Pure analysis and synthesis.

Three windows. One model. Run sequentially.

---

# PIPELINE CONTRACT

**Read this section before executing any window. It overrides any conflicting instruction elsewhere in this file.**

## Authoritative Source

`LINKEDIN_RAW_DATA_CONTAINER.md` is the sole data source for all analysis.
No data exists in this pipeline beyond what the container explicitly contains.
Do not infer, reconstruct, or supplement any field from external knowledge.

## Unavailable Fields — Hard Stops

These fields do not exist in the v08 container. Do not reference them, infer them, or substitute for them:

| Field | Status | Correct handling |
|---|---|---|
| `featured_section_items` | NOT IN PIPELINE | Write `[NOT COLLECTED — not available from v08 pipeline]` and stop |
| `OFFER` / `PRIMARY CTA` | NOT IN PIPELINE | Infer from `ABOUT` text only; tag [HYPOTHESIS]; [NOT INFERABLE] if ABOUT is absent |
| `Minutes since post published` | NOT IN PIPELINE | Write `[REPLY SPEED NOT CALCULABLE]` and stop |
| `follower_growth_velocity` | NOT IN PIPELINE | Never reference |
| `post_impressions_reach` | NOT IN PIPELINE | Never reference |
| `profile_visit_count_per_post` | NOT IN PIPELINE | Never reference |

## Permanently Banned Metrics

These must not appear anywhere in REPORT_1.md, VALIDATION_LAYER.md, or REPORT_2.md:

- Engagement rate of any form
- Per-follower engagement rate
- Any ratio with follower count as denominator
- Comment-voice inference beyond Phase 2B enriched posts

If an NLM source in Window 1B recommends an engagement rate metric, note the recommendation in the CORPUS ADDITIONS block but explicitly exclude it from REPORT_2.md synthesis.

## Partial Data — Downgrade Rules

Apply these rules every time a data-quality flag is encountered. These rules are not suggestions.

**CRITICAL_DATA_FLAG: true** (fewer than 20 posts)
- All findings for this profile are tagged: `[LOW CONFIDENCE — fewer than 20 posts]`
- This profile must not serve as the sole basis for any cross-profile conclusion
- This profile must not be used as the "format from" or "tone from" source anchor in Section 7.3 crossover opportunities
- Tone/voice mapping (Section 7.2) is permitted but every characteristic is tagged `[LOW CONFIDENCE]`

**LOW_DATA_FLAG: true** (fewer than 80 posts, ≥ 20)
- All findings for this profile are noted as less reliable in per-profile breakdowns
- May contribute to cross-profile synthesis but must be noted as a lower-weight input

**PHASE_1B_STATUS: failed**
- HEADLINE, ABOUT, FOLLOWER_COUNT, EXPERIENCE, EDUCATION, TOP_SKILLS are all `[NOT COLLECTED]`
- Sections 6.1, 6.2, 6.4, 6.5 for this profile must be written from post content only and tagged `[HYPOTHESIS — no metadata available]`
- This profile must not anchor any Section 7.2 tone mapping that relies on HEADLINE or ABOUT text
- Archetype assignment in Section 6.5 is `[INSUFFICIENT DATA]` if the profile is also CRITICAL_DATA_FLAG: true

## Minimum Evidence Floors

A finding may not be stated as a pattern unless it meets the minimum evidence count below.
Below the floor: note the observation, state the evidence count, and write `[THIN DATA — treat as anecdote, not pattern]`.

| Finding type | Minimum posts required |
|---|---|
| "Consistent trait" across outliers (Section 2.3) | ≥ 3 posts sharing the trait |
| Hook × narrative coupling pattern — top or bottom 20% (Section 3.4) | ≥ 3 posts sharing the combination |
| Cross-profile offer/funnel pattern (Section 6.4) | ≥ 3 profiles with explicit CTA/offer evidence in ABOUT |
| Tone/voice characteristic for a single profile (Section 7.2) | ≥ 20 posts for that profile |

## Comment Data Scope

`ACTIVITY_REACTIONS` is an activity-feed-derived signal. It is not a per-post comment count.
True comment data exists only for Phase 2B enriched posts (top posts per profile, if Phase 2B was run).
If Phase 2B was not run and no enriched posts exist in the container, Section 5 must write:
`[NO ENRICHED POST DATA — Phase 2B was not run. Section 5 analysis cannot be performed.]`
and stop. Do not attempt to analyze comment behavior from ACTIVITY_REACTIONS.

---

# PHASE 0: COLLECT INPUTS

Print the following to terminal and wait for user responses:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
LinkedIn Clerk — Phase 2: Analysis Pipeline (v05)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Before starting, provide the following inputs:

1. CONTAINER_PATH
   Full path to LINKEDIN_RAW_DATA_CONTAINER.md
   (produced by v08-lnkdn-scrape.md)
   → 

2. USER_GOAL
   What do you want to achieve on LinkedIn?
   (1–3 sentences)
   → 

3. USER_NICHE
   Your niche — check the container header for a pre-filled value.
   Confirm it or enter a replacement below.
   → 
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

After the user provides all three inputs:

1. Validate that the file at CONTAINER_PATH exists and is readable.
2. Read the DATASET SUMMARY block from the bottom of the container
   (written by v08 Phase 4 — appears after all per-profile blocks,
   under the heading "DATASET SUMMARY").
3. Print the following confirmation block:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ Container loaded
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Profiles:      [n]
Profile IDs:   [P1: slug | P2: slug | ...]
               (slugs from DATASET SUMMARY PER-PROFILE POST COUNTS block;
                full names from each profile's FULL_NAME field if PHASE_1B_STATUS ≠ failed)
Total posts:   [n]
Date range:    [x] – [y]
Niche:         [USER_NICHE]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Confirm these details are correct before proceeding. (yes/no)
→ 
```

Wait for user confirmation.

- If **yes**: store all inputs and proceed.
- If **no**: ask "What needs to be corrected?" — accept correction, update the relevant input, re-display the confirmation block, and wait for confirmation again.

After confirmed yes, print:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ Inputs locked. Three windows to complete.

WINDOW 1A → REPORT_1.md (8 sections)
WINDOW 1B → VALIDATION_LAYER.md (4 NLM batches)
WINDOW 2  → REPORT_2.md (SOP + Content Ideas)

Starting Window 1A now...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

# WINDOW 1A: GENERATE REPORT_1.md

## Window 1A Rules — Enforce Strictly

- The PIPELINE CONTRACT at the top of this file is authoritative. Re-read it before writing anything.
- Read LINKEDIN_RAW_DATA_CONTAINER.md fully before writing anything.
- Complete the PRE-PASS (see below) before writing any section.
- Write all 8 sections in order — do not skip, merge, or reorder any section.
- Every section follows this internal structure:
  - Per-profile breakdown
  - Cross-profile synthesis
  - [DATA-BACKED CONCLUSIONS]
  - [MODEL CONJECTURES]
- **[DATA-BACKED CONCLUSIONS]** — only findings directly tied to scraped metrics. No "should" language. No prescriptions.
- **[MODEL CONJECTURES]** — cautious extrapolations only. "This may suggest X" is allowed. "You should do X" is NOT. Zero prescriptions.
- REPORT_1.md is read-only the moment it is saved. It is never edited again.
- If quota interrupts mid-section: on resuming, read the partial REPORT_1.md, identify the last completed section, continue from the next one. Never re-run completed sections.

## Window 1A PRE-PASS: Data Extraction

**Run this before writing any section. Do not write narrative until the PRE-PASS is complete.**

Read the full container and extract the following into a working fact table. Print the table to terminal before beginning Section 1. This table is for execution reference only — it does not appear in REPORT_1.md.

```
PRE-PASS FACT TABLE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
For each profile Pn:
  Slug:                  [slug]
  FULL_NAME:             [value or NOT COLLECTED]
  FOLLOWER_COUNT:        [integer or NOT COLLECTED]
  PHASE_1A_STATUS:       [success | partial | failed]
  PHASE_1B_STATUS:       [success | partial | failed]
  LOW_DATA_FLAG:         [true | false]
  CRITICAL_DATA_FLAG:    [true | false]
  Post count:            [n]
  Date range:            [earliest – latest]
  DATE ONLY flag:        [yes | no]
  Enriched posts exist:  [yes | no] (check for COMMENTS blocks with actual data)
  Format counts:         Text-only=[n] | Text+Image=[n] | Carousel=[n] | Video=[n]
  TOTAL_REACTIONS range: [min – max]
  ACTIVITY_REACTIONS range: [min – max]
  ENGAGEMENT_SCORE range: [min – max]
  Median ENGAGEMENT_SCORE (all posts): [n]
  Top decile avg ENGAGEMENT_SCORE: [n]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Apply downgrade flags from the PIPELINE CONTRACT now. Any profile with CRITICAL_DATA_FLAG: true or PHASE_1B_STATUS: failed must be noted in the table with its applicable constraints before section writing begins.

Print after PRE-PASS:
```
✓ PRE-PASS complete. Beginning Section 1...
```

## Window 1A Terminal Output

Print at the start of Window 1A:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 WINDOW 1A: GENERATING REPORT_1.md
Reading container... [n] profiles | [n] posts
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Print before each section:

```
Writing Section [n]/8: [section name]...
```

Print after each section:

```
✓ Section [n] complete — saved checkpoint
```

---

## REPORT_1.md — Full File Contents

Write the following as REPORT_1.md, top to bottom, in full:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
REPORT 1 — LINKEDIN COMPETITOR DATA ANALYSIS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Generated: [timestamp]
Container: [CONTAINER_PATH]
Profiles:  [n] | Posts: [n] | Date range: [x – y]
User goal: [USER_GOAL]
User niche: [USER_NICHE]

THIS FILE IS READ-ONLY. Do not edit after generation.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### SECTION 1 — OVERVIEW & DATASET SANITY

```
## SECTION 1 — OVERVIEW & DATASET SANITY
```

#### 1.1 Profile Roster

For each profile P1 through Pn, provide the following in a structured block. Pull all values directly from the PRE-PASS fact table — do not re-derive.

- Profile label (P1, P2, etc.) and FULL_NAME (write `[NOT COLLECTED]` if PHASE_1B_STATUS: failed)
- FOLLOWER_COUNT (write `[NOT COLLECTED]` if absent — do not estimate)
- Total posts scraped
- Date range of posts
- Format breakdown (Text-only / Text+Image / Carousel / Video — counts from PRE-PASS)

#### 1.2 Dataset Quality Check

Read directly from the PRE-PASS fact table. Do not re-read the container here.

- For each profile where `CRITICAL_DATA_FLAG: true`, write:
  `⚠ CRITICAL LOW DATA [Pn: slug] — [n] posts. All findings for this profile tagged [LOW CONFIDENCE]. This profile cannot anchor cross-profile conclusions or crossover source mappings.`

- For each profile where `LOW_DATA_FLAG: true` (and CRITICAL_DATA_FLAG: false), write:
  `⚠ LOW DATA [Pn: slug] — [n] posts. Findings are less reliable. Lower-weight input for cross-profile synthesis.`

- For each profile where `PHASE_1B_STATUS: failed`, write:
  `⚠ METADATA UNAVAILABLE [Pn: slug] — PHASE_1B_STATUS failed. HEADLINE, ABOUT, FOLLOWER_COUNT, EXPERIENCE, EDUCATION, TOP_SKILLS are all [NOT COLLECTED]. Sections 6 and 7 for this profile: post content only, tagged [HYPOTHESIS].`

- List any profiles with DATE ONLY timestamps. Write:
  `[TIMING ANALYSIS SKIPPED FOR Pn — time-of-day data not available from Apify]`

- Confirm 70/20/10 split by verifying that every post in the container carries a TIER field valued POPULAR, WORST, or RECENT. If any post is missing a TIER field, write:
  `⚠ CONTAINER INTEGRITY: [n] posts in [Pn] are missing TIER field.`

- Note any date range gaps or inconsistencies across profiles.

#### 1.3 Dataset Summary (3–5 bullets)

Write 3–5 factual bullet points summarising the full dataset:
- Who was analyzed (profiles, niche, time period)
- Total scale (post count, format variety)
- Data quality summary (how many profiles are full-data / low-data / critical / metadata-failed)

Keep this block under 80 words. Factual only. No interpretation.

> **NOTE:** This summary block will be used as the context header for all NLM batch queries in Window 1B. Write it clearly and precisely.

#### [DATA-BACKED CONCLUSIONS]

Write only findings directly supported by the dataset structure and metadata above. No prescriptions.

#### [MODEL CONJECTURES]

Write cautious extrapolations about what the dataset composition may suggest. Use "This may suggest..." framing. No prescriptions.

---

### SECTION 2 — FORMAT PERFORMANCE & POSTING STRATEGY

```
## SECTION 2 — FORMAT PERFORMANCE & POSTING STRATEGY
```

#### 2.1 Raw Format Metrics (per profile)

For each profile, produce the following table. Pull numbers directly from the container — compute medians and top decile averages from the post-level data.

| Format | Post count | Median TOTAL_REACTIONS | Median ACTIVITY_REACTIONS* | Median ENGAGEMENT_SCORE | Top decile avg ENGAGEMENT_SCORE |
|--------|-----------|----------------------|--------------------------|------------------------|--------------------------------|
| Text-only | | | | | |
| Text+Image | | | | | |
| Carousel | | | | | |
| Video | | | | | |

If a format has zero posts for a given profile, write `—` in all cells for that row.

If a profile is CRITICAL_DATA_FLAG: true, add this note below its table:
`⚠ [LOW CONFIDENCE — fewer than 20 posts. Format medians derived from thin data.]`

*`ACTIVITY_REACTIONS` is an activity-feed-derived signal — not a per-post comment count. Do not label this column as "comments."

#### 2.2 Cross-Profile Format Ranking

Produce a combined ranking of all formats by:
1. Median ENGAGEMENT_SCORE (primary sort)
2. Top decile avg ENGAGEMENT_SCORE (tiebreaker)

Exclude any profile with CRITICAL_DATA_FLAG: true from the combined medians. Note the exclusion explicitly:
`[Pn excluded from combined medians — CRITICAL_DATA_FLAG: true]`

Flag any format that ranks differently when viewed per-profile versus combined:
`⚠ FORMAT DELTA: [Format] ranks [x] overall but ranks [y] for profile P[n] — possible niche-specific signal.`

#### 2.3 Outlier Analysis

Identify the top 5 posts per format across all profiles (excluding posts from CRITICAL_DATA_FLAG profiles).

For each post, provide:
- POST_URL
- Profile (P-label + slug)
- ENGAGEMENT_SCORE
- TIER value (POPULAR / RECENT)
- 1-sentence description of why it stands out (hook type, visual pattern, or topic)

After listing outliers per format, identify consistent traits only if ≥ 3 posts within that format share the trait.
If fewer than 3 posts share a trait, write: `[THIN DATA — [n] posts share this trait. Treat as anecdote, not pattern.]`

#### 2.4 Posting Cadence & Timing

- Average posts per week per profile (from post count and date range — compute directly).
- Cadence range across full-data profiles only (LOW_DATA_FLAG or CRITICAL_DATA_FLAG profiles noted separately).
- Time-of-day and day-of-week patterns: only if the container's TIMING DATA AVAILABILITY confirms full ISO timestamps for a given profile. If DATE ONLY: write `[TIMING ANALYSIS SKIPPED FOR Pn]` and do not attempt to infer timing patterns.

#### 2.5 Format Mix Patterns

- Format portfolio per profile as % breakdown (count per format / total posts).
- Any observable correlation between format mix and FOLLOWER_COUNT — only for profiles where both are available (PHASE_1B_STATUS ≠ failed and FOLLOWER_COUNT ≠ [NOT COLLECTED]).
- Note: follower growth velocity is not available — see KNOWN DATA GAPS in container.

#### [DATA-BACKED CONCLUSIONS]

Write only findings directly tied to scraped format metrics above. No prescriptions.

#### [MODEL CONJECTURES]

Write cautious extrapolations about what the format data may suggest. No prescriptions.

---

### SECTION 3 — HOOK & NARRATIVE SYSTEMS

```
## SECTION 3 — HOOK & NARRATIVE SYSTEMS
```

#### 3.1 Hook Type Taxonomy (per profile)

Classify every post's opening sentence (from the OPENING_SENTENCE field) by hook type:
- controversy
- promise
- pain
- story
- numbers
- question
- authority
- pattern-interrupt
- other

For each profile, produce:
- Count and percentage per hook type
- Median ENGAGEMENT_SCORE per hook type

If a profile is CRITICAL_DATA_FLAG: true, add:
`⚠ [LOW CONFIDENCE — fewer than 20 posts. Hook percentages are not reliable at this sample size.]`

Then produce a combined cross-profile hook ranking by median ENGAGEMENT_SCORE. Exclude CRITICAL_DATA_FLAG profiles from combined medians and note the exclusion.

#### 3.2 Opening Sentence Analysis

List the verbatim OPENING_SENTENCE field value of the top 10 posts per profile.

For each:
- Identify the structural pattern if detectable.
- Note what makes it distinct versus a generic opener.

Do not paraphrase. Use the verbatim field value.

#### 3.3 Narrative Structure Types

Classify each post's narrative structure:
- problem-solution
- before-after-bridge
- listicle
- case study
- contrarian take
- pure story
- other

For each profile:
- Count per narrative type
- Median ENGAGEMENT_SCORE per narrative type

CRITICAL_DATA_FLAG profiles: add `[LOW CONFIDENCE]` note below per-profile table.

#### 3.4 Hook × Narrative Coupling

- Which hook types pair with which narrative structures in the top 20% of posts by ENGAGEMENT_SCORE?
- Any hook × narrative combinations in the bottom 20% of posts?

**Pattern floor:** A hook × narrative combination counts as a finding only if ≥ 3 posts share it within the relevant tier. Below 3 posts: write `[THIN DATA — [n] posts. Not a pattern.]`

- Cross-profile consensus: state explicitly whether the coupling pattern holds across ≥ 3 full-data profiles, or is isolated to one.

#### 3.5 Per-Profile vs Cross-Profile Delta

- Hook and narrative patterns that appear in ≥ 3 profiles — note as cross-profile signal.
- Patterns unique to one profile — note as profile-specific. Do not elevate to niche-wide conclusion.

#### [DATA-BACKED CONCLUSIONS]

Write only findings directly tied to scraped hook and narrative data above. No prescriptions.

#### [MODEL CONJECTURES]

Write cautious extrapolations about what the hook/narrative data may suggest. No prescriptions.

---

### SECTION 4 — CTA & CONVERSION STRATEGY

```
## SECTION 4 — CTA & CONVERSION STRATEGY
```

#### 4.1 CTA Type Inventory (per profile)

All CTA types found in post FULL_TEXT. Categories:
- comment
- follow
- DM
- newsletter
- external link
- profile visit
- tag someone
- none

For each profile: count and percentage per CTA type.

#### 4.2 CTA Placement Analysis

Classify each CTA by placement zone:
- Opening CTA (first 2 lines of post)
- Mid CTA (post body)
- Closing CTA (final line or sign-off)

Per profile:
- Count per placement zone
- Frequency of single vs multiple CTAs per post
- Count of posts with zero CTAs and their median ENGAGEMENT_SCORE vs posts with at least one CTA

#### 4.3 Explicit vs Soft CTA Performance

Definitions:
- **Explicit CTA**: "Click the link," "Follow me for more," "Subscribe now"
- **Soft CTA**: "What do you think?" "Drop a comment below," "Tag someone who needs this"

Produce:
- Median ENGAGEMENT_SCORE for explicit CTAs — per profile and combined
- Median ENGAGEMENT_SCORE for soft CTAs — per profile and combined

#### 4.4 CTA Presence in Top vs Bottom Performers

- CTA type distribution in TIER: POPULAR posts
- CTA type distribution in TIER: WORST posts
- Any CTA type that appears exclusively in one tier — flag it explicitly. Minimum: ≥ 3 posts of that type in that tier.

#### 4.5 Per-Profile vs Cross-Profile Delta

- CTA patterns that appear in ≥ 3 profiles — note as cross-profile signal.
- Profile-specific anomalies — note as such. Do not generalize.

#### [DATA-BACKED CONCLUSIONS]

Write only findings directly tied to scraped CTA data above. No prescriptions.

#### [MODEL CONJECTURES]

Write cautious extrapolations about what the CTA data may suggest. No prescriptions.

---

### SECTION 5 — COMMENT & COMMUNITY ENGINE

```
## SECTION 5 — COMMENT & COMMUNITY ENGINE

SCOPE CHECK — run before writing any subsection:
Check the container for enriched posts (posts where COMMENTS field contains actual
comment data, not "PENDING — Phase 2B enrichment required").

If no enriched posts exist in the container:
  Write: [NO ENRICHED POST DATA — Phase 2B was not run. Section 5 analysis
  cannot be performed. All subsections below are skipped.]
  Do not attempt to fill any subsection.
  Proceed directly to Section 6.

If enriched posts exist: note how many per profile and continue.
All analysis in this section is scoped to enriched posts only.
Findings must not be extrapolated to the full post set.
```

#### 5.1 Comment Volume by Format

- Median comment count per format type — for enriched posts only.
- State the enriched post count per format explicitly before stating any median.
- Formats with fewer than 3 enriched posts: write `[THIN DATA — [n] enriched posts in this format. Median not meaningful.]`

#### 5.2 Creator Reply Behavior

For each profile across enriched posts:
- **Reply rate**: percentage of enriched posts where the creator replied to at least one comment.
- **Reply speed**: v08 does not write a minutes-elapsed field. If Phase 2B enrichment data includes comment timestamps and the post's POST_DATE is a full ISO datetime (not DATE ONLY), elapsed time may be calculated from those. Otherwise write: `[REPLY SPEED NOT CALCULABLE — field not available in v08 container]`
- **Reply depth**: average word count per creator reply (count from verbatim reply text).
- **Reply style**: classify each creator reply as one of:
  - question-back
  - insight-add
  - acknowledgment
  - none
  Summarise the dominant style per profile. Minimum: ≥ 3 creator replies to establish a pattern. Fewer: write `[THIN DATA — [n] replies. No dominant style determinable.]`

#### 5.3 High-Leverage Reply Examples

Identify 3–5 specific instances where a creator reply demonstrably generated follow-on comments in the thread.

For each:
- Profile slug and POST_URL
- Verbatim creator reply text (from container — do not paraphrase)
- Observable pattern: question seeding / insight drop / community invitation / other

If fewer than 3 such instances exist across all enriched posts, write:
`[INSUFFICIENT EXAMPLES — only [n] qualifying reply chains found. Cannot establish pattern.]`

#### 5.4 Comment Patterns in Top Enriched Posts

- Average audience comment count per enriched post per profile.
- Note any observable pattern in top-performing enriched post comments — only where clearly detectable from text. If not detectable, write `[NOT DETERMINABLE FROM AVAILABLE TEXT]`.

#### 5.5 Observed Comment Playbooks

Identify recurring creator behaviors across enriched posts:
- Seeding questions in comments
- Dropping mini-insights or data in comments
- Pinned comments used for CTA or context

A playbook requires ≥ 2 profiles showing the same behavior to be called a cross-profile playbook. Otherwise note it as profile-specific.

#### 5.6 Per-Profile vs Cross-Profile Delta

- Comment and reply patterns confirmed in ≥ 2 profiles — cross-profile signal.
- Patterns found in only 1 profile — note as profile-specific. Do not generalize.

#### [DATA-BACKED CONCLUSIONS]

Write only findings directly tied to enriched post comment data above. No prescriptions.

#### [MODEL CONJECTURES]

Write cautious extrapolations about what the comment/community data may suggest. No prescriptions.

---

### SECTION 6 — PROFILE POSITIONING & FUNNEL

```
## SECTION 6 — PROFILE POSITIONING & FUNNEL
```

#### 6.1 Headline Pattern Analysis

For each profile:
- Verbatim HEADLINE field from container.
- If PHASE_1B_STATUS: failed or HEADLINE is [NOT COLLECTED]: write `[NOT AVAILABLE — PHASE_1B_STATUS failed]` and skip that profile for this subsection.
- Structural formula identified per profile.
- Keyword frequency table: list every keyword appearing in ≥ 2 headlines across the profile set (metadata-available profiles only).

#### 6.2 About Section Structure

For each profile:
- If PHASE_1B_STATUS: failed or ABOUT is [NOT COLLECTED]: write `[NOT AVAILABLE FOR THIS PROFILE — PHASE_1B_STATUS failed]` and skip.
- Word count
- Components present — mark YES or NO for each:
  - Hook opener
  - Credibility statement
  - Niche statement
  - Social proof
  - CTA
- Tone classification: personal story / authority / results-led / community-led

#### 6.3 Featured Section Usage

⚠ DATA UNAVAILABLE — v08 pipeline does not collect featured section data.
The harvestapi/linkedin-profile-scraper actor does not return featured section items.
This field is listed in the PIPELINE CONTRACT as permanently unavailable.

For each profile, write:
`Featured section: [NOT COLLECTED — not available from v08 pipeline]`

Do not infer featured section content from posts or about text. This subsection produces no analysis output.

#### 6.4 Offer & Funnel CTA in Profile

**Per profile:**

v08 does not write a dedicated OFFER or PRIMARY CTA field.

- If ABOUT is available (PHASE_1B_STATUS ≠ failed): scan verbatim ABOUT text for explicit CTAs, links, or offer statements. Quote verbatim only. Do not paraphrase. Tag each item [HYPOTHESIS].
- If ABOUT is [NOT COLLECTED]: write `[NOT INFERABLE — ABOUT not available for this profile]`

**Cross-profile pattern claim — hard gate:**

A cross-profile offer/funnel pattern claim is only permitted when ≥ 3 profiles have explicit CTA or offer evidence in their ABOUT text.

If ≥ 3 profiles qualify: state the pattern, list the qualifying profiles, tag [HYPOTHESIS].
If fewer than 3 profiles qualify: write `[INSUFFICIENT EVIDENCE — only [n] profiles have ABOUT-level CTA evidence. No cross-profile offer pattern can be stated.]`

#### 6.5 Niche Positioning Archetypes

For each profile, assign one archetype:
- practitioner-educator
- case-study storyteller
- data-driven thought leader
- contrarian expert
- community builder
- other (describe in one sentence)

**Assignment rules:**

- PHASE_1B_STATUS: success or partial AND post count ≥ 20: assign from HEADLINE + ABOUT + post content. Tag [DATA].
- PHASE_1B_STATUS: failed AND post count ≥ 20: assign from post content only. Tag [HYPOTHESIS — no metadata available].
- PHASE_1B_STATUS: failed AND CRITICAL_DATA_FLAG: true (< 20 posts): write `[INSUFFICIENT DATA — metadata failed and fewer than 20 posts. Archetype assignment not possible.]`

Cross-profile archetype patterns: note any archetype appearing in ≥ 2 profiles.

#### 6.6 Per-Profile vs Cross-Profile Delta

- Positioning patterns shared across ≥ 3 profiles — cross-profile signal.
- Patterns unique to one profile — note as such.

#### [DATA-BACKED CONCLUSIONS]

Write only findings directly tied to scraped profile data above. No prescriptions.

#### [MODEL CONJECTURES]

Write cautious extrapolations. No prescriptions.

---

### SECTION 7 — NICHE-BENDING & FORMAT×TONE CROSSOVERS

```
## SECTION 7 — NICHE-BENDING & FORMAT×TONE CROSSOVERS

IMPORTANT: Per-profile breakdown is mandatory throughout this section.
Cross-profile blending destroys niche-bending signal.
Never merge profiles before completing Sections 7.1 and 7.2.

DATA QUALITY GATES — apply before any mapping:
- CRITICAL_DATA_FLAG: true profiles → all tone/voice characteristics tagged [LOW CONFIDENCE]
- PHASE_1B_STATUS: failed profiles → tone mapping from post content only; tagged [HYPOTHESIS — no metadata]
- CRITICAL_DATA_FLAG: true profiles → MUST NOT serve as "Format from" or "Tone from"
  source anchors in Section 7.3 crossover opportunities
```

#### 7.1 Format Style Source Mapping (per profile)

For each profile:
- Identify the format this profile uses most distinctively (by count and ENGAGEMENT_SCORE pattern).
- Describe signature format characteristics specifically.
  Example: "P1 owns long-form carousel with data tables and bold headers."
  Example: "P3 owns short text-only posts with single-sentence paragraphs and no punctuation on final line."
- If CRITICAL_DATA_FLAG: true: append `[LOW CONFIDENCE — fewer than 20 posts. Format ownership claim is tentative.]`

#### 7.2 Tone & Voice Source Mapping (per profile)

For each profile, assign:
- **Tone**: one of: analytical / motivational / conversational / contrarian / storytelling / tactical
- **Voice characteristics**:
  - Sentence length pattern (short punchy / medium balanced / long flowing)
  - Emoji density (none / occasional / frequent)
  - Formality level (formal / semi-formal / informal)
  - Punctuation style (minimal / standard / expressive)

**Confidence rules:**

- Post count ≥ 80: assign characteristics without qualification.
- Post count 20–79 (LOW_DATA_FLAG: true): assign characteristics, append `[LOWER CONFIDENCE — [n] posts]` to each characteristic.
- Post count < 20 (CRITICAL_DATA_FLAG: true): assign tentatively, append `[LOW CONFIDENCE — fewer than 20 posts]` to every characteristic.
- PHASE_1B_STATUS: failed: derive from post content only. Append `[HYPOTHESIS — no HEADLINE or ABOUT available]` to tone and all voice characteristics.

#### 7.3 Identified Crossover Opportunities

Based on the per-profile maps in 7.1 and 7.2, identify specific untested format × tone combinations.

**Source anchor eligibility:**
A profile may only serve as a "Format from" or "Tone from" anchor if:
- CRITICAL_DATA_FLAG: false (post count ≥ 20), AND
- The specific characteristic being sourced is not tagged [LOW CONFIDENCE] or [HYPOTHESIS]

If no eligible anchor profiles exist for a given crossover, do not fabricate one. Write:
`[NO ELIGIBLE CROSSOVER ANCHORS — all profiles with this format/tone characteristic are below the data threshold for confident sourcing.]`

Format each valid opportunity as:
```
CROSSOVER OPPORTUNITY:
Format from: P[x] — [format name + signature characteristics]
Tone from: P[y] — [tone + voice characteristics]
Anchor eligibility: P[x] — [post count] posts | P[y] — [post count] posts [both eligible]
Why this combination does not currently appear in the analyzed set: [explanation]
Opportunity strength: High / Medium / Speculative
Reasoning: [1–2 sentences]
```

#### 7.4 Underserved Combinations in the Niche

Format × Tone pairings that none of the analyzed profiles use.

For each:
- Name the pairing
- Assess whether it is a genuine gap or deliberate avoidance
- Tag:
  - `[DATA]` — absence confirmed across all eligible (non-CRITICAL) profiles
  - `[MODEL CONJECTURES]` — inferred, or only confirmed across CRITICAL/LOW profiles

#### [DATA-BACKED CONCLUSIONS]

Write only findings tied to per-profile format and tone data above. No prescriptions.

#### [MODEL CONJECTURES]

Write cautious extrapolations. No prescriptions.

---

### SECTION 8 — RISK, WEAK SIGNALS & OPEN QUESTIONS

```
## SECTION 8 — RISK, WEAK SIGNALS & OPEN QUESTIONS
```

#### 8.1 Thin Data Patterns

List every finding from Sections 2–7 where the evidence count is below the relevant minimum floor from the PIPELINE CONTRACT.

Format per entry:
```
Finding: [state the finding]
Evidence count: [n posts]
Required minimum: [n per PIPELINE CONTRACT]
Affected section: [Section x.x]
Confidence level: Low / Medium
```

#### 8.2 Cross-Profile Conflicts

List every instance where two or more profiles directly contradict each other on the same pattern.

Format per conflict:
```
Conflict: [P[x] shows pattern A / P[y] shows pattern B on the same dimension]
Data quality of conflicting profiles: [flag statuses]
Possible explanations:
  - Audience maturity difference
  - Niche sub-segment difference
  - Execution quality difference
  - Timing difference
  [State the most likely 1–2 and briefly justify]
```

#### 8.3 Explicit Unknowns

Copy the KNOWN DATA GAPS block from the container verbatim here.

For each gap listed, write:
```
Gap: [gap name from container]
Findings weakened by this gap: [list specific sections and subsections]
```

The following gaps are permanent in all v08 containers and must always appear here:
- `featured_section_items: not available` → weakens Section 6.3 (no data) and Section 6.4 (ABOUT inference only, tagged [HYPOTHESIS])
- `follower_growth_velocity: not available` → weakens Section 2.5 (format mix vs follower count)
- `post_impressions_reach: not available` → ENGAGEMENT_SCORE is a proxy, not a reach metric
- `comment thread truth: ACTIVITY_FEED only` → weakens all ACTIVITY_REACTIONS-based medians in Section 2.1; true comment data limited to Section 5 enriched posts only

#### 8.4 Sampling Bias Flags

- List any obvious dataset skew: too few profiles / short date range / one niche sub-segment over-represented.
- For each skew: name the sections most affected.
- List all profiles with PHASE_1B_STATUS: failed and note their absence from Sections 6 and 7 positioning analysis.
- List all profiles with CRITICAL_DATA_FLAG: true and note their exclusion from combined medians and crossover anchoring.

#### 8.5 Recommended Pre-Commitment Experiments

Specific A/B tests to run before treating any [MODEL CONJECTURES] finding as strategy.

Format per experiment:
```
Test: [A] vs [B]
Duration: [n posts or n weeks]
Metric: [what to measure]
Relevant finding: [Section x.x — MODEL CONJECTURES]
```

#### [DATA-BACKED CONCLUSIONS]

Write only findings directly tied to data quality and conflict evidence above. No prescriptions.

#### [MODEL CONJECTURES]

Write cautious extrapolations about what the risk and gap patterns may suggest. No prescriptions.

---

## Window 1A Completion

After all 8 sections are fully written, save REPORT_1.md and print:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ WINDOW 1A COMPLETE
REPORT_1.md saved and locked: [full path]
DO NOT edit this file. It is now read-only.

Open a new session (same model).
Paste the Window 1B startup instruction below.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

WINDOW 1B STARTUP INSTRUCTION:
"Read v05-lnkdn-analysis.md and run Window 1B.
REPORT_1 = [full path to REPORT_1.md]
USER_GOAL = [USER_GOAL]
USER_NICHE = [USER_NICHE]"
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

# WINDOW 1B: GENERATE VALIDATION_LAYER.md

## Window 1B Rules

- The PIPELINE CONTRACT at the top of this file is authoritative.
- Open and read REPORT_1.md fully before firing any NLM calls.
- Extract the Section 1.3 Dataset Summary (the 3–5 bullets) — this becomes the context header for every batch query.
- NotebookLM's notebook contains only the 63 expert sources. Scraped competitor data and REPORT_1.md are **never** added as NLM sources.
- Fire 4 CLI calls in sequence — **never in parallel**.
- VALIDATION_LAYER.md is organised by section heading, not by batch. Split each batch response into its per-section headings when saving.
- VALIDATION_LAYER.md is read-only the moment it is saved.
- If quota interrupts mid-batch: read partial VALIDATION_LAYER.md, identify last completed section, continue from the next batch. Never re-run completed batches.
- If any NLM response recommends an engagement rate metric or references per-follower calculations: record it under CORPUS ADDITIONS but add this note: `[EXCLUDED FROM SYNTHESIS — banned metric per PIPELINE CONTRACT]`

## Window 1B Terminal Output

Print at the start of Window 1B:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔍 WINDOW 1B: NLM VALIDATION — 4 CLI BATCHES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Print before each batch:

```
Firing Batch [n]/4: Sections [x + y]...
```

Print after each batch:

```
✓ Batch [n] complete — saving to VALIDATION_LAYER.md
```

---

## NLM CLI Batch Prompt Template

Use this exact template for every batch call. Fill in the variable fields from REPORT_1.md:

```
DATASET CONTEXT:
[Paste the Section 1.3 Dataset Summary verbatim — 3–5 bullets from REPORT_1.md]

REPORT SECTIONS FOR REVIEW:
[Paste the relevant REPORT_1 section(s) verbatim — full section text including all subsections,
DATA-BACKED CONCLUSIONS, and MODEL CONJECTURES blocks]

REVIEW INSTRUCTIONS:
You are reviewing a data-driven LinkedIn competitor analysis against your expert knowledge
sources on LinkedIn strategy and personal branding.

Respond using EXACTLY these six headings in this exact order.
If you have nothing to add under a heading, write [NOTHING TO ADD].
Do not skip any heading. Do not merge headings.

AGREEMENTS
What your sources confirm or strongly support from the findings above.
Be specific — name the pattern and confirm it.

CONTRADICTIONS
Where your sources say something meaningfully different from what the data shows.
State the contradiction clearly and explain one or two likely reasons it exists.

ADDITIONS
Important tactics, nuances, or patterns from your sources that are not present in the
analysis above and would strengthen the strategy.

MISSING FROM YOUR ANALYSIS
Critical factors your sources consistently treat as essential in this domain that the
report did not address at all.

COMMON TRAPS / WARNINGS
Specific failure patterns or mistakes your sources warn about in this strategic domain.
Be concrete — name the trap.

DATA GAPS
Specific claims or questions in the report you cannot evaluate because the data or
context is insufficient. State exactly what is missing.
```

---

## Batch Mapping

```
Batch 1: Section 2 + Section 3
Batch 2: Section 4 + Section 5
Batch 3: Section 6 + Section 7
Batch 4: Section 8 only
```

Section 1 is never sent as a query target. Its 3–5 bullet summary is the context header for all batches.

---

## VALIDATION_LAYER.md — Full File Contents

Write the following as VALIDATION_LAYER.md, top to bottom, in full:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
VALIDATION LAYER — NLM CORPUS REVIEW
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Generated: [timestamp]
REPORT_1 reviewed: [full path]
NLM batches fired: 4
Sections reviewed: 7 (Sections 2–8)
Note: Section 1 used as context header only — not a review target.
THIS FILE IS READ-ONLY. Do not edit after generation.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 2 — FORMAT PERFORMANCE & POSTING STRATEGY [from Batch 1]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### AGREEMENTS
[NLM response — scoped to Section 2]

### CONTRADICTIONS
[NLM response — scoped to Section 2]

### ADDITIONS
[NLM response — scoped to Section 2]

### MISSING FROM YOUR ANALYSIS
[NLM response or NOTHING TO ADD]

### COMMON TRAPS / WARNINGS
[NLM response — scoped to Section 2]

### DATA GAPS
[NLM response or NOTHING TO ADD]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 3 — HOOK & NARRATIVE SYSTEMS [from Batch 1]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### AGREEMENTS
[NLM response — scoped to Section 3]

### CONTRADICTIONS
[NLM response — scoped to Section 3]

### ADDITIONS
[NLM response — scoped to Section 3]

### MISSING FROM YOUR ANALYSIS
[NLM response or NOTHING TO ADD]

### COMMON TRAPS / WARNINGS
[NLM response — scoped to Section 3]

### DATA GAPS
[NLM response or NOTHING TO ADD]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 4 — CTA & CONVERSION STRATEGY [from Batch 2]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### AGREEMENTS
[NLM response — scoped to Section 4]

### CONTRADICTIONS
[NLM response — scoped to Section 4]

### ADDITIONS
[NLM response — scoped to Section 4]

### MISSING FROM YOUR ANALYSIS
[NLM response or NOTHING TO ADD]

### COMMON TRAPS / WARNINGS
[NLM response — scoped to Section 4]

### DATA GAPS
[NLM response or NOTHING TO ADD]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 5 — COMMENT & COMMUNITY ENGINE [from Batch 2]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### AGREEMENTS
[NLM response — scoped to Section 5]

### CONTRADICTIONS
[NLM response — scoped to Section 5]

### ADDITIONS
[NLM response — scoped to Section 5]

### MISSING FROM YOUR ANALYSIS
[NLM response or NOTHING TO ADD]

### COMMON TRAPS / WARNINGS
[NLM response — scoped to Section 5]

### DATA GAPS
[NLM response or NOTHING TO ADD]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 6 — PROFILE POSITIONING & FUNNEL [from Batch 3]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### AGREEMENTS
[NLM response — scoped to Section 6]

### CONTRADICTIONS
[NLM response — scoped to Section 6]

### ADDITIONS
[NLM response — scoped to Section 6]

### MISSING FROM YOUR ANALYSIS
[NLM response or NOTHING TO ADD]

### COMMON TRAPS / WARNINGS
[NLM response — scoped to Section 6]

### DATA GAPS
[NLM response or NOTHING TO ADD]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 7 — NICHE-BENDING & FORMAT×TONE CROSSOVERS [from Batch 3]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### AGREEMENTS
[NLM response — scoped to Section 7]

### CONTRADICTIONS
[NLM response — scoped to Section 7]

### ADDITIONS
[NLM response — scoped to Section 7]

### MISSING FROM YOUR ANALYSIS
[NLM response or NOTHING TO ADD]

### COMMON TRAPS / WARNINGS
[NLM response — scoped to Section 7]

### DATA GAPS
[NLM response or NOTHING TO ADD]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 8 — RISK, WEAK SIGNALS & OPEN QUESTIONS [from Batch 4]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### AGREEMENTS
[NLM response]

### CONTRADICTIONS
[NLM response]

### ADDITIONS
[NLM response]

### MISSING FROM YOUR ANALYSIS
[NLM response or NOTHING TO ADD]

### COMMON TRAPS / WARNINGS
[NLM response]

### DATA GAPS
[NLM response or NOTHING TO ADD]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
END OF VALIDATION LAYER
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Window 1B Completion

After all 4 batches are saved to VALIDATION_LAYER.md, print:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ WINDOW 1B COMPLETE
VALIDATION_LAYER.md saved and locked: [full path]
DO NOT edit this file. It is now read-only.

Open a new session (same model).
Paste the Window 2 startup instruction below.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

WINDOW 2 STARTUP INSTRUCTION:
"Read v05-lnkdn-analysis.md and run Window 2.
REPORT_1 = [full path to REPORT_1.md]
VALIDATION_LAYER = [full path to VALIDATION_LAYER.md]
USER_GOAL = [USER_GOAL]
USER_NICHE = [USER_NICHE]"
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

# WINDOW 2: GENERATE REPORT_2.md

## Window 2 Rules

- The PIPELINE CONTRACT at the top of this file is authoritative. Re-read it before writing anything.
- Read REPORT_1.md and VALIDATION_LAYER.md fully before writing anything.
- Write REPORT_2.md in one complete pass — do not stop mid-file.
- **Conflict hierarchy: Data > Corpus > Model**
  - **Data** (scraped metrics from REPORT_1.md) is the primary source of truth.
  - **Corpus** (NLM AGREEMENTS / ADDITIONS from VALIDATION_LAYER.md) can reinforce or enrich Data findings.
  - **Corpus CONTRADICTIONS** override Data only when the dataset is narrow (fewer than 20 posts for that specific finding) or when the corpus has repeated strong warnings about that pattern.
  - **Model** fills true gaps where neither Data nor Corpus answers — always tagged [HYPOTHESIS].
- Every recommendation carries exactly one provenance tag:
  - `[DATA]` — finding directly supported by scraped metrics
  - `[CORPUS]` — from NLM expert sources only
  - `[DATA+CORPUS]` — supported by both
  - `[HYPOTHESIS]` — model inference, no direct data or corpus support
- **[HYPOTHESIS] tags must always appear in the final output. Never strip them.**
- [DATA], [CORPUS], [DATA+CORPUS] tags may be removed in a clean SOP read — keep them in REPORT_2.md.
- Any NLM-recommended engagement rate metric noted in VALIDATION_LAYER as `[EXCLUDED FROM SYNTHESIS]` must not appear in REPORT_2.md. Do not reference it.
- Conflicts between Data and Corpus become testable experiments. Do not resolve them by picking a side.
- DATA GAPS and [THIN DATA] flags from REPORT_1.md must be acknowledged — never filled with inference.
- Any recommendation based on a finding tagged [LOW CONFIDENCE] in REPORT_1.md must carry [HYPOTHESIS] in REPORT_2.md regardless of its provenance tag in the source.
- If quota interrupts: read partial REPORT_2.md, identify last completed section, continue from next. Never re-run completed sections.

## Window 2 Terminal Output

Print at the start of Window 2:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🧠 WINDOW 2: SYNTHESIZING REPORT_2.md
Reading REPORT_1.md + VALIDATION_LAYER.md...
Conflict hierarchy: Data > Corpus > Model
Pipeline Contract: active
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## REPORT_2.md — Full File Contents

Write the following as REPORT_2.md, top to bottom, in full:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
REPORT 2 — LINKEDIN STRATEGY SYNTHESIS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Generated: [timestamp]
Source: REPORT_1.md + VALIDATION_LAYER.md
User goal: [USER_GOAL]
User niche: [USER_NICHE]

Provenance tags:
  [DATA]         — scraped metrics
  [CORPUS]       — NLM expert sources only
  [DATA+CORPUS]  — both
  [HYPOTHESIS]   — model inference; no direct data or corpus support
                   ⚠️  Never strip [HYPOTHESIS] tags.

Note: Any recommendation derived from a [LOW CONFIDENCE] source finding
carries [HYPOTHESIS] regardless of other provenance.

Conflict resolution: Data > Corpus > Model
Conflicts = experiments, not resolved arbitrarily.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

TABLE OF CONTENTS
  PART 1: LINKEDIN_PROFILE_SOP
    SOP-1. Profile Architecture
    SOP-2. Content Format Strategy
    SOP-3. Hook & Narrative Playbook
    SOP-4. CTA System
    SOP-5. Community Engine
    SOP-6. Niche Positioning
    SOP-7. Experiments Queue

  PART 2: LINKEDIN_CONTENT_IDEAS
    CI-1. Format Playbooks
    CI-2. Hook Library
    CI-3. Starter Content Set
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### PART 1: LINKEDIN PROFILE SOP

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PART 1: LINKEDIN PROFILE SOP
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

#### SOP-1: Profile Architecture

*Sources: Report 1 Section 6 + Validation Layer Section 6*

1. **Headline formula recommendation**
   From Section 6.1 data. Tag.

2. **About section structure checklist**
   Components in recommended order, each tagged individually. Based on Section 6.2 data.

3. **Featured section strategy**
   Write: `Featured section strategy: [NOT AVAILABLE — featured_section_items not in v08 pipeline. No data-backed recommendation possible. Any guidance here would be [HYPOTHESIS] only — omitted.]`

4. **Primary CTA recommendation — two separate items:**

   4a. *Post-level CTA* (from Section 4 data):
   State the CTA type with strongest signal in TIER: POPULAR posts. Tag [DATA] or [DATA+CORPUS].

   4b. *Profile-level CTA inferred from ABOUT* (from Section 6.4):
   State the most common offer type inferred from ABOUT text, with the qualifying profile count.
   This item must always be tagged [HYPOTHESIS].
   If Section 6.4 produced [INSUFFICIENT EVIDENCE], write: `Profile-level CTA: [INSUFFICIENT EVIDENCE — fewer than 3 profiles had explicit ABOUT-level CTA evidence. No profile CTA recommendation can be stated.]`

   Do not merge 4a and 4b into a single tag or a single statement.

5. **Niche positioning archetype recommendation**
   From Sections 6.5 and 7.x. Note how many profiles were eligible (metadata available, post count ≥ 20). If fewer than 3 eligible profiles, tag [HYPOTHESIS]. Tag.

6. **NLM Additions from Section 6**
   Any ADDITIONS from VALIDATION_LAYER Section 6 not present in the raw data. Tag each [CORPUS].

7. **Contradictions from Section 6 — converted to experiments**
   For each CONTRADICTION in VALIDATION_LAYER Section 6:
   ```
   EXPERIMENT: Test [Data approach] vs [Corpus approach] over [timeframe]
   ```

---

#### SOP-2: Content Format Strategy

*Sources: Report 1 Section 2 + Validation Layer Section 2*

1. **Primary format recommendation**
   Highest combined signal (median ENGAGEMENT_SCORE + top decile avg) from full-data profiles, corpus-supported. Tag.

2. **Secondary format recommendation**
   Next best format. Tag.

3. **Format to deprioritize**
   Lowest engagement signal and/or corpus warnings. Tag.

4. **Posting frequency recommendation**
   Based on Section 2.4 cadence data from full-data profiles only. Tag.

5. **Format mix target (% breakdown)**
   Based on Section 2.5 data. Tag each percentage.

6. **Contradictions from Section 2 — experiments**
   ```
   EXPERIMENT: Test [Data approach] vs [Corpus approach] over [timeframe]
   ```

---

#### SOP-3: Hook & Narrative Playbook

*Sources: Report 1 Section 3 + Validation Layer Section 3 + Section 7 crossovers*

1. **Top 3 hook types to lead with**
   Ranked by median ENGAGEMENT_SCORE from Section 3.1 (full-data profiles). For each:
   - Name, data summary in one sentence, provenance tag.

2. **Narrative structure recommendation by content type**
   For each recommended content type, state the best-paired narrative structure from Section 3.3. Tag each.

3. **Hook × Narrative combinations to use**
   Combinations from Section 3.4 that met the ≥ 3 post minimum. Tag.

4. **Hook × Narrative combinations to avoid**
   Combinations from Section 3.4 (bottom 20%) that met the ≥ 3 post minimum. Tag.
   If none met the minimum: write `[INSUFFICIENT EVIDENCE — no avoidance pattern met the 3-post minimum floor.]`

5. **Crossover opportunities from Section 7**
   Only crossovers with eligible source anchors (per Section 7.3 rules). Tag [DATA] or [HYPOTHESIS].

---

#### SOP-4: CTA System

*Sources: Report 1 Section 4 + Validation Layer Section 4*

1. **Primary CTA type recommendation** — highest signal in TIER: POPULAR posts per Section 4.4. Tag.
2. **CTA placement recommendation** — per Section 4.2 data. Tag.
3. **Explicit vs soft CTA guidance** — per Section 4.3 data. Tag.
4. **CTA-per-post frequency recommendation** — per Section 4.2 data. Tag.
5. **NLM TRAPS from Section 4** — copy each COMMON TRAPS / WARNINGS item verbatim. Tag each [CORPUS].

---

#### SOP-5: Community Engine

*Sources: Report 1 Section 5 + Validation Layer Section 5*

```
NOTE: All recommendations in SOP-5 are based on enriched posts only (Phase 2B).
If Section 5 was skipped due to absent enrichment data, write:
[SOP-5 NOT AVAILABLE — Section 5 was skipped. No Phase 2B enrichment data in container.]
and proceed to SOP-6.
```

1. **Creator reply strategy**
   - Reply speed: if calculable from enrichment timestamps, state recommendation. Otherwise: `[REPLY SPEED NOT CALCULABLE]`. Tag.
   - Reply depth (word count target from Section 5.2). Tag.
   - Reply style (from Section 5.2 — only if ≥ 3 replies classified). Tag.

2. **Reply playbook** — pattern from Section 5.3 examples. Tag.

3. **Comment volume expectations per format** — from Section 5.1 (enriched posts only). Tag.

4. **NLM Additions from Section 5** — tag each [CORPUS].

---

#### SOP-6: Niche Positioning

*Sources: Report 1 Sections 6 + 7 + Validation Layer Sections 6 + 7*

1. **Recommended positioning archetype for USER_NICHE**
   Based on Section 6.5 (eligible profiles only). State how many profiles contributed. Tag.
   If fewer than 3 eligible profiles: tag [HYPOTHESIS].

2. **Differentiating format × tone crossover opportunity to own**
   Single most actionable opportunity from Section 7.3 — only if both source anchors were eligible. Tag.
   If no eligible crossover exists: write `[NO ELIGIBLE CROSSOVER — insufficient anchor profiles met data threshold.]`

3. **Underserved combination to test**
   From Section 7.4. Tag [DATA] or [HYPOTHESIS] as determined there.

4. **What to avoid (over-represented by competitors)**
   From Sections 7.1–7.2, full-data profiles only. Tag.

---

#### SOP-7: Experiments Queue

*Sources: All CONTRADICTIONS from Validation Layer Sections 2–7*

List every Data vs Corpus conflict as a structured experiment.

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
EXPERIMENT [n]:
Hypothesis:        [what we think will happen if we follow the Data approach]
Test:              [A — Data approach] vs [B — Corpus approach]
Duration:          [n posts or n weeks]
Metric:            [specific: ENGAGEMENT_SCORE / comment count / follow rate / etc.]
Source of conflict: Data said [X] / Corpus said [Y]
Relevant section:  [Section x.x + Validation Layer Section x.x]
Priority:          High / Medium / Low
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

One entry per conflict. Do not merge. If no contradictions for a section pair:
`[No contradictions recorded — Data and Corpus in agreement.]`

---

### PART 2: LINKEDIN CONTENT IDEAS

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PART 2: LINKEDIN CONTENT IDEAS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

#### CI-1: Format Playbooks

For each recommended format from SOP-2 (primary and secondary at minimum):

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
FORMAT PLAYBOOK: [Format name]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Structural template:
Step 1: [Hook line — specify hook type]
Step 2: [Context / setup]
Step 3: [Core payload]
Step 4: [Landing line / punchline]
Step 5: [CTA]
(Add or remove steps as the format warrants.)

Best hook types to pair: [from SOP-3, one sentence per type]
Best narrative structure: [from SOP-3, one sentence]

Estimated engagement benchmark:
Median ENGAGEMENT_SCORE: [x] | Top decile avg: [y]
[State clearly which profiles this is drawn from; note if any profiles were excluded]

Example post idea scoped to USER_NICHE:
[Topic, hook angle, core argument or payload, CTA — not a full draft]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

#### CI-2: Hook Library

20 ready-to-use hook opening sentences scoped to USER_NICHE and USER_GOAL.

Organised by hook type — 5 per top-performing type (Section 3.1 ranking, full-data profiles). Use only the top 4 hook types.

```
─────────────────────────────────────────────────────
Hook [n] of 20
Type:            [hook type]
Opening:         "[verbatim ready-to-use sentence — written for USER_NICHE]"
Pairs best with: [narrative structure]
Provenance:      [DATA — structurally modeled from a Section 3.2 top performer] /
                 [HYPOTHESIS — original, not modeled on a specific post]
─────────────────────────────────────────────────────
```

Rules:
- Every hook immediately usable — scoped to USER_NICHE, no generic filler.
- [DATA] hooks model structure from a Section 3.2 OPENING_SENTENCE — same structural pattern, different angle. Not verbatim copy.
- [HYPOTHESIS] hooks are original constructions using the top-performing hook types.

---

#### CI-3: Starter Content Set

10 specific post ideas scoped to USER_NICHE and USER_GOAL.

Each must use a format + hook + narrative combination from the SOP.

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
POST IDEA [n] of 10
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Format:           [from SOP-2]
Hook type:        [from SOP-3]
Opening sentence: "[verbatim — written for USER_NICHE]"
Narrative:        [from SOP-3]
Core angle:       [1 sentence, specific to USER_NICHE]
CTA:              [from SOP-4]
Provenance:       [tag]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Rules:
- No two posts use the identical format + hook + narrative combination.
- At least 3 posts use the primary format from SOP-2.
- At least 2 posts use the secondary format from SOP-2.
- At least 1 post uses a crossover format × tone from SOP-6 (only if an eligible crossover exists; if not, replace with a [HYPOTHESIS]-tagged original combination and note why).
- Every opening sentence is immediately usable, written for USER_NICHE.

---

## Window 2 Completion

After REPORT_2.md is fully written, print:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓✓✓ PHASE 2 COMPLETE (v05) ✓✓✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

FILES GENERATED:
  REPORT_1.md          → [full path] (read-only)
  VALIDATION_LAYER.md  → [full path] (read-only)
  REPORT_2.md          → [full path]

REPORT_2.md contains:
  PART 1: LINKEDIN_PROFILE_SOP — strategy and execution system
  PART 2: LINKEDIN_CONTENT_IDEAS — format playbooks, hook library, starter set

⚠️  All [HYPOTHESIS] tags mark speculative recommendations.
    Any recommendation derived from a [LOW CONFIDENCE] source finding
    also carries [HYPOTHESIS] regardless of provenance.
    Validate via the Experiments Queue before committing.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Analysis complete. No further windows required.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
