# Repolog MCP Server

Website audits for your coding agent. Connect [Repolog](https://repolog.io) to Claude Code, Cursor, Claude Desktop, VS Code, or any MCP client. Your agent can then scan your pages for **SEO, performance, accessibility, security, and AI readiness**, read the scored report, and get a ready-made prompt for fixing what it found. Keyword research, Google Search Console data, and backlinks are included too.

This is a **hosted (remote) MCP server**. There is nothing to install or run. You point your client at the endpoint and authenticate with a Repolog API key.

```
https://repolog.io/api/mcp
```

Transport: Streamable HTTP · Auth: `Authorization: Bearer rl_…`

## Get an API key

1. Sign up at [repolog.io](https://repolog.io). Every plan starts with a 7-day free trial.
2. Add your website, and verify it through DNS if you want to scan pages other than the homepage.
3. Go to **Dashboard → API keys** and create a key with the scopes `websites:read`, `websites:write`, `scans:read`, and `scans:write`. Grant fewer scopes if you only want read access.

## Setup

### Claude Code (plugin)

```bash
claude plugin marketplace add repolog/repolog-mcp
claude plugin install repolog@repolog
```

Claude Code asks for your API key when you install the plugin and stores it securely.

### Claude Code (manual)

```bash
claude mcp add --transport http repolog https://repolog.io/api/mcp \
  --header "Authorization: Bearer rl_YOUR_KEY"
```

### Cursor

`.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "repolog": {
      "url": "https://repolog.io/api/mcp",
      "headers": { "Authorization": "Bearer rl_YOUR_KEY" }
    }
  }
}
```

### VS Code (Copilot)

`.vscode/mcp.json`:

```json
{
  "servers": {
    "repolog": {
      "type": "http",
      "url": "https://repolog.io/api/mcp",
      "headers": { "Authorization": "Bearer rl_YOUR_KEY" }
    }
  }
}
```

### Claude Desktop

`claude_desktop_config.json`. This setup uses [`mcp-remote`](https://www.npmjs.com/package/mcp-remote) to bridge to the hosted server:

```json
{
  "mcpServers": {
    "repolog": {
      "command": "npx",
      "args": [
        "-y", "mcp-remote", "https://repolog.io/api/mcp",
        "--header", "Authorization: Bearer ${REPOLOG_API_KEY}"
      ],
      "env": { "REPOLOG_API_KEY": "rl_YOUR_KEY" }
    }
  }
}
```

### Any other client

Send JSON-RPC `POST` requests to `https://repolog.io/api/mcp` with the `Authorization: Bearer rl_…` header. The server is stateless and exposes only tools.

## Tools

| Tool | Scope | What it does |
| --- | --- | --- |
| `list_websites` | websites:read | Lists the websites in your workspace. |
| `list_pages` | websites:read | Lists a website's registered pages. Their ids are what the scan and keyword tools take. |
| `add_pages` | websites:write | Registers pages for scanning, by path or URL. Your sitemap is the usual source. |
| `run_scan` | scans:write | Starts an audit of the homepage or a registered page. |
| `wait_for_scan` | scans:read | Waits up to about 45s for a scan to finish, then returns the report. |
| `get_scan` | scans:read | Returns scores, pass/warn/fail counts, and the top issues. Pass `view="checks"` to get every check. |
| `list_scans` | scans:read | Shows a website's scan history. |
| `get_fix_prompt` | scans:read | Returns a ready-to-use prompt that tells your agent what to fix in the code. |
| `list_keywords` | websites:read | Lists tracked keywords with volume, difficulty, CPC, and Search Console metrics. |
| `get_search_console_queries` | websites:read | Returns untracked queries you already rank for, plus keyword cannibalization. |
| `run_keyword_research` | websites:write | Runs keyword research from seed terms, using one of your monthly research runs. |
| `track_keyword` / `untrack_keyword` | websites:write | Starts or stops tracking keywords. Both take one keyword or a list. |
| `set_keyword_target_page` | websites:write | Maps keywords to the page that should rank for them. |
| `get_backlinks` | websites:read | Returns domain rank, backlinks, referring domains, spam score, and weekly history. |
| `get_usage` | websites:read | Shows your plan and how much of each quota is used and left. |

## Example prompts

- *"Scan the pricing page on example.com and fix the top accessibility issues."*
- *"Which Search Console queries am I getting impressions for but not tracking? Map the best ones to the right pages."*
- *"Compare this week's scan of the homepage with last week's. What regressed?"*

## Links

- Website: <https://repolog.io>
- Pricing: <https://repolog.io/pricing>
- Support: open an issue in this repository

This repository only lists the hosted Repolog MCP server. It holds no server source code.
