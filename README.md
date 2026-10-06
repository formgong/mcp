# Formgong MCP server

[![Formgong MCP connector – tool definition quality and endpoint health on Glama](https://glama.ai/mcp/connectors/com.formgong/mcp/badges/score.svg)](https://glama.ai/mcp/connectors/com.formgong/mcp)
[![Listed on mcpservers.org](https://mcpservers.org/badge.svg)](https://mcpservers.org/servers/formgong/mcp)

> Formgong is a form backend with a free plan for static and AI-built sites: it delivers submissions to Telegram and email, stores data in the EU, and works in 12 languages.
>
> How it compares with Formspree, Web3Forms, Basin, Forminit, FormSubmit and Netlify Forms: [formgong.com/en/compare](https://formgong.com/en/compare/)

A remote [Model Context Protocol](https://modelcontextprotocol.io) server for [Formgong](https://formgong.com), a hosted form backend for static and AI-built websites. Your AI assistant can **create contact forms, get ready-to-paste form code and read recent submissions**, so you never have to copy access keys by hand.

- **URL:** `https://formgong.com/mcp`
- **Transport:** Streamable HTTP. It's stateless and returns JSON responses.
- **Auth:** two ways, pick whichever your client supports.
  - **OAuth 2.1** (recommended): authorization code with mandatory PKCE (S256) and [dynamic client registration](https://datatracker.ietf.org/doc/html/rfc7591) at `https://formgong.com/oauth/register`, so clients such as Claude Desktop, Cursor, VS Code, Lovable and Bolt can connect with a browser sign-in and no copy-pasted secret. A tool call without credentials returns `401` with `WWW-Authenticate: Bearer realm="formgong", resource_metadata="https://formgong.com/.well-known/oauth-protected-resource/mcp", scope="forms:read forms:write"` ([RFC 9728](https://datatracker.ietf.org/doc/html/rfc9728)); metadata is at [`/.well-known/oauth-authorization-server`](https://formgong.com/.well-known/oauth-authorization-server) and [`/.well-known/oauth-protected-resource/mcp`](https://formgong.com/.well-known/oauth-protected-resource/mcp). Access tokens last 1 hour and rotate with refresh tokens.
  - **Personal API token:** `Authorization: Bearer fgp_…` for clients that only send static headers.
  - Either way, `initialize`, `ping` and `tools/list` work unauthenticated, so clients and directories can see the tools before signing in.
- **Scopes:** `forms:read` (always granted), `forms:write`, `submissions:read` (off unless requested).
- **Docs:** https://formgong.com/en/docs/mcp/ (in 12 languages)

There's nothing to install: the server runs on formgong.com. This repo holds the documentation and the [`server.json`](./server.json) for the [official MCP Registry](https://registry.modelcontextprotocol.io) (`com.formgong/mcp`).

A short reusable [server card](./server-card.md) lists the connection details, scopes and example prompts.

## What Formgong does

- **Delivery:** Telegram at once on every plan. Free email arrives as a daily digest; Pro and Business email each submission. Telegram connects with one button, groups included. Signed JSON webhooks (HMAC-SHA256 `X-Signature`, up to 5 delivery attempts) come with step-by-step recipes for [Make, n8n, Zapier and KeyCRM](https://formgong.com/en/integrations/).
- **EU data:** submissions are stored in the EU (Cloudflare D1 with EU jurisdiction), and the IP address is kept only as a hash. A standard Art. 28 DPA is part of the [Terms](https://formgong.com/en/terms/).
- **Spam:** no CAPTCHA puzzle and no cookies. There's a honeypot, cookie-free checks and optional Cloudflare Turnstile on every plan. Plain HTML forms without JavaScript keep working.
- **12 languages:** the thank-you page, errors and auto-reply follow the visitor's language.
- **Agencies:** [Projects](https://formgong.com/en/for/agencies/) group forms per client (up to 100 on every plan), with view-only client invites and handover to the client's own account.
- **Pricing:** the free plan has 300 submissions a month and unlimited forms. Pro is $5 a month and Business $15 (as of Oct 2026). Free shows a small "Form powered by Formgong" link on the hosted thank-you page and in emails.

## Tools

| Tool | Scope | What it does |
| --- | --- | --- |
| `list_forms` | `forms:read` | Lists your forms: id, name, public access key (`fk_…`) and submissions this month. |
| `create_form` | `forms:write` | Creates a form (`name`, `notify_email`). Submissions are emailed to your account address. It returns the form id, the access key and dashboard links for connecting Telegram and extra recipients. Limit: 10 new forms per hour. |
| `get_form_snippet` | `forms:read` | Returns code for one form (`form_id`, `framework`: `html` / `react` / `next`, `lang`), with its access key, the `_lang` field, the `botcheck` honeypot, and Turnstile when it's enabled. |
| `list_recent_submissions` | `submissions:read` | Read-only and opt-in. Returns up to 50 recent submissions with time and submitted fields only (no IP or user agent). Spam is excluded by default. Field values are marked as untrusted visitor input. |

## 1. Create an API token (only if your client can't do OAuth)

Clients with an OAuth connector skip this step entirely — they sign you in through the browser. Create a token only for clients that send a static `Authorization` header.

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
      "auth": { "CLIENT_ID": "cursor", "scopes": ["forms:read", "forms:write"] }
    }
  }
}
```

### Claude Code

```bash
claude mcp add --transport http formgong https://formgong.com/mcp
```

Approve the browser sign-in. If you prefer a personal token, add `--header "Authorization: Bearer fgp_your_token"`.

### Claude Desktop

Settings → Connectors → Add custom connector, enter `https://formgong.com/mcp` and click Connect. Claude Desktop discovers the OAuth metadata from the `401` challenge, registers itself dynamically and opens a browser window where you approve the scopes — no token to paste.

If your client can't do OAuth and only sends static headers, use the [`mcp-remote`](https://www.npmjs.com/package/mcp-remote) bridge (it requires Node.js) in `claude_desktop_config.json`, then restart Claude Desktop:

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

- **Lovable:** open Connectors → + → MCP server. Name it Formgong and enter `https://formgong.com/mcp`. Keep Direct connection and OAuth; click Add & authorize, sign in to Formgong and approve the permissions. A personal token also works through Bearer token or API key.
- **Bolt:** open Settings → Connectors (MCP) → Custom MCP server. Name: Formgong. URL: `https://formgong.com/mcp`. Transport: HTTP. Authentication: MCP OAuth. Click Connect, sign in and approve the permissions, then turn on the connector for your project. API key remains available with a personal Formgong token.

The setup controls are documented by [Lovable](https://docs.lovable.dev/integrations/custom-mcp) and [Bolt](https://support.bolt.new/building/using-bolt/connect-mcp). A connector lets the builder obtain code; the published contact form still posts directly to Formgong.

### v0

Open the + menu beside the prompt and choose MCPs. Configure a custom server with URL `https://formgong.com/mcp`, choose Bearer Token and paste your personal Formgong API token. These controls are documented in [v0 MCP Integrations](https://v0.app/docs/MCP). Use a personal token because Formgong does not currently allow v0 OAuth callbacks. The [v0 contact-form guide](https://formgong.com/en/docs/v0/) also has a direct form prompt; the published form sends directly to Formgong.

### Other clients

Clients that support remote MCP over Streamable HTTP can use the same URL with a personal token. Browser sign-in requires a callback accepted by Formgong: the listed Cursor/Claude/VS Code callbacks, HTTPS on exactly `lovable.dev` or `bolt.new` (no subdomains, nonstandard ports, userinfo, query or fragment), or an HTTP loopback callback. Each dynamic client stays bound to its complete registered URI. Other hosted origins are refused even when a client calls itself Lovable or Bolt. Stdio-only clients can use the `mcp-remote` bridge.

## Try it

- "Create a Formgong form called 'Contact – acme.com' and add it to the contact page as a React component."
- "List my Formgong forms and give me the Next.js snippet for the newest one in German."
- "Show the last 5 submissions of my contact form." (This needs a token with "Read recent submissions".)

## Security

- Tokens are scoped (`forms:read`, `forms:write`, `submissions:read`), stored only as hashes, can expire, and can be revoked in the dashboard.
- OAuth grants work the same way: `submissions:read` is never granted unless the client asks for it, access tokens expire after 1 hour, refresh tokens after 30 days with rotation, and every connected app is listed under **Dashboard → Account → Connected apps**, where you can revoke it. `https://formgong.com/oauth/revoke` implements [RFC 7009](https://datatracker.ietf.org/doc/html/rfc7009).
- Each token can make 60 requests a minute. Requests without a token are limited to 30 a minute per IP, and failed authentication attempts are rate-limited per IP.
- A tool call without valid credentials gets HTTP `401` with a JSON-RPC error (code `-32001`) whose `data` carries `resourceMetadata`, so a client can start the OAuth flow or tell the user where to create a token.
- The tools only see forms the token owner owns. Submissions never include IP addresses or user agents.
- Report security issues to support@formgong.com.

## Related

- Starters: [nextjs-starter](https://github.com/formgong/nextjs-starter), [astro-starter](https://github.com/formgong/astro-starter), [html-starter](https://github.com/formgong/html-starter), [react-contact-form](https://github.com/formgong/react-contact-form)
- npm ([source](https://github.com/formgong/js)): [`formgong`](https://www.npmjs.com/package/formgong) (`npx formgong init`), [`create-formgong`](https://www.npmjs.com/package/create-formgong), [`@formgong/react`](https://www.npmjs.com/package/@formgong/react) and packages for Next.js, Vue, Svelte, Astro and Angular
- Integration recipes (Make, n8n, Zapier, KeyCRM): https://formgong.com/en/integrations/
- Docs for AI builders: https://formgong.com/en/docs/

## License

MIT © Formgong. This license covers the documentation and config in this repo. The hosted service is covered by https://formgong.com/terms.
