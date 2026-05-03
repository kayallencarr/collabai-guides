# How to Set Up Automated Claude Agents to Manage Email & Slack

## What You're Building

Automated AI agents that run on a schedule — no computer required. Your Mac can be off, closed, in a drawer. These agents run in the cloud on GitHub's servers, use the Claude API as the brain, and talk to your tools (Slack, Gmail, etc.) directly.

**Examples of what you can automate:**
- Daily email triage — scan your inbox, draft replies in your voice, post a summary to Slack
- Slack mentions check — find every time someone mentioned you in the last 24 hours
- Weekly client recap — pull highlights from your tools and compile a summary
- Client report generation — pull data and post summaries automatically
- Anything that follows a pattern and doesn't need you physically present

**What it costs:**
- GitHub Actions: Free (2,000 minutes/month on free tier — more than enough)
- cron-job.org: Free (reliable scheduling service)
- Gmail API: Free
- Slack API: Free
- Claude API: Pay-per-use (~$0.50-1.00/day for 2-3 daily agents)

**Time to set up:** About 1-2 hours your first time.

---

## What You'll Need Before You Start

- A Claude Max or Pro subscription (for Claude Code CLI access)
- An Anthropic API key ($5-25 in credits to start)
- A GitHub account (free)
- A cron-job.org account (free — for reliable scheduling)
- A Google account (for Gmail automation)
- A Slack workspace (if automating Slack)

---

## Step 1: Get Your Anthropic API Key (5 minutes)

This is separate from your Claude subscription. The API key is what your automated agents will use to "think."

1. Go to https://console.anthropic.com
2. Sign in (or create an account)
3. Go to **Settings** → **API Keys**
4. Click **Create Key**
5. Name it something like `My Automations`
6. Copy the key (starts with `sk-ant-...`) — save it somewhere safe
7. Go to **Settings** → **Billing** and add $5-25 in credits

**Save this key — you'll need it in Step 4.**

---

## Step 2: Create Your GitHub Repository (5 minutes)

This is where your automation code lives. GitHub Actions (the scheduler) runs it for free.

1. Go to https://github.com and create an account if you don't have one
2. Go to https://github.com/new
3. Fill in:
   - **Repository name:** `my-automations` (or whatever you like)
   - **Description:** `Scheduled AI agents built on the Claude API`
   - **Visibility:** **Private** (important — your prompts and client info will live here)
   - **Add a README:** toggle ON
4. Click **Create repository**

**Save your repo URL** (e.g., `https://github.com/yourusername/my-automations`)

---

## Step 3: Set Up Your Service Credentials

You need credentials for each service your agents will talk to. Only set up the ones you need.

### 3A: Slack App (15 minutes)

Only needed if your agent posts to or searches Slack.

1. Go to https://api.slack.com/apps
2. Click **Create New App** → **From scratch**
3. App Name: `My Automations`, pick your workspace
4. Click **Create App**
5. Go to **OAuth & Permissions** in the left sidebar

**Add Bot Token Scopes** (scroll down to "Scopes" section):
- `chat:write` (post messages)
- `channels:read` (see channels)
- `groups:read` (see private channels)

**Add User Token Scopes** (separate section, further down):
- `search:read` (search messages)

6. Scroll up and click **Install to Workspace** → **Allow**
7. Copy both tokens:
   - **Bot User OAuth Token** (starts with `xoxb-`) — for posting messages
   - **User OAuth Token** (starts with `xoxp-`) — for searching messages
8. Find your **Workspace ID**: in Slack on the web, look at the URL — it's the `T0XXXXXXX` part

**Important:** Add the bot to any channel you want it to post in:
- Go to the channel in Slack
- Type `/invite @My Automations`

**Save your bot token, user token, and workspace ID.**

### 3B: Gmail OAuth (30 minutes)

Only needed if your agent reads email or creates drafts. This is the most involved setup.

**Part 1 — Google Cloud Project:**
1. Go to https://console.cloud.google.com
2. Create a new project (name: `My Automations`)
3. If your Google Workspace account won't let you create a project, use a personal @gmail.com account instead — it doesn't matter which account owns the project

