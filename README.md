# Slack AI Agent: New Member Fit Analysis

A Slack bot that automatically analyzes new members joining a workspace or channel and posts a fit-score summary to a private channel, using Gemini for analysis and basic web research (company site + GitHub lookup) for context.

## What it does

When someone joins the Slack workspace (or a specific channel), the bot:

1. Pulls their Slack profile info (name, email, title, timezone)
2. Runs lightweight research, checks their email domain's company site and searches GitHub for a matching profile
3. Sends that context to Gemini, which scores the person's likely fit for the product (0–100) with supporting insights and engagement recommendations
4. Posts a formatted summary to a private Slack channel
5. Saves the full analysis to Postgres for later review

## Tech stack

- **Node.js** with `@slack/bolt` (Socket Mode) and `@slack/web-api`
- **LangChain** + **Gemini 2.5 Flash** (`@langchain/google-genai`) for the analysis
- **Express** for a health check endpoint and a local test endpoint
- **PostgreSQL** (via `pg`) for persisting analyses
- **Axios** for lightweight external lookups (company site, GitHub search)

## Deployment
The application is deployed on Railway.
**URL:** https://slack-ai-bot-production-f2ab.up.railway.app

> **Note:** This application is a Slack bot, so its primary functionality is accessible only through an authorized Slack workspace. Visiting the deployment URL in a browser will not display the bot interface.

## Setup

1. Clone the repo and install dependencies:
   ```bash
   npm install
   ```
2. Copy `.env.example` to `.env` and fill in your own credentials:
   ```bash
   cp .env.example .env
   ```
3. Start in development mode:
   ```bash
   npm run dev
   ```

The bot listens for `team_join` and `member_joined_channel` Slack events. In development mode, there's also a manual test endpoint:

```bash
curl -X POST http://localhost:3000/test/analyze-member \
  -H "Content-Type: application/json" \
  -d '{
    "memberInfo": {
      "name": "John Doe",
      "email": "john@techcorp.com",
      "title": "Senior Software Engineer at TechCorp"
    }
  }'
```

## Example output

> **New Member: John Doe**
> **Fit Score:** 78/100
> **Email:** john@techcorp.com
> **Title:** Senior Software Engineer at TechCorp
>
> **Insights:**
> - Strong technical background suggests high product affinity
> - Company profile indicates mid-size B2B SaaS, likely has budget authority
>
> **Recommendations:**
> - Invite to a technical demo
> - Share developer-focused onboarding content

## Notes

- Research is skipped for personal email domains (Gmail, Yahoo, etc.) to avoid noise.
- If the AI analysis fails (rate limits, API errors), the bot falls back to a neutral score and posts a "manual review recommended" note rather than failing silently.
- Free-tier Gemini rate limits apply — check [Google AI Studio](https://aistudio.google.com) if you hit `429` errors during testing.
