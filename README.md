# Onetime Secret for Zendesk Support

The official Onetime Secret app for Zendesk Support. Share passwords, credentials, and other sensitive data with customers through encrypted, self-destructing links directly from your ticket workspace.

## Features

- **Create secret links** from the ticket sidebar without leaving Zendesk
- **Region support** for US, EU, CA, UK, and NZ data centers
- **Custom domains** for branded sharing links (e.g. `secrets.yourcompany.com`)
- **Passphrase protection** so only the intended recipient can view
- **Configurable TTL** from 5 minutes to 30 days
- **One-click insert** directly into ticket comments
- **Copy to clipboard** for use in any channel

## Installation

### From Zendesk Marketplace

Search for "Onetime Secret" in the Zendesk Marketplace and click Install.

### Manual / Development

1. Install the [Zendesk CLI (ZCLI)](https://developer.zendesk.com/documentation/apps/getting-started/using-zcli/)
2. Clone this repository
3. Run the local dev server from the project root:

   ```bash
   zcli apps:server "$(pwd)"
   ```

   If you have `zcli` installed via pnpm in another workspace, use `pnpm exec zcli apps:server "$(pwd)"`. Quoting the path is required if your checkout sits under a directory containing spaces or `&` — `zcli` splits unquoted paths on whitespace and fails with `Invalid app path`.
4. Package with `zcli apps:package "$(pwd)"` to create a ZIP for upload

### Testing the app in Zendesk

The local dev server only serves the app's static assets — `http://localhost:4567` will return 404 if you open it directly. Zendesk apps run inside the agent workspace, so you need a Zendesk instance to load the app into.

1. **Get a Zendesk Suite trial** at [zendesk.com/register](https://www.zendesk.com/register/) (14 days, no card). Pick a subdomain — you'll get `YOUR-SUBDOMAIN.zendesk.com`.
2. **Log in** at `https://YOUR-SUBDOMAIN.zendesk.com/agent`.
3. **Create a test ticket**: click the `+` icon top-left → Ticket → fill in any requester / subject → Submit as New.
4. **Accept the self-signed cert**: visit `https://localhost:4567` once and accept the browser warning (zcli signs its own cert).
5. **Load the app** by appending `?zcli_apps=true` to the agent URL:

   ```
   https://YOUR-SUBDOMAIN.zendesk.com/agent/?zcli_apps=true
   ```

6. **Open the ticket** you created — the Onetime Secret panel appears in the right sidebar, fetched live from your local server. Edits to `assets/iframe.html` are picked up on browser reload.

**Note on local dev vs. uploaded apps:** ZCLI's local server does not support secure settings or the ZAF proxy. The `{{setting.apiKey}}` and `{{basic_auth.token}}` placeholders are passed through as literal strings, so API calls will fail with 401. To test against the real OTS API, package the app with `zcli apps:package` and upload it as a private app via Admin Center → Apps → Upload private app.

## Configuration

After installing, you'll be prompted for these settings:

| Setting | Required | Description |
|---------|----------|-------------|
| API Username | Yes | Your Onetime Secret account email |
| API Key | Yes | Your API key (found on your account page) |
| Region | Yes | `us`, `eu`, `ca`, `uk`, or `nz` |
| Custom Domain | No | Your custom domain for branded links |

### Getting your API key

1. Sign in at your regional domain (e.g. [us.onetimesecret.com](https://us.onetimesecret.com))
2. Go to your Account page
3. Generate or copy your API key

## How it works

1. Agent opens a ticket and sees the Onetime Secret sidebar
2. Agent pastes sensitive content (password, API key, credentials, etc.)
3. Optionally sets a passphrase and expiry time
4. Clicks "Generate Link" which calls the OTS v2 API
5. The encrypted link appears with options to insert into the comment or copy
6. Once the recipient views the link, the secret is permanently destroyed

## Project structure

```
zendesk-app/
  manifest.json           # App configuration and settings
  assets/
    iframe.html           # Sidebar UI (self-contained)
  translations/
    en.json               # English translations and marketplace copy
  LICENSE                 # MIT
```

## API

This app uses the [Onetime Secret v2 API](https://docs.onetimesecret.com/en/rest-api/v2/):

- `POST /api/v2/secret/conceal` to create secrets
- HTTP Basic Auth with your account email and API key
- Supports `share_domain` for custom domain links
- All regional endpoints follow the pattern `https://{region}.onetimesecret.com`

## Security

- API keys are stored as secure parameters in Zendesk and sent via the ZAF server-side proxy (the browser never sees the raw key)
- The `domainWhitelist` restricts API calls to official OTS regional domains and your custom domain
- Secret content is encrypted at rest by OTS and destroyed after first view
- No secret content is ever stored in Zendesk

## License

MIT. See [LICENSE](./LICENSE).
