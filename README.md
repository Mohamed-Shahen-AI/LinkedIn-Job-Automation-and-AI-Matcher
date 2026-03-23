# LinkedIn Jobs Auto-Apply Workflow

An n8n automation workflow that scrapes LinkedIn job postings, evaluates their relevance using AI, and automatically generates personalized cover letters — all on a scheduled basis.

---

## Overview

This workflow automates the job hunting process in 3 steps:

1. **Scrape & deduplicate** new LinkedIn job postings
2. **AI-score** each job for relevance against your resume
3. **Generate a personalized cover letter** for relevant jobs

---

## Workflow Steps

### Step 1 — Get and Remove Duplicated Jobs
Runs on a **weekly schedule** and:
- Reads existing jobs from a Google Sheet to build a list of already-seen postings
- Scrapes up to 100 new LinkedIn job postings via the **Apify LinkedIn Jobs Scraper** actor
- Filters out duplicates by comparing `Company + Job Title` against existing records
- Appends only **new jobs** to the Google Sheet with fields: Company, Job Title, URL, Apply URL, Description, Location, and Posted Date

### Step 2 — Check Job Relevance
Runs on a **per-minute schedule** and:
- Reads unprocessed job rows from the Google Sheet
- Fetches the candidate's resume from Google Docs
- Builds a prompt combining the job description and resume
- Sends it to an **AI Agent** (via OpenRouter) with a **Structured Output Parser** to score relevance
- If the job is **not relevant** → deletes the row from the sheet
- If the job **is relevant** → proceeds to Step 3, or marks it as reviewed
- Uses a **Telegram tool** to notify the candidate about relevant matches

### Step 3 — Prepare Cover Letter
Triggered by relevant jobs passing Step 2:
- Builds a tailored prompt from the job description and resume
- Sends it to a second **AI Agent** (via OpenRouter) to generate a professional cover letter (under 250 words)
- Creates a new **Google Doc** with the cover letter content
- Updates the Google Doc with final formatting
- Updates the Google Sheet row with the cover letter document link

---

## Tech Stack & Integrations

| Service | Purpose |
|---|---|
| **n8n** | Workflow orchestration |
| **Apify** | LinkedIn job scraping (`hKByXkMQaC5Qt9UMN`) |
| **Google Sheets** | Job tracking database |
| **Google Docs** | Cover letter storage |
| **OpenRouter** | LLM API gateway (AI relevance scoring + cover letter generation) |
| **Telegram** | Notifications for relevant job matches |

---

## Setup Instructions

### 1. Prerequisites
- A running **n8n** instance (self-hosted or cloud)
- Accounts for: Apify, Google (Sheets + Docs), OpenRouter, Telegram

### 2. Credentials to Configure
In n8n, set up the following credentials:

| Credential | Used By |
|---|---|
| `Google Sheets OAuth2` | Read/write job tracking sheet |
| `Google Docs OAuth2` | Create and update cover letter documents |
| `Apify API` | Run the LinkedIn scraper actor |
| `OpenRouter API` | Power both AI Agents |
| `Telegram API` | Send job match notifications |

### 3. Google Sheet Structure
Create a Google Sheet with the following columns:

| Column | Description |
|---|---|
| `Company` | Hiring company name |
| `Job Title` | Position title |
| `Url` | LinkedIn job listing URL |
| `Apply Url` | Direct application URL |
| `Description` | Full job description (HTML) |
| `Location` | Job location |
| `Post At` | Date the job was posted |
| `Cover Letter Path` | Google Doc link (filled automatically in Step 3) |

### 4. Import the Workflow
1. Download `LinkedIn_Jobs.json`
2. In n8n, go to **Workflows → Import from file**
3. Select the downloaded file
4. Update the Google Sheet document ID in the sheet nodes
5. Update the Telegram chat ID in the Telegram tool node
6. Configure all credentials as listed above
7. Activate each schedule trigger

### 5. Customize the Cover Letter Prompt
In the **Code in JavaScript1** node (Step 3), update the candidate name and focus areas:

```javascript
const prompt = `
Generate a concise, professional cover letter for [YOUR NAME]. 
Focus on his/her expertise in [YOUR KEY SKILLS].
Job Description: ${jobDesc}
Resume: ${resume}
Requirements:
- Under 250 words.
- Professional and enthusiastic tone.
- Highlight relevant project experience from the resume.
`.trim();
```

---


## Notes

- The **deduplication** logic uses a `Company-JobTitle` composite key, so the same role at a different company will not be filtered out.
- The Apify actor is configured to scrape **n8n-related jobs worldwide** by default. Update the URL in the **Run an Actor** node to target different roles or locations.
- Both AI Agents use **OpenRouter**, so you can swap the underlying model (GPT-4, Claude, Mistral, etc.) without changing the workflow structure.
- The workflow is set to **inactive** by default — activate each trigger manually after setup.

---

## License

MIT — free to use, modify, and share.
