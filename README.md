# Social by InstantDM — AI Agent Mode

> Give your AI agent the ability to **post, schedule, and track social media** across **Instagram,
> Facebook, X/Twitter, LinkedIn, and TikTok** from a single command — built on the
> [Social by InstantDM](https://socialbyidm.com) API and hosted MCP server.

**Agent Mode** turns any MCP-capable assistant — Claude, Claude Code, Cursor, Windsurf, ChatGPT,
Grok, OpenClaw, Hermes — into an autonomous social media manager. No SDK to learn, no servers to
run: connect with one URL and a workspace API key, then just ask in plain language.

```text
"Schedule this carousel to Instagram and LinkedIn for tomorrow at 9am."
"Draft an X thread from these five bullets and queue it for Monday."
"Cross-post my latest reel to Facebook and TikTok, then tell me when it's live."
"What was my best-performing post this month, and on which platform?"
```

This repository is a **skill package** — a single [`SKILL.md`](skills/social-by-idm/SKILL.md) that
teaches an agent the full, safe posting workflow, plus a Claude Code plugin manifest. No secrets are
baked in; every user supplies their own scoped, instantly-revocable key.

---

## Contents

- [Quick start](#quick-start)
- [Install](#install)
- [Connect your agent — step by step](#connect-your-agent--step-by-step)
  - [Claude Code](#1-connect-claude-code)
  - [Claude Desktop, Cursor & Windsurf](#2-connect-claude-desktop-cursor--windsurf)
  - [Claude.ai, ChatGPT & Grok (web)](#3-connect-claudeai-chatgpt--grok-web)
  - [OpenClaw & ClawHub](#4-connect-openclaw--clawhub)
  - [Your own code (REST API)](#5-connect-your-own-code-rest-api)
- [Get your API key](#get-your-api-key)
- [Supported platforms & post types](#supported-platforms--post-types)
- [FAQ](#faq)
- [Docs & links](#docs--links)
- [License](#license)

---

## Quick start

1. **Create a workspace** at [social-app.instantdm.com](https://social-app.instantdm.com) and connect your social accounts.
2. **Mint a scoped API key** — Developer → API keys → New key.
3. **Connect your agent** using one of the methods below.
4. **Ask it to post.** That's it.

---

## Install

| Client | One-liner |
|--------|-----------|
| Agent Skill (any `skills`-compatible agent) | `npx skills add social-by-idm/agent-mode` |
| Claude Code plugin | `/plugin marketplace add social-by-idm/agent-mode` then `/plugin install social-by-idm` |
| OpenClaw / ClawHub | `openclaw skills install @social-by-idm/social-media-manager` |
| Cursor | `git clone https://github.com/social-by-idm/agent-mode .cursor/skills/social-by-idm` |
| Manual | Copy [`skills/social-by-idm/`](skills/social-by-idm/) into your agent's skills directory |

> Prefer a direct connection with no skill file? Use the [hosted MCP server](#connect-your-agent--step-by-step) — most agents need only a URL and key.

---

## Connect your agent — step by step

Pick your agent. Each path uses the same hosted MCP server (`https://social-api.instantdm.com/mcp`)
and your workspace API key — the only difference is *how* the client sends the key.

### 1. Connect Claude Code

Anthropic's terminal agent registers the MCP server in one command, sending your key as a header.

1. In the dashboard, open **Developer → API keys** and mint a workspace-scoped key.
2. Run this in your terminal (nothing installs locally — it points Claude Code at the hosted server):
   ```bash
   claude mcp add --transport http social-by-idm \
     https://social-api.instantdm.com/mcp \
     --header "X-Api-Key: sk_live_xxx"
   ```
3. Verify by asking Claude Code: **"List my social accounts."**

### 2. Connect Claude Desktop, Cursor & Windsurf

Any desktop MCP client — paste one block into the MCP config and the agent discovers all 13 tools.

1. Mint a scoped API key in the dashboard.
2. Open your client's MCP config file and add the server:
   ```json
   {
     "mcpServers": {
       "social-by-idm": {
         "type": "streamable-http",
         "url": "https://social-api.instantdm.com/mcp",
         "headers": { "X-Api-Key": "sk_live_xxx" }
       }
     }
   }
   ```
3. Restart the client — the Social by InstantDM tools appear automatically.

### 3. Connect Claude.ai, ChatGPT & Grok (web)

Browser agents can't send custom headers, so put the key **in the URL** via the connector dialog.

1. Mint a **dedicated, scoped** key (it goes in a URL — keep it least-privilege).
2. In the agent, choose **"Add custom connector"** (or custom MCP server) and paste:
   ```text
   https://social-api.instantdm.com/mcp?key=sk_live_xxx
   ```
3. **Leave the OAuth / Client ID fields blank** and save. Revoke the key any time to cut access instantly.

### 4. Connect OpenClaw & ClawHub

Install the published skill and your OpenClaw agent learns the full posting workflow.

1. Install the skill:
   ```bash
   openclaw skills install @social-by-idm/social-media-manager
   ```
2. Store your key in the workspace `.env`:
   ```bash
   SOCIAL_BY_IDM_API_KEY=sk_live_xxx
   ```
3. Ask your agent to draft, schedule, or publish a post.

### 5. Connect your own code (REST API)

For scripts, CI and custom backends — one signed endpoint, the same publish pipeline.

1. Mint a scoped key and send it as the `X-Api-Key` header.
2. Create a post:
   ```bash
   curl -X POST https://social-api.instantdm.com/v1/posts \
     -H "X-Api-Key: sk_live_xxx" \
     -H "Content-Type: application/json" \
     -d '{"accountIds":["acc_x"],"content":"shipping.","publishNow":true}'
   ```
3. Poll `GET /v1/posts/{id}/status` to confirm each platform published.

Full endpoint reference: [socialbyidm.com/docs](https://socialbyidm.com/docs).

---

## Get your API key

1. Sign up at [social-app.instantdm.com](https://social-app.instantdm.com) and connect your accounts.
2. Go to **Developer → API keys → New key** and grant only the scopes you need
   (`accounts:read`, `posts:read`, `posts:write`, `media:write`, `analytics:read`).
3. Store it in your workspace `.env` and treat it like a password:
   ```bash
   SOCIAL_BY_IDM_API_KEY=sk_live_xxx
   ```

Keys are **workspace-scoped and revocable** — delete one from the dashboard and it stops working on
the very next request (every call re-checks the key, so there's no cache or delay).

---

## Supported platforms & post types

| Platform | Posts your agent can create |
|----------|------------------------------|
| Instagram | image, carousel, reel, story |
| Facebook | text, image, carousel, video |
| X / Twitter | text, image, thread |
| LinkedIn | text, image, article, PDF document |
| TikTok | video, photo |

Schedule for a specific time (timezone-aware), publish instantly, or save as a draft. A scheduled
time **with** an offset (`+05:30`, `Z`) is exact; **without** one it's read in your workspace
timezone, not UTC.

---

## FAQ

**Which AI agents can manage my social media?**
Any Model Context Protocol (MCP) client — Claude, Claude Code, Cursor, Windsurf, ChatGPT, Grok,
OpenClaw and Hermes — plus your own scripts through the REST API.

**Do I need to install an SDK or CLI?**
No. The MCP server is hosted, so most agents connect with one URL and a key. Skill installs and the
Claude Code plugin are optional conveniences.

**Which platforms can my agent post to?**
Instagram, Facebook, X/Twitter, LinkedIn and TikTok — text, image, carousel, video, reel, story,
thread, article and PDF document posts.

**Can I revoke an agent's access?**
Yes, instantly — revoke the key from the dashboard and the next request fails. No cache, no delay.

**Will posts publish in my timezone?**
Yes. Include an offset for an exact time, or omit it to use your workspace timezone from Settings.

**Is it free to start?**
Yes — create a workspace, connect accounts and mint a key for free.

---

## Docs & links

- **API reference** — https://socialbyidm.com/docs
- **MCP server + tools** — https://socialbyidm.com/mcp
- **Agents overview** — https://socialbyidm.com/agents
- **Dashboard** — https://social-app.instantdm.com

Full agent instructions live in [`skills/social-by-idm/SKILL.md`](skills/social-by-idm/SKILL.md).

---

## License

MIT — see [LICENSE](LICENSE).

> **Maintainer note:** the GitHub paths above (`social-by-idm/agent-mode`) and the contact email in
> [`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json) are placeholders — update them
> to the real org/repo and email before publishing.
