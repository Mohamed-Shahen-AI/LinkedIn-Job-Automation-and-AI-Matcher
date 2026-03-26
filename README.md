# LinkedIn Jobs Auto-Apply Workflow

An **n8n** automation workflow that scrapes LinkedIn job postings, evaluates their relevance using AI, and automatically generates personalized cover letters — all on a scheduled basis.

## Overview
This workflow automates the job hunting process in 3 steps:
1. **Scrape & deduplicate** new LinkedIn job postings.
2. **AI-score** each job for relevance against your resume.
3. **Generate a personalized cover letter** for relevant jobs.

![LinkedIn Jobs Workflow](LinkedIn%20Jobs%20Image.png)

## Workflow Steps

### Step 1 — Get and Remove Duplicated Jobs
Runs on a weekly schedule and:
*   Reads existing jobs from a **Google Sheet** to build a list of already-seen postings.
*   Scrapes up to 100 new LinkedIn job postings via the **Apify LinkedIn Jobs Scraper** actor.
*   Filters out duplicates by comparing **Company + Job Title** against existing records.
*   Appends only new jobs to the Google Sheet with fields: Company, Job Title, URL, Apply URL, Description, Location, and Posted Date.

### Step 2 — Check Job Relevance
Runs on a per-minute schedule and:
*   Reads unprocessed job rows from the Google Sheet.
*   Fetches the candidate's resume from **Google Docs**.
*   Builds a prompt combining the job description and resume.
*   Sends it to an **AI Agent** (via OpenRouter) with a Structured Output Parser to score relevance.
*   **If not relevant**: Deletes the row from the sheet.
*   **If relevant**: Notifies the candidate via **Telegram** and proceeds to Step 3.

### Step 3 — Prepare Cover Letter
Triggered by relevant jobs passing Step 2:
*   Builds a tailored prompt from the job description and resume.
*   Sends it to a second **AI Agent** (via OpenRouter) to generate a professional cover letter (under 250 words).
*   Creates and formats a new **Google Doc** with the cover letter content.
*   Updates the Google Sheet row with the specific document link.

## Tech Stack & Integrations

| Service | Purpose |
| :--- | :--- |
| **n8n** | Workflow orchestration |
| **Apify** | LinkedIn job scraping (hKByXkMQaC5Qt9UMN) |
| **Google Sheets** | Job tracking database |
| **Google Docs** | Resume storage and Cover letter generation |
| **OpenRouter** | LLM API gateway (AI relevance scoring + cover letter generation) |
| **Telegram** | Real-time notifications for relevant job matches |

## Setup Instructions

### 1. Prerequisites
*   A running n8n instance (self-hosted or cloud).
*   Accounts for: Apify, Google (Sheets + Docs), OpenRouter, Telegram.

### 2. Credentials to Configure
In n8n, set up the following credentials:
*   **Google Sheets & Docs OAuth2**: For database and document management.
*   **Apify API**: To run the LinkedIn scraper actor.
*   **OpenRouter API**: To power both AI Agents.
*   **Telegram API**: To send job match notifications.

### 3. Google Sheet Structure
Create a Google Sheet with the following columns:
`Company`, `Job Title`, `Url`, `Apply Url`, `Description`, `Location`, `Post At`, `Cover Letter Path`.

### 4. Customize the Cover Letter Prompt
In the **Code in JavaScript** node (Step 3), update your name and focus areas:

```javascript
const prompt = `Generate a concise, professional cover letter for [YOUR NAME]. 
Focus on expertise in [YOUR KEY SKILLS].
Job Description: ${jobDesc}
Resume: ${resume}
Requirements:
- Under 250 words.
- Professional and enthusiastic tone.
- Highlight relevant projects from the resume.`.trim();
