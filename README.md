# Serpel plugin for AI assistants

Ask Claude, ChatGPT, Codex, Cursor, GitHub Copilot, VS Code or the Gemini CLI about the SEO data of your own websites. The plugin connects your assistant to the [Serpel](https://serpel.app) MCP server and adds a skill that explains which tool to use when.

The plugin contains manifests and one skill only. It has no program code and no hooks.

## What you can ask

- Which projects do I have, and how healthy is each one?
- Which keywords moved up or down since the last rank check?
- What do Google Search Console clicks and impressions look like compared with the previous period?
- Which queries rank on positions 4 to 20 and are quick wins?
- Where do my competitors rank better than I do?
- Which technical issues did the latest crawl find, and what should I fix first?
- How often is my site cited or mentioned in ChatGPT answers and in Google AI Overviews?
- Has my crawl or rank check finished?
- Optional, uses credits: keyword ideas with search volume, difficulty and intent.

## Tools

| Tool | Purpose | Costs |
| --- | --- | --- |
| `list_projects` | Projects of your account with IDs and domains | free |
| `get_project_overview` | Audit score, rankings, AI visibility, tasks and next steps | free |
| `get_keyword_rankings` | Tracked keywords with positions and changes | free |
| `get_search_performance` | Search Console or Bing clicks, impressions, CTR, position | free |
| `find_keyword_opportunities` | Queries on positions 4 to 20 with impressions | free |
| `analyze_competitors` | Visibility and head-to-head comparison from stored results | free |
| `get_crawl_issues` | Issues of the latest crawl by severity | free |
| `get_ai_visibility` | Citations and mentions in ChatGPT answers and Google AI Overviews | free |
| `get_job_status` | Status and result of a background job | free |
| `get_seo_recommendations` | Prioritised recommendations with impact, effort and evidence | free |
| `get_codebase_overview` | Routes, metadata and live-page mapping from the codebase snapshot | free |
| `get_conversion_performance` | Sessions, conversions and conversion rate per landing page | free |
| `research_keywords` | Keyword ideas and metrics | credits |

All lists contain at most 25 rows. Tools only read data of projects you can access, and they never change your website.

ChatGPT does not offer `research_keywords`. In all other clients it needs an active Starter or Pro plan and the workspace setting “Agents can spend credits”. A daily credit limit for agents applies, and credits come from your monthly plan allowance first.

## Interactive cards

In apps that can display MCP Apps, such as ChatGPT and Claude, four tools show their result as an interactive card: `get_keyword_rankings`, `get_crawl_issues`, `find_keyword_opportunities` and `get_ai_visibility`. Every other client, for example Claude Code, Cursor, Codex and the Gemini CLI, gets the same data as text. The cards load no external resources and open links only to the Serpel dashboard.

## Set up

The server address is `https://serpel.app/mcp` (Streamable HTTP). Clients that support OAuth open a browser window for the sign-in on first use. Alternatively create an API token with the `mcp:read` permission (plus the read permissions for the data you want to use) in the Serpel dashboard under Settings, API tokens, and send it as `Authorization: Bearer <token>`.

### Claude Code

```bash
claude mcp add --transport http serpel https://serpel.app/mcp
```

To load the plugin with its skill from a local checkout: `claude --plugin-dir ./serpel-plugin`.

### ChatGPT and Codex

Install the plugin from the plugin directory, or add the remote MCP server `https://serpel.app/mcp` in the connector settings and sign in with your Serpel account.

### Cursor

Add the server in `.cursor/mcp.json` or install the plugin from the marketplace:

```json
{
  "mcpServers": {
    "serpel": { "url": "https://serpel.app/mcp" }
  }
}
```

### VS Code with GitHub Copilot

Add the server in `.vscode/mcp.json` or install it from the MCP gallery (`@mcp`):

```json
{
  "servers": {
    "serpel": { "type": "http", "url": "https://serpel.app/mcp" }
  }
}
```

### Gemini CLI

```bash
gemini extensions install https://github.com/Serpelapp/serpel-plugin
```

## Files

| File | For |
| --- | --- |
| `plugin.json`, `mcp.json`, `skills/serpel/SKILL.md` | Agent Plugins 1.0: ChatGPT, Codex, Cursor, GitHub Copilot, VS Code |
| `chatGpt/skills/serpel/SKILL.md` | Skill for the ChatGPT app directory, without keyword research, which ChatGPT does not offer |
| `.claude-plugin/plugin.json`, `.mcp.json` | Claude Code |
| `gemini-extension.json` | Gemini CLI |
| `server.json` | Official MCP registry, name `app.serpel/mcp` |
| `assets/` | Icon |

All manifests carry the same version.

## Privacy and support

- Privacy policy: https://serpel.app/privacy
- Terms: https://serpel.app/terms
- Support and contact: https://serpel.app/about#how-can-you-contact-serpel
- Legal notice: https://serpel.app/imprint

Tool results contain data from your Serpel account only. Disconnect a client at any time in the Serpel dashboard under Settings, API, connected apps.
