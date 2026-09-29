# Source: https://www.adscrawl.net/docs

On this page

## Authenticate every account-level request

Send your complete API key in the x-api-key header. Keep the key on a trusted server or in a secret manager; never publish it in client-side code.

Cloud browser lifecycle requests also accept a session JWT or cookie. Viewer access requires the profile owner's session; an API key alone does not grant viewer access.

**Session tokens have narrower scopes.**

Remote CDP discovery uses the data token embedded in the session URL. Live control uses a separate single-use token that expires after 30 seconds.

## Base URL

`https://api.adscrawl.net`

[Download OpenAPI (YAML)](https://www.adscrawl.net/openapi.yaml)

The public API origin follows the website deployment environment, so local, development, pre-release, and production documentation point to their matching service.

## Make your first request

Start with POST /html when you need rendered content. It is synchronous, uses safe browser defaults, and avoids managing a stateful session.

cURL

```bash
export ADSCRAWL_API_KEY="your-api-key"

curl --fail-with-body -sS -X POST "https://api.adscrawl.net/html" \
  -H "content-type: application/json" \
  -H "x-api-key: $ADSCRAWL_API_KEY" \
  -d '{
    "url": "https://example.com/article",
    "contentMode": "markdown",
    "waitUntil": "domcontentloaded"
  }'
```

## Choose the smallest browser workflow

[Rendered content\\ \\ HTML, Markdown, readable text, or structured article data from one URL.](https://www.adscrawl.net/docs/browser-tasks/#html) [Structured extraction\\ \\ DOM and network fields from an interactive SPA, with optional actions or site templates.](https://www.adscrawl.net/docs/browser-tasks/#spa-extract) [Stateful browser\\ \\ A durable Chromium session for Playwright, Puppeteer, or direct CDP control.](https://www.adscrawl.net/docs/remote-cdp/#create-session) [Cloud browsers\\ \\ Saved profiles, lifecycle, proxies, and limits](https://www.adscrawl.net/docs/cloud-browsers/#cloud-launch)