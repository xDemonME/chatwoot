# Embedding a conversation in an iframe

Chatwoot can render a single conversation as a standalone page that contains only
the conversation window and the reply box — no sidebar, no conversation list and
(by default) no contact panel. This is useful for embedding the conversation view
into an external application such as a CRM that is integrated with Chatwoot.

## Embed URL

```
https://<your-chatwoot-host>/app/accounts/<accountId>/embed/conversations/<conversationId>
```

Place it in an iframe on the host application:

```html
<iframe
  src="https://<your-chatwoot-host>/app/accounts/1/embed/conversations/123"
  style="width:100%;height:100%;border:0"
  allow="clipboard-write"
></iframe>
```

You can also grab a ready-made snippet from the conversation header: open the
**…** (more actions) menu and choose **Copy embed code**.

### Show the contact panel

The right-hand contact panel is hidden by default. Append `?contactPanel=1` to the
URL to show it:

```
https://<your-chatwoot-host>/app/accounts/1/embed/conversations/123?contactPanel=1
```

## Allow framing (required)

By default Chatwoot only allows the conversation page to be framed by itself
(`frame-ancestors 'self'`). To embed it on a different host you must allow that
host explicitly through the `EMBED_FRAME_ANCESTORS` installation config.

1. Open the Super Admin console at `/super_admin/installation_configs`.
2. Find **EMBED_FRAME_ANCESTORS** (Embeddable Conversation Frame Ancestors).
3. Set it to a space-separated list of the **origins of the parent pages** that are
   allowed to embed Chatwoot. Keep `'self'` if you also want Chatwoot to frame
   itself. Examples:

   ```
   'self' https://crm.example.com
   ```

   ```
   'self' https://*.example.com
   ```

4. Save. The value is applied immediately (the config cache is cleared on save).

> The value must be the origin (`scheme://host[:port]`) of the page that hosts the
> iframe — not the Chatwoot host. Multiple origins are separated by spaces.

This value feeds the `Content-Security-Policy: frame-ancestors …` header that is
sent for `/embed/` pages (and for the login/auth pages, so the login screen still
renders inside the iframe when the agent is not yet authenticated). The legacy
`X-Frame-Options` header is removed for these pages so it does not conflict.

You can verify the header:

```bash
curl -sI "https://<your-chatwoot-host>/app/accounts/1/embed/conversations/123" \
  | grep -i content-security-policy
# Content-Security-Policy: frame-ancestors 'self' https://crm.example.com
```

## Authentication

The embedded page reuses the agent's existing Chatwoot session, so the agent must
be logged in. If they are not, the login screen is shown inside the iframe.

Cookies are sent automatically as long as the parent page and Chatwoot share the
same registrable domain (for example `crm.example.com` embedding
`chat.example.com`) — different sub-domains of the same domain are treated as
same-site by the browser. Embedding across completely different domains is not
supported, because the session cookie would not be sent to the iframe.
