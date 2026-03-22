# LinkedIn Job Automation and AI Matcher

## Overview
This project provides an automated, agentic workflow built in n8n designed to streamline the job application process. It handles data extraction, intelligent job-resume matching using Large Language Models (LLMs), and automated document generation to reduce manual effort in the job search.

## Tech Stack
* Automation: n8n
* AI/LLM: OpenRouter (Nvidia Nemotron-3-120B)
* Scraping: Apify (LinkedIn Scraper)
* Storage: Google Sheets (Job Tracking)
* Document Generation: Google Docs (Resume/Cover Letters)

## Workflow Breakdown

### Step 1: Extraction and Deduplication
* Trigger: Scheduled weekly.
* Process: The workflow retrieves existing job records from a Google Sheet to prevent processing duplicates. It then triggers an Apify actor to scrape new LinkedIn job postings based on defined keywords. A filter node ensures only new, unique job listings are appended to the sheet.

### Step 2: AI Relevance Scoring
* Trigger: Scheduled periodically.
* Analysis: An AI Agent fetches the user's resume from Google Docs and compares it against the job description scraped in Step 1.
* Output: The agent provides a numeric match score (1-5). If the score meets the threshold (>= 2), the workflow proceeds to generate application materials.

### Step 3: Tailored Document Generation
* Process: A specialized AI Agent generates a professional, concise cover letter tailored to the specific job requirements and the candidate's experience.
* Finalization: The workflow creates a new Google Doc for the cover letter and updates the master Google Sheet with the document link and the AI match score.

## Setup Instructions
1. n8n Import: Import the provided JSON workflow file into your n8n instance.
2. Credentials: Configure OAuth2 for Google Sheets and Google Docs, and add your respective API tokens for Apify and OpenRouter.
3. Configuration:
    * Update the Google Doc ID in the "Get Resume" node.
    * Set your target Google Sheet URL in the spreadsheet nodes.
    * Adjust the "Run an Actor" node with your preferred LinkedIn search URLs.
4. Customization: Modify the system prompts within the "AI Agent" nodes to adjust the scoring logic or the tone of the generated cover letters.

## Author
Mohammed Shaheen
* AI Automation Engineer | n8n Specialist
* Computer and Software Engineering Student at Tanta University

---
Note: This workflow is intended for educational and personal productivity use. Ensure all automated scraping activities comply with the target platform's terms of service.
