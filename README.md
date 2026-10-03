# Formgong MCP server

[![Formgong MCP connector – tool definition quality and endpoint health on Glama](https://glama.ai/mcp/connectors/com.formgong/mcp/badges/score.svg)](https://glama.ai/mcp/connectors/com.formgong/mcp)

> Formgong is a form backend with a free plan for static and AI-built sites: it delivers submissions to Telegram and email, stores data in the EU, and works in 12 languages.

A remote [Model Context Protocol](https://modelcontextprotocol.io) server for [Formgong](https://formgong.com), a hosted form backend for static and AI-built websites. Your AI assistant can **create contact forms, get ready-to-paste form code and read recent submissions**, so you never have to copy access keys by hand.

- **URL:** `https://formgong.com/mcp`
- **Transport:** Streamable HTTP. It's stateless and returns JSON responses.
- **Auth:** `Authorization: Bearer fgp_…` with a personal API token. OAuth is not supported yet.
- **Docs:** https://formgong.com/en/docs/mcp/ (in 12 languages)

There's nothing to install: the server runs on formgong.com. This repo holds the documentation and the [`server.json`](./server.json) for the [official MCP Registry](https://registry.modelcontextprotocol.io) (`com.formgong/mcp`).

## Tools

| Tool | Scope | What it does |
| --- | --- | --- |
| `list_forms` | `forms:read` | Lists your forms: id, name, public access key (`fk_…`) and submissions this month. |
| `create_form` | `forms:write` | Creates a form (`name`, `notify_email`). Submissions are emailed to your account address. It returns the form id, the access key and dashboard links for connecting Telegram and extra recipients. Limit: 10 new forms per hour. |
| `get_form_snippet` | `forms:read` | Returns code for one form (`form_id`, `framework`: `html` / `react` / `next`, `lang`), with its access key, the `_lang` field, the `botcheck` honeypot, and Turnstile when it's enabled. |
| `list_recent_submissions` | `submissions:read` | Read-only and opt-in. Returns up to 50 recent submissions with time and submitted fields only (no IP or user agent). Spam is excluded by default. Field values are marked as untrusted visitor input. |

## 1. Create an API token

1. Sign up at https://formgong.com (the free plan includes 300 submissions a month, and data is stored in the EU), then confirm your email.
2. Open **Dashboard → Account → API tokens**.
3. Name the token after the tool that will use it, and choose the access:
   - "Create forms" is on by default.
   - "Read recent submissions" is off unless you turn it on.
   - You can set an optional expiry.
4. Copy the token (`fgp_…`). It's shown only once, and Formgong stores only a hash of it. You can revoke it from the same page at any time.

## 2. Add the server to your client

### Cursor

`~/.cursor/mcp.json` (all projects) or `.cursor/mcp.json` (one project):

```json
{
  "mcpServers": {
    "formgong": {
      "url": "https://formgong.com/mcp",
      "headers": { "Authorization": "Bearer ${env:FORMGONG_TOKEN}" }
    }
  }
}
```

### Claude Code

```bash
claude mcp add --transport http formgong https://formgong.com/mcp \
  --header "Authorization: Bearer fgp_your_token"
```

### Claude Desktop

Custom connectors in Settings → Connectors require OAuth, which Formgong doesn't support yet. Instead, use the [`mcp-remote`](https://www.npmjs.com/package/mcp-remote) bridge (it requires Node.js) in `claude_desktop_config.json`, then restart Claude Desktop:

```json
{
  "mcpServers": {
    "formgong": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://formgong.com/mcp", "--header", "Authorization:${FORMGONG_AUTH}"],
      "env": { "FORMGONG_AUTH": "Bearer fgp_your_token" }
    }
  }
}
```

### VS Code (Copilot agent mode)

`.vscode/mcp.json`. VS Code asks for the token once and stores it securely:

```json
{
  "inputs": [{ "type": "promptString", "id": "formgong-token", "description": "Formgong API token (fgp_…)", "password": true }],
  "servers": {
    "formgong": {
      "type": "http",
      "url": "https://formgong.com/mcp",
      "headers": { "Authorization": "Bearer ${input:formgong-token}" }
    }
  }
}
```

### Windsurf

`~/.codeium/windsurf/mcp_config.json`:

```json
{
  "mcpServers": {
    "formgong": {
      "serverUrl": "https://formgong.com/mcp",
      "headers": { "Authorization": "Bearer ${env:FORMGONG_TOKEN}" }
    }
  }
}
```

### Lovable and Bolt

- **Lovable:** go to Settings → Connectors → Custom MCP server. Enter URL `https://formgong.com/mcp`, choose "Bearer token or API key", and paste only the token (without the word `Bearer`).
- **Bolt:** go to Settings → Connectors (MCP) → Custom MCP server. Enter URL `https://formgong.com/mcp`, set Transport to HTTP and Authentication to API key, then paste the token.

### Other clients

Any client that supports remote MCP over Streamable HTTP with custom headers works with the same URL and header. Clients that only support stdio can use the `mcp-remote` bridge shown above.

## Try it

- "Create a Formgong form called 'Contact – acme.com' and add it to the contact page as a React component."
- "List my Formgong forms and give me the Next.js snippet for the newest one in German."
- "Show the last 5 submissions of my contact form." (This needs a token with "Read recent submissions".)

## Security

- Tokens are scoped (`forms:read`, `forms:write`, `submissions:read`), stored only as hashes, can expire, and can be revoked in the dashboard.
- Each token can make 60 requests a minute, and failed authentication attempts are rate-limited per IP.
- The tools only see forms the token owner owns. Submissions never include IP addresses or user agents.
- Report security issues to support@formgong.com.

## Related

- Starters: [nextjs-starter](https://github.com/formgong/nextjs-starter), [astro-starter](https://github.com/formgong/astro-starter), [html-starter](https://github.com/formgong/html-starter), [react-contact-form](https://github.com/formgong/react-contact-form)
- npm: [`create-formgong`](https://www.npmjs.com/package/create-formgong), [`@formgong/react`](https://www.npmjs.com/package/@formgong/react)
- Docs for AI builders: https://formgong.com/en/docs/

## License

MIT © Formgong. This license covers the documentation and config in this repo. The hosted service is covered by https://formgong.com/terms.