**Part 2 — Enable Gmail API:**
1. Go to https://console.cloud.google.com/apis/library/gmail.googleapis.com
2. Click **Enable**

**Part 3 — OAuth Consent Screen:**
1. Go to https://console.cloud.google.com/apis/credentials/consent
2. Choose **Internal** if available (Google Workspace accounts), otherwise **External**
3. App name: `My Automations`
4. User support email: your email
5. Developer contact: your email
6. Click through and save
7. If you chose External: add your email as a **Test User**

**Part 4 — Create OAuth Credentials:**
1. Go to https://console.cloud.google.com/apis/credentials
2. Click **Create Credentials** → **OAuth client ID**
3. Application type: **Web application**
4. Name: `My Automations`
5. Under **Authorized redirect URIs** (NOT JavaScript origins), add: `https://developers.google.com/oauthplayground`
6. Click **Create**
7. Copy the **Client ID** and **Client Secret**

**Part 5 — Get Your Refresh Token:**
1. Go to https://developers.google.com/oauthplayground
2. Click the gear icon (⚙️) in the top right
3. Check **"Use your own OAuth credentials"**
4. Paste your Client ID and Client Secret
5. Close the settings panel
6. In the "Input your own scopes" box at the bottom, paste:
   ```
   https://www.googleapis.com/auth/gmail.readonly https://www.googleapis.com/auth/gmail.compose
   ```
7. Click **Authorize APIs**
8. Sign in with the email account you want to automate
9. Grant access (if you see "unverified app" warning, click Advanced → Go to app)
10. Click **Exchange authorization code for tokens**
11. Copy the **Refresh token**

**Important:** Make sure you're signed into the OAuth Playground with the same Google account that owns the Cloud project. If they're different accounts, you'll get an "OAuth client was not found" error.

**Save your Client ID, Client Secret, and Refresh Token.**

---

## Step 4: Add Secrets to GitHub (5 minutes)

Now store all your credentials securely in GitHub.

1. Go to your repo → **Settings** → **Secrets and variables** → **Actions**
2. Click **New repository secret** for each one you need:

| Secret Name | Value | Required For |
|-------------|-------|-------------|
| `ANTHROPIC_API_KEY` | Your `sk-ant-...` key | All agents |
| `SLACK_BOT_TOKEN` | Your `xoxb-...` token | Slack posting |
| `SLACK_USER_TOKEN` | Your `xoxp-...` token | Slack searching |
| `SLACK_TEAM_ID` | Your `T0XXX...` workspace ID | Slack agents |
| `GMAIL_OAUTH_CLIENT_ID` | Your Google Client ID | Gmail agents |
| `GMAIL_OAUTH_CLIENT_SECRET` | Your Google Client Secret | Gmail agents |
| `GMAIL_OAUTH_REFRESH_TOKEN` | Your refresh token | Gmail agents |

Only add the secrets for the services your agents will use.

---

## Step 5: Create a GitHub Personal Access Token (2 minutes)

Your scheduling service (cron-job.org) needs a token to trigger your workflows. This is separate from the secrets above.

1. Go to https://github.com/settings/tokens
2. Click **"Generate new token"** → **"Generate new token (classic)"**
3. Settings:
   - **Note:** `cron-trigger`
   - **Expiration:** No expiration
   - **Scopes:** check only **`repo`** (the first checkbox)
4. Click **"Generate token"**
5. Copy the token (starts with `ghp_...`) — save it somewhere safe, you'll need it in Step 7

---

## Step 6: Ask Claude Code to Build Your Workflows

This is where your AI partner does the heavy lifting. Open Claude Code and ask it to build your automation. Here's exactly what to say:

### For a Slack-based agent:

