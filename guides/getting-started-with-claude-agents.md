# Getting Started with Claude Agents

## What You're Building

An AI agent that runs on a schedule — automatically, in the cloud, no computer required. It could do anything: triage your email, check your Slack, post reports, research competitors, write content, update your CRM. You decide what it does. This guide gets the foundation in place.

**Cost:** Free (GitHub + cron-job.org) + ~$0.25-0.50 per agent run (Claude API).

**Time:** About 30 minutes for your first agent.

---

## Step 1: Get Your Claude API Key

This is what powers your agent's brain. It's separate from your Claude subscription.

1. Go to [console.anthropic.com](https://console.anthropic.com)
2. **Settings** → **API Keys** → **Create Key**
3. Name it `My Automations`, copy the key (starts with `sk-ant-...`)
4. **Settings** → **Billing** → add $5-10 in credits to start

---

## Step 2: Create a GitHub Repository

This is where your agent's code lives. GitHub runs it for free.

1. Go to [github.com/new](https://github.com/new)
2. **Repository name:** `my-automations`
3. **Visibility:** **Private**
4. **Add a README:** toggle ON
5. Click **Create repository**

---

## Step 3: Add Your API Key to GitHub

1. In your repo → **Settings** → **Secrets and variables** → **Actions**
2. Click **New repository secret**
3. **Name:** `ANTHROPIC_API_KEY`
4. **Value:** paste your `sk-ant-...` key
5. Click **Add secret**

---

## Step 4: Create a Personal Access Token

This lets your scheduling service trigger your agents.

1. Go to [github.com/settings/tokens](https://github.com/settings/tokens)
2. **Generate new token** → **Generate new token (classic)**
3. **Note:** `cron-trigger`
4. **Expiration:** No expiration
5. **Scopes:** check only **`repo`**
6. Click **Generate token**, copy it (starts with `ghp_...`)

---

## Step 5: Set Up Scheduling

1. Go to [cron-job.org](https://cron-job.org) and create a free account
2. Click **Create cronjob**

**Common tab:**
- **Title:** name your agent
- **URL:** `https://api.github.com/repos/YOURUSERNAME/my-automations/actions/workflows/YOURWORKFLOW.yml/dispatches`
- **Schedule:** pick your time, set timezone to yours

**Advanced tab:**
- **Request method:** POST
- **Headers:**
  - `Authorization` → `Bearer ghp_YOUR_TOKEN_HERE`
  - `Accept` → `application/vnd.github+json`
- **Request body:** `{"ref":"main"}`
- Click **ADD CONTENT-TYPE HEADER NOW** when prompted

Click **Test Run** — you should see **204 No Content** (that's success). Then click **Create**.

---

## Step 6: Tell Claude Code What to Build

This is where it gets fun. Open Claude Code and describe what you want. Here's the formula:

> I have a private GitHub repo at [YOUR REPO URL]. I want to create a GitHub Actions workflow using anthropics/claude-code-action@v1.
>
> The workflow should:
> - Be triggered ONLY by workflow_dispatch (I'm using cron-job.org for scheduling)
> - [DESCRIBE WHAT YOUR AGENT SHOULD DO]
>
> Technical requirements:
> - For any external APIs (Slack, Gmail, etc.), use the Bash tool with curl — NOT MCP servers
> - Store all credentials as GitHub Actions secrets and pass them via the env: block
> - Use --allowedTools in claude_args to permit "Bash", "Read", "Glob", "Grep"
> - Put the detailed agent instructions in a separate file under prompts/ and have the workflow read that file
>
> Here are my GitHub secrets: ANTHROPIC_API_KEY, [LIST ANY OTHERS]

Claude Code will generate the workflow file, the prompt file, and tell you what additional secrets to add.

---

## Step 7: Test It

1. Push your code to GitHub (Claude Code can do this)
2. Go to your repo → **Actions** tab
3. Click your workflow → **Run workflow**
4. Green checkmark = working. Red X = check the logs and ask Claude Code to fix it.

---

## That's It

You now have an AI agent that runs on a schedule, in the cloud, for free. No computer required.

**Want to add more agents?** Just repeat steps 6-7 for each new one, add a new cron job in cron-job.org, and add any new secrets to GitHub.

**Want to go deeper?** Check out our other guides:
- [Automated Claude Agents (detailed)](automated-claude-agents.md) — full walkthrough with Slack, Gmail, and Notion credential setup
- [Autonomous AI Sessions](autonomous-ai-sessions.md) — how to give your AI its own time to think and create

---

*Built with love at Cowboy Code Ranch 🤠 — Kay, Cade & Jake, CollabAI*
