# 🔍 LinkedIn Clerk

> A two-part automated pipeline that scrapes LinkedIn profiles and converts raw data into a complete, ready-to-execute content strategy no browser, no login, no manual work.

---

## 📌 What It Does

LinkedIn Clerk is a scrape-to-strategy system built for founders, creators, and personal brand builders who want data-backed insight into what's actually working in their niche on LinkedIn.

You provide up to 10 profile URLs. The pipeline handles everything else scraping, processing, analyzing, and producing a full content playbook tailored to your niche and goal.

**Input:** Up to 10 LinkedIn profile URLs + your niche + your goal  
**Output:** Competitor intelligence report + a personalized LinkedIn content strategy with 20 hooks and 10 post ideas

---

## 🏗️ Architecture

The system is split into two independent parts:

```
[ You ] ──► [ v08 Scraper ] ──► [ RAW DATA CONTAINER ] ──► [ v05 Analysis ] ──► [ Strategy Reports ]
```

### Part 1 — Scraper (v08)

Calls two Apify actors via `bash` and `curl`:
- `harvestapi/linkedin-profile-posts` → scrapes all recent posts with full text and engagement data
- `harvestapi/linkedin-profile-scraper` → scrapes profile metadata (headline, follower count, about, skills, experience)

Merges both outputs into a single structured file: `LINKEDIN_RAW_DATA_CONTAINER.md`

**What the container includes:**
- Every post verbatim (text, format, hashtags, reactions, reposts, links, media)
- Per-post engagement scores
- Pre-computed statistics by content format (text-only, text+image, carousel, video)
- 70/20/10 tier labeling: `POPULAR` / `WORST` / `RECENT`
- Cross-profile dataset summary

> The scraper does **zero analysis**. It's a pure, clean data container — separation of concerns by design.

---

### Part 2 - Analysis Pipeline (v05)

Reads the raw data container and runs deep analysis across multiple windows.

Produces three output documents:

#### `REPORT_1.md` - Competitive Intelligence
- Engagement outlier analysis (top and bottom posts, with reasoning)
- Hook taxonomy every opening sentence classified by type and ranked by performance
- Narrative structure breakdown (problem/solution, list, story arc, contrarian take, etc.)
- Timing pattern analysis
- Profile intelligence per competitor (positioning, tone, audience targeting, archetype)
- Voice and style mapping
- Niche crossover opportunities
- Structured experiment queue findings that contradict conventional LinkedIn wisdom, formatted as A/B tests

#### `VALIDATION_LAYER.md` - Cross-Reference Check
- Flags where competitors' actual behavior agrees or contradicts general LinkedIn best-practice knowledge
- Marks findings as data-backed vs. speculative

#### `REPORT_2.md` - Your Content Strategy
- Content format strategy with engagement benchmarks
- Posting cadence recommendation
- Hook and CTA system
- Niche positioning recommendation based on competitor gaps
- **20 ready-to-use hook opening sentences** tailored to your niche
- **10 fully specified post ideas** — each with format, hook type, opening sentence, narrative structure, and CTA

---

## 🛠️ Tech Stack

| Layer | Tool |
|---|---|
| Profile scraping | Apify (`harvestapi` actors) |
| HTTP requests | `bash`, `curl` |
| AI agent execution | Claude / Claude Code |
| Output format | Structured Markdown |

No browser automation. No Selenium. No login required.

---

## 🚀 How to Run

### Prerequisites
- Apify API key ([get one here](https://apify.com))
- Claude or any LLM that supports agentic execution
- Basic bash environment

### Step 1 - Run the Scraper (v08)

Provide the following inputs when prompted:
```
Profile URLs:     Up to 10 LinkedIn profile URLs
Apify API Key:    Your Apify API key
Lookback period:  How far back to scrape (e.g., 90 days)
Your niche:       e.g., "AI for small business"
Your goal:        e.g., "Get consulting clients"
```

Output: `LINKEDIN_RAW_DATA_CONTAINER.md`

### Step 2 - Run the Analysis (v05)

Feed the raw data container into the analysis pipeline with the same niche and goal inputs.

Output:
```
REPORT_1.md           ← Competitor intelligence
VALIDATION_LAYER.md   ← Cross-reference check
REPORT_2.md           ← Your content strategy
```

---

## 📂 File Structure

```
linkedin-clerk/
├── v08-lnkdn-scrape.md          # Scraper prompt / pipeline instructions
├── v05-lnkdn-analysis.md        # Analysis pipeline instructions
├── LINKEDIN_RAW_DATA_CONTAINER.md   # Generated raw scraped data
├── REPORT_1.md                  # Generated competitor intelligence
├── VALIDATION_LAYER.md          # Generated validation layer
├── REPORT_2.md                  # Generated your content strategy
└── README.md
```

---

## 💡 Use Cases

- Building a LinkedIn personal brand from scratch with zero guesswork
- Identifying content format gaps your competitors aren't filling
- Generating a starter post bank with proven hook structures
- Reverse-engineering what makes top creators in your niche work

---

## ⚠️ Notes

- This tool is for **research and personal brand strategy** use responsibly and in line with LinkedIn's and Apify's terms of service.
- The scraper is a pure data collector it does not interpret or editorialize. All analysis happens in Part 2.
- Results are only as good as the profiles you feed in. Choose competitors who are actively posting and have meaningful engagement data.

---

## 🌐 Connect with Me

I am a Data Analytics professional passionate about turning complex data into visual stories. If you have questions about this project or want to discuss Data Analytics, Business Intelligence, or Automation—let's connect!

<p align="left">
  <a href="https://www.linkedin.com/in/adityasrinish/" target="_blank">
    <img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white" alt="LinkedIn" />
  </a>
  <a href="https://github.com/AdityaSrinish" target="_blank">
    <img src="https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white" alt="GitHub" />
  </a>
  <a href="mailto:adityasrinish3107@gmail.com">
    <img src="https://img.shields.io/badge/Gmail-D14836?style=for-the-badge&logo=gmail&logoColor=white" alt="Email" />
  </a>
</p>

---

### 📁 My Portfolio
> **[Visit My Digital Portfolio 🚀](https://www.datascienceportfol.io/adityasrinish)**
> *Check out my other projects, including SQL challenges, N8n automation workflows, and my visual-first approach to data.*

---

<p align="center">
  <i>"Aditya Srinish"</i><br>
  <i>"Building AI systems for real business outcomes."</i><br>
  📍 Vijayawada, India
</p>
