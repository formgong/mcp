# Formgong — contact forms for AI-built sites

Let your assistant create a contact form, retrieve HTML/React/Next.js code with its public access key, and read recent submissions when you explicitly permit it. Published forms send directly to Formgong; no custom form backend is needed.

| Connection | Value |
| --- | --- |
| Endpoint | `https://formgong.com/mcp` |
| Transport | Streamable HTTP, JSON responses |
| Authentication | OAuth 2.1 browser sign-in with mandatory PKCE S256, or `Authorization: Bearer fgp_…` |
| Public discovery | `initialize`, `ping`, `tools/list`; every tool call requires authentication |
| Tools | `list_forms`, `create_form`, `get_form_snippet`, `list_recent_submissions` |
| Permissions | `forms:read`; optional `forms:write` and `submissions:read` |
| Data access | Only the connected account’s own forms; reading submissions requires explicit permission |

**Connect:** [Lovable and Bolt setup](https://formgong.com/en/docs/mcp/) · [Cursor guide](https://formgong.com/en/docs/cursor/) · [v0 direct form integration](https://formgong.com/en/docs/v0/).

**Try:** “Create a Formgong form called Contact and add it to my site.” Then send a test submission and check the Formgong inbox. To read messages, request and approve `submissions:read` first.

**Delivery:** Telegram is immediate on all plans. Free email is a daily digest; paid plans email each submission. Formgong stores submissions in the EU. Optional external destinations, such as Google Sheets, follow their provider’s storage policy.

[Service and pricing](https://formgong.com/en/pricing/) · [Full setup and security](https://formgong.com/en/docs/mcp/) · [Official registry](https://registry.modelcontextprotocol.io/?q=com.formgong%2Fmcp) · [Source](https://github.com/formgong/mcp)