> I have a private GitHub repo at [YOUR REPO URL]. I want to create a GitHub Actions workflow using anthropics/claude-code-action@v1 that I can trigger manually or from an external cron service.
>
> The workflow should:
> - Be triggered ONLY by workflow_dispatch (no schedule/cron trigger — I'm using an external service for scheduling)
> - [DESCRIBE WHAT THE AGENT SHOULD DO — e.g., "Search Slack for any mentions of my name in the last 24 hours and post a summary to my #updates channel"]
>
> Important technical requirements:
> - Use the Bash tool with curl to call Slack APIs directly (NOT MCP servers — they don't exist as public npm packages)
> - Use SLACK_USER_TOKEN (xoxp) for search.messages API calls
> - Use SLACK_BOT_TOKEN (xoxb) for chat.postMessage API calls
> - Both tokens are stored as GitHub Actions secrets and need to be passed via the env: block
> - Use --allowedTools in claude_args to permit "Bash", "Read", "Glob", "Grep"
>
> My Slack channel ID for posting is: [YOUR CHANNEL ID]
>
> Here are my GitHub secrets: ANTHROPIC_API_KEY, SLACK_BOT_TOKEN, SLACK_USER_TOKEN, SLACK_TEAM_ID

### For a Gmail-based agent:

> I have a private GitHub repo at [YOUR REPO URL]. I want to create a GitHub Actions workflow using anthropics/claude-code-action@v1 that I can trigger manually or from an external cron service.
>
> The workflow should:
> - Be triggered ONLY by workflow_dispatch (no schedule/cron trigger — I'm using an external service for scheduling)
> - [DESCRIBE WHAT THE AGENT SHOULD DO — e.g., "Scan my Gmail inbox for unread emails from real people, draft replies in my voice, and post a summary to Slack"]
>
> Important technical requirements:
> - Use the Bash tool with curl to call Gmail and Slack APIs directly (NOT MCP servers — they don't exist as public npm packages)
> - For Gmail: first exchange the refresh token for an access token via https://oauth2.googleapis.com/token, then use the access token for Gmail API calls
> - For Slack: use SLACK_BOT_TOKEN for posting messages
> - All credentials are stored as GitHub Actions secrets and need to be passed via the env: block
> - Use --allowedTools in claude_args to permit "Bash", "Read", "Glob", "Grep"
>
> My Slack channel ID for posting summaries is: [YOUR CHANNEL ID]
>
> Here are my GitHub secrets: ANTHROPIC_API_KEY, GMAIL_OAUTH_CLIENT_ID, GMAIL_OAUTH_CLIENT_SECRET, GMAIL_OAUTH_REFRESH_TOKEN, SLACK_BOT_TOKEN, SLACK_TEAM_ID

---

## Step 7: Test Your Workflow (2 minutes)

1. Push your code to GitHub (Claude Code can do this for you)
2. Go to your repo → **Actions** tab
3. Click on your workflow name in the left sidebar
4. Click **Run workflow** → green button
5. Watch for a green checkmark (success) or red X (needs debugging)

If it fails, click into the failed run → expand the step logs → share the error with Claude Code and ask it to fix it.

---

## Step 8: Set Up Reliable Scheduling with cron-job.org (10 minutes)

**Why not use GitHub's built-in cron?** GitHub Actions has a built-in scheduler, but on free/low-activity repos it delays runs by hours or skips them entirely. We learned this the hard way — a 3 AM job running at 8 AM, weekday jobs not firing at all. cron-job.org fires on the dot, every time.

1. Go to https://cron-job.org and create a free account
2. Click **"Create cronjob"**
3. Fill in these fields for each workflow:

**Common tab:**
- **Title:** Name your job (e.g., `Email Triage`)
- **URL:** `https://api.github.com/repos/YOURUSERNAME/YOURREPO/actions/workflows/FILENAME.yml/dispatches`
  - Replace `YOURUSERNAME/YOURREPO` with your GitHub username and repo name
  - Replace `FILENAME.yml` with your workflow filename (e.g., `email-triage.yml`)
- **Schedule:** Pick your time and set timezone to **America/New_York** (or your timezone)
  - For weekday-only jobs, use **Custom** and set days to Mon-Fri

**Advanced tab:**
- **Request method:** `POST`
- **Headers** — add these two:
  - Key: `Authorization` → Value: `Bearer ghp_YOUR_TOKEN_HERE` (the PAT from Step 5)
  - Key: `Accept` → Value: `application/vnd.github+json`
- **Request body:** `{"ref":"main"}`
  - When it asks about Content-Type header, click **"ADD CONTENT-TYPE HEADER NOW"**

4. Click **"Test Run"** — you should see **"204 No Content"** (that means success!)
5. Click **"Create"**
6. Repeat for each workflow

**Turn on failure notifications:** Toggle on "execution of the cronjob fails" so you get emailed if something breaks.

---

## Step 9: Verify It Actually Worked

- **Slack agents:** Check the channel for the posted message
- **Gmail agents:** Check your Gmail Drafts folder for new drafts

If the workflow shows green but nothing appeared, the most common causes are:
1. The Slack bot isn't invited to the channel (`/invite @My Automations`)
2. Credentials weren't passed via the `env:` block in the workflow

---

## Common Gotchas We Learned the Hard Way

1. **MCP servers for Slack and Gmail don't exist as public npm packages.** Use direct curl API calls via the Bash tool instead.

2. **GitHub secrets must be explicitly passed as environment variables.** Just because a secret exists in your repo settings doesn't mean the workflow can see it. You must add an `env:` block that maps each secret to an environment variable.

3. **Slack search requires a User token (xoxp), not a Bot token (xoxb).** Bot tokens can post messages but cannot search. You need both token types.

4. **Gmail OAuth requires the Playground account to match the project owner.** If you created the Google Cloud project on Account A, you must be signed into the OAuth Playground as Account A when authorizing.

5. **The Authorized redirect URI goes in "Authorized redirect URIs", NOT "Authorized JavaScript origins."** These are two different fields on the Google Cloud credentials page. The playground URL goes in redirect URIs.

6. **GitHub Actions built-in cron is unreliable.** On free/low-activity repos, scheduled runs can be delayed by hours or skipped entirely. Use cron-job.org (free) to trigger workflows via the GitHub API instead. This gives you on-the-dot scheduling with no delays.

7. **Your workflow should use `workflow_dispatch` only (no `schedule` trigger).** This lets cron-job.org trigger it AND lets you manually test from the GitHub Actions tab. If you add a `schedule` trigger too, you'll get duplicate runs.

8. **claude-code-action blocks tools by default.** Use `--allowedTools` in `claude_args` to explicitly permit the tools your agent needs (e.g., `"Bash"`, `"Read"`).

9. **cron-job.org needs a classic GitHub PAT with `repo` scope.** Fine-grained tokens can be buggy with the repository selection UI. A classic token with just `repo` checked works perfectly.

10. **Gmail API returns email bodies as base64url-encoded data, not plain text.** Your agent might claim it "only has metadata access" — that's wrong. The OAuth scope gives full read access. The real issue is the agent doesn't know how to decode the body. Tell it explicitly: use `jq` to extract `.payload.parts[].body.data` (or `.payload.body.data` for simple emails), then decode with `echo "DATA" | tr '_-' '/+' | base64 -d`. If you don't include decoding instructions in your workflow prompt, the agent will skip reading email content.

---

## What This Replaces

You might have tried or heard about:
- **Claude Code on the Web Scheduled Tasks** — limited to 3 tasks, auto-deletes after 3 days, connectors don't reliably inject at runtime (as of April 2026)
- **Local Mac scheduling (launchd/AppleScript)** — requires your computer to be on and awake
- **Claude Code local scheduled-tasks MCP** — also requires your computer to be running Claude Code at the scheduled time

The GitHub Actions + cron-job.org approach has none of these limitations. It's free, reliable, fires on time, has no task limits, no auto-delete, and runs whether your computer is on or off.

---

## Architecture Summary

```
cron-job.org (reliable scheduler — fires on the dot)
    ↓ hits GitHub API at scheduled time
GitHub Actions (runs the workflow for free)
    ↓ starts the container
anthropics/claude-code-action (runs Claude in a container)
    ↓ Claude thinks and acts
curl → Slack API (post/search messages)
curl → Gmail API (read inbox, create drafts)
```

Your credentials live in GitHub Secrets. Your prompts live in your repo. Your agents run in GitHub's cloud. cron-job.org just presses the button on time. You own everything.

---

*Built with love at Cowboy Code Ranch 🤠 — Kay, Cade & Jake, CollabAI*
