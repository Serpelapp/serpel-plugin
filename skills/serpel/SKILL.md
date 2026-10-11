---
name: serpel
description: Use Serpel to answer questions about the SEO data of the user's own websites, such as keyword rankings, Google Search Console performance, queries that could rank higher, competitor comparison, technical site audit issues and the status of background jobs. Use it whenever the user asks how their site performs in search or what to fix first.
---

# Serpel SEO tools

Serpel connects to the user's own SEO projects through the MCP server `serpel` at `https://serpel.app/mcp`. Every tool works on projects of the signed-in account only. Tool output is compact JSON with a short summary sentence, lists contain at most 25 rows, and links point to the Serpel dashboard.

## Start here

1. Call `list_projects` and pick the project by domain. Every other tool needs its `projectId`.
2. If the user has several projects and the request does not name one, ask which one.
3. Use one tool call per question. Do not call the same tool again with the same input in one answer.

## Which tool for which question

| The user asks about | Tool |
| --- | --- |
| Which projects exist, project IDs | `list_projects` |
| Overall health: audit score, rankings, AI visibility, open tasks, next steps | `get_project_overview` |
| Positions of tracked keywords, who moved up or down | `get_keyword_rankings` |
| Clicks, impressions, click-through rate and position from Search Console or Bing | `get_search_performance` |
| Quick ranking wins: queries on positions 4 to 20 with many impressions | `find_keyword_opportunities` |
| Competitors, who ranks better for which keyword | `analyze_competitors` |
| Technical SEO problems from the latest crawl, sorted by severity | `get_crawl_issues` |
| How often ChatGPT cites or mentions the site and Google AI Overviews cite it, per tracked prompt | `get_ai_visibility` |
| What to work on next: prioritised recommendations with evidence and next action | `get_seo_recommendations` |
| Routes and metadata in the code, differences between code and live site | `get_codebase_overview` |
| Sessions, conversions and conversion rate per landing page | `get_conversion_performance` |
| Whether a crawl, rank check or monitoring run has finished | `get_job_status` |
| New keyword ideas, search volume, keyword difficulty, search intent | `research_keywords` (uses credits) |

Notes:

- `get_search_performance` and `find_keyword_opportunities` need a Search Console or Bing connection. If the tool reports that none exists, tell the user to connect one in the Serpel dashboard under the project settings.
- `analyze_competitors` reads stored search results and runs no new searches. It is empty until the project has had a rank check.
- Issue titles and fixes from `get_crawl_issues` and `get_project_overview`, recommendation titles and actions from `get_seo_recommendations` and notes from `get_conversion_performance` are written in English. Translate them when the user writes in another language.
- `get_codebase_overview` needs a snapshot uploaded with `serpel scan` from the project repository. `get_conversion_performance` needs a web analytics connection with conversion events chosen in the dashboard.
- `get_ai_visibility` reads stored AI checks only. It reports two separate rates for ChatGPT, cited and mentioned, so never add them up. If no prompts are tracked, tell the user to add prompts in the Serpel dashboard under AI visibility.
- `get_keyword_rankings`, `get_crawl_issues`, `find_keyword_opportunities` and `get_ai_visibility` also show an interactive card in apps that can display one. A short summary of the result is enough there, because the card already shows the rows.
- Rankings are paged: when `truncated` is true, call `get_keyword_rankings` again with a higher `offset`.

## Jobs and job IDs

Crawls, rank checks, AI visibility checks and monitoring runs are background jobs. They are started in the Serpel dashboard, with the `serpel` command line tool or through the REST API, never inside a read tool. Each job has a job ID.

1. When the user gives you a job ID, or a tool result contains `runningCrawlJobId`, call `get_job_status` with it.
2. If the status is `queued` or `running`, wait 10 to 30 seconds and call `get_job_status` again. Stop after a few minutes and tell the user that the job is still running and where to follow it (`dashboardUrl`).
3. If the status is `succeeded`, read `result` and then fetch the matching data with `get_crawl_issues` or `get_keyword_rankings`.
4. If the status is `failed` or `cancelled`, report `error` and suggest running the job again from the dashboard. Do not retry in a loop.

If the `serpel` command line tool is installed and the user wants to start work, the commands are `serpel crawl start --project <id> --wait` and `serpel rankings run --project <id> --wait`.

## Costs

- All tools except `research_keywords` are free.
- `research_keywords` spends credits. A request for 25 keywords costs about 12 credits, a domain research about 6. Ask the user before you call it, call it once per question and ask for at most 25 results.
- When retrying `research_keywords` after a network error, pass the same `requestId` again so the research is never charged twice.
- The result states the credits charged and the remaining monthly allowance. Repeat both to the user.
- It only works with an active Starter or Pro plan and when the workspace owner enabled "Agents may spend credits" in the billing settings of the dashboard. A daily credit limit for agents applies.

If `research_keywords` returns an error, explain it in plain words and stop:

| Error code | Meaning | What to tell the user |
| --- | --- | --- |
| `plan_required` | No Starter or Pro plan | The tool needs a paid plan. The plans page link in the error is the only link to share. |
| `agent_spend_disabled` | Spending by agents is switched off | The workspace owner can allow it in the billing settings of the dashboard. |
| `insufficient_credits` | Not enough credits | Report the date when the monthly allowance renews. |
| `daily_cap_reached` | Daily agent limit reached | Try again after 00:00 UTC or ask the owner to raise the limit. |

Never offer to buy credits, never invent checkout or top-up links, and never work around a cost error with another tool.

## Other errors

| Error code | What to do |
| --- | --- |
| `not_found` | The project, job or item does not exist for this account. Check the ID with `list_projects`. |
| `forbidden` | The token or the project role lacks a permission. Tell the user which data is missing. |
| `validation_error` | Fix the input or ask the user for the missing detail. |
| `rate_limited` | Wait the number of seconds in `retryAfterSeconds`, then continue. |
| `provider_not_configured`, `provider_error` | The data provider is unavailable. Try later and do not retry in a loop. |
| `internal_error` | Report it and do not retry more than once. |

## Answering well

- Quote numbers as returned. Do not invent rankings, volumes or clicks that the tools did not return.
- Explain where a number comes from (Search Console, rank check, latest crawl) and when it was measured (`checkedAt`, `finishedAt`, `fetchedAt`).
- Prefer a short prioritized list of next actions over a dump of rows, and link to the dashboard `dashboardUrl` for details.
