# Source: https://www.adscrawl.net/docs/browser-tasks

On this page

Screenshot, HTML, and SPA extraction are synchronous metered requests. Bodies are limited to 1 MiB, and one credit is consumed after validation but before enqueue. Random User-Agents, trusted proxies, and dynamic region routing are supported.

POST`/screenshot`

## Capture a page screenshot

Page Screenshot · Country/Region Proxy

### Request

#### Request headers

| Field | Type | Description |
| --- | --- | --- |
| `x-api-key`Required | `string` | Full API key created in the dashboard. |
| `content-type`Required | `application/json` | Request body must be JSON. |

#### Request body

| Field | Type | Description |
| --- | --- | --- |
| `url`Required | `string` | Must be a reachable HTTP(S) URL using port 80 or 443. |
| `viewport` | `{ width: number; height: number }` | Viewport size used for the screenshot. |
| `fullPage` | `boolean` | Whether to capture the full page. Defaults to true; selector captures only the matched element. |
| `selector` | `string` | Capture only the first matching element. Missing selectors return 422 CONTENT\_SELECTOR\_NOT\_FOUND. |
| `waitUntil` | `"load" | "domcontentloaded" | "networkidle"` | Defaults to load. Only load, domcontentloaded, and networkidle are accepted; other values fall back to load.
`domcontentloaded`

Waits for DOMContentLoaded. HTML is parsed without waiting for secondary resources such as images; recommended for HTML extraction.

`load`

Waits for window.load after the page and dependent resources such as images and stylesheets finish loading.

`networkidle`

Waits until there are no network connections for at least 500 ms. Long polling, analytics, or lazy-loaded resources may cause a timeout.

 |
| `timeoutMs` | `number` | Must be positive and no greater than 3,600,000; other values use the server default. |
| `locale` | `string` | Browser locale such as en-US. |
| `timezoneId` | `string` | IANA timezone id such as Asia/Shanghai. |
| `geolocation` | `{ latitude: number; longitude: number }` | Optional geolocation coordinates. |
| `proxy` | `proxy` | Custom proxy; cannot be combined with countryCode. |
| `countryCode` | `string` | Managed proxy region. GLOBAL selects a dynamic exit from 15 popular regions. A two-letter region code prefers a trusted proxy with dynamic fallback. Omitted values use a random trusted proxy. Cannot be combined with custom proxy. |
| `userAgentMode` | `"custom" | "random"` | Set to random to let the server select from the User-Agent library; requests without a User-Agent default to random. |
| `userAgentOs` | `"windows" | "macos"` | Operating system used by random mode. Defaults to windows. |
| `fingerprint` | `fingerprint` | Browser fingerprint settings. When omitted, every signal defaults to random while keeping the OS, GPU, CPU, memory, fonts, and device signals coherent. |
| `userAgent` | `string` | Override the default User-Agent. |
| `cookies` | `cookies[]` | Cookie list injected into the browser context before navigation. |

### Responses

200

`image/png`Returns a PNG binary stream.

400

`application/json`Invalid JSON, URL, cookies, proxy, region, or random User-Agent parameters. Bodies over 1 MiB are also rejected as invalid JSON.

401

`application/json`Missing or invalid x-api-key.

402

`application/json`Insufficient balance. Returns INSUFFICIENT\_CREDITS, balance, and requiredCredits.

422

`application/json`The selector did not match or the Worker rejected the task payload.

429

`application/json`The task was rate limited.

502

`application/json`Proxy unreachable, target HTTP failure, or an oversized inline result.

503

`application/json`Queue, Worker, managed proxy, or User-Agent resources are unavailable.

504

`application/json`Task, navigation, target, or proxy connection timed out.

500

`application/json`Unclassified task execution failure.

### Request examples

cURL

```bash
curl -sS -X POST "https://api.adscrawl.net/screenshot" \
  -H "content-type: application/json" \
  -H "x-api-key: YOUR_API_KEY" \
  -d '{"url":"https://example.com","viewport":{"width":1440,"height":900},"fullPage":true,"waitUntil":"load","countryCode":"GLOBAL","userAgentMode":"random","userAgentOs":"windows"}' \
  --output page.png
```

Body JSON

```json
{
  "url": "https://example.com",
  "viewport": { "width": 1440, "height": 900 },
  "fullPage": true,
  "waitUntil": "load",
  "locale": "zh-CN",
  "timezoneId": "Asia/Shanghai",
  "countryCode": "GLOBAL",
  "userAgentMode": "random",
  "userAgentOs": "windows"
}
```

### Response examples

200 image/png

```text
HTTP/1.1 200 OK
Content-Type: image/png

<binary PNG stream>
```

400 Proxy JSON

```json
{
  "error": "countryCode is not supported",
  "code": "INVALID_COUNTRY_CODE"
}
```

503 Proxy JSON

```json
{
  "error": "Dynamic country/region routing is unavailable",
  "code": "DYNAMIC_PROXY_NOT_CONFIGURED"
}
```

402 Credits JSON

```json
{
  "error": "Insufficient credits",
  "code": "INSUFFICIENT_CREDITS",
  "balance": 0,
  "requiredCredits": 1
}
```

POST`/html`

## HTML / Readability Extraction · Country/Region Proxy

### Request

#### Request headers

| Field | Type | Description |
| --- | --- | --- |
| `x-api-key`Required | `string` | Full API key created in the dashboard. |
| `content-type`Required | `application/json` | Request body must be JSON. |

#### Request body

| Field | Type | Description |
| --- | --- | --- |
| `url`Required | `string` | Target page URL. Only ports 80 and 443 are supported. |
| `contentMode` | `"html" | "markdown" | "json"` | Defaults to "html". "markdown" and "json" extract the readable article content. |
| `selector` | `string` | Wait for the first matching element. contentMode=html returns only that element HTML; markdown/json run Readability against the element. Missing selectors return 422. |
| `waitUntil` | `"load" | "domcontentloaded" | "networkidle"` | Defaults to load. domcontentloaded is usually preferred for HTML extraction; long polling or analytics can delay networkidle.
`domcontentloaded`

Waits for DOMContentLoaded. HTML is parsed without waiting for secondary resources such as images; recommended for HTML extraction.

`load`

Waits for window.load after the page and dependent resources such as images and stylesheets finish loading.

`networkidle`

Waits until there are no network connections for at least 500 ms. Long polling, analytics, or lazy-loaded resources may cause a timeout.

 |
| `timeoutMs` | `number` | Must be positive and no greater than 3,600,000; other values use the server default. |
| `viewport` | `{ width: number; height: number }` | Viewport size. |
| `locale` | `string` | Browser locale. |
| `timezoneId` | `string` | IANA timezone id. |
| `geolocation` | `{ latitude: number; longitude: number }` | Optional geolocation coordinates. |
| `proxy` | `proxy` | Custom proxy; cannot be combined with countryCode. |
| `countryCode` | `string` | Managed proxy region. GLOBAL selects a dynamic exit from 15 popular regions. A two-letter region code prefers a trusted proxy with dynamic fallback. Omitted values use a random trusted proxy. Cannot be combined with custom proxy. |
| `userAgentMode` | `"custom" | "random"` | Set to random to let the server select from the User-Agent library; requests without a User-Agent default to random. |
| `userAgentOs` | `"windows" | "macos"` | Operating system used by random mode. Defaults to windows. |
| `fingerprint` | `fingerprint` | Browser fingerprint settings. When omitted, every signal defaults to random while keeping the OS, GPU, CPU, memory, fonts, and device signals coherent. |
| `userAgent` | `string` | Override the default User-Agent. |
| `cookies` | `cookies[]` | Preloaded cookies. |

### Responses

200

`text/html; charset=utf-8`Returns full page HTML when contentMode=html, or only the matched element HTML when selector is set.

200

`text/markdown; charset=utf-8`Returns readable article Markdown when contentMode=markdown.

200

`application/json`Returns a structured Readability article JSON payload when contentMode=json.

400

`application/json`Invalid JSON, URL, contentMode, cookies, proxy, region, or random User-Agent parameters.

401

`application/json`Missing or invalid x-api-key.

402

`application/json`Insufficient balance with code INSUFFICIENT\_CREDITS.

422

`application/json`Selector not found, readable content missing, or invalid Worker payload.

429

`application/json`The task was rate limited.

502

`application/json`Proxy, target HTTP, or result-size failure.

503

`application/json`Queue, Worker, managed proxy, or User-Agent resources are unavailable.

504

`application/json`Task, navigation, target, or proxy connection timed out.

500

`application/json`Unclassified task execution failure.

### Request examples

cURL

```bash
curl -sS -X POST "https://api.adscrawl.net/html" \
  -H "content-type: application/json" \
  -H "x-api-key: YOUR_API_KEY" \
  -d '{"url":"https://example.com/article","contentMode":"json","waitUntil":"domcontentloaded","countryCode":"GLOBAL","userAgentMode":"random","userAgentOs":"windows"}'
```

Body JSON

```json
{
  "url": "https://example.com/article",
  "contentMode": "json",
  "waitUntil": "domcontentloaded",
  "viewport": { "width": 1280, "height": 720 },
  "locale": "en-US",
  "countryCode": "GLOBAL",
  "userAgentMode": "random",
  "userAgentOs": "windows"
}
```

### Response examples

200 JSON

```json
{
  "title": "Example Article",
  "byline": "OpenAI",
  "excerpt": "A concise article summary.",
  "siteName": "Example",
  "lang": "en",
  "dir": null,
  "content": "<div><p>Readable body...</p></div>",
  "textContent": "Readable body...",
  "length": 2487,
  "publishedTime": null
}
```

200 Markdown

```markdown
# Example Article

Readable body...

- key point one
- key point two
```

422 JSON

```json
{
  "error": "Readable article content was not found",
  "code": "READABILITY_CONTENT_NOT_FOUND"
}
```

400 Proxy JSON

```json
{
  "error": "countryCode is not supported",
  "code": "INVALID_COUNTRY_CODE"
}
```

503 Proxy JSON

```json
{
  "error": "Dynamic country/region routing is unavailable",
  "code": "DYNAMIC_PROXY_NOT_CONFIGURED"
}
```

GET`/spa-extract/templates`

## List SPA Extraction Templates

### Request

#### Request headers

| Field | Type | Description |
| --- | --- | --- |
| `x-api-key`Required | `string` | Full API key created in the dashboard. |

### Responses

200

`application/json`Returns active templates with their inputs, defaults, and output fields.

401

`application/json`Missing or invalid x-api-key.

500

`application/json`Failed to load the template catalog.

### Request examples

cURL

```bash
curl -sS "https://api.adscrawl.net/spa-extract/templates" \
  -H "x-api-key: YOUR_API_KEY"
```

### Response examples

200 JSON

```json
{
  "templates": [
    {
      "id": "similarweb-overview",
      "name": "SimilarWeb Website Overview",
      "description": "Get public website traffic, engagement, and ranking metrics from SimilarWeb.",
      "urlPattern": "^https://www\\.similarweb\\.com/website/[^/?#]+/?(?:[?#].*)?$",
      "exampleUrl": "https://www.similarweb.com/website/dolphin-anty.com/#overview",
      "version": 5,
      "updatedAt": "2026-07-18T14:07:33.000Z",
      "outputFields": [
        "avgVisitDuration",
        "avgVisitDurationSeconds",
        "bounceRate",
        "categoryRank",
        "countryRank",
        "globalRank",
        "pagesPerVisit",
        "period",
        "targetDomain",
        "totalVisits",
        "totalVisitsDisplay"
      ],
      "waitUntil": "domcontentloaded",
      "input": {
        "type": "domain",
        "example": "dolphin-anty.com",
        "urlTemplate": "https://www.similarweb.com/website/{value}/#overview"
      },
      "catalog": {
        "category": "analytics",
        "featured": true,
        "keywords": ["traffic", "rank", "website analytics", "流量", "排名", "网站分析"]
      }
    }
  ]
}
```

POST`/spa-extract`

## SPA Page Data Extraction

### Request

#### Request headers

| Field | Type | Description |
| --- | --- | --- |
| `x-api-key`Required | `string` | Full API key created in the dashboard. |
| `content-type`Required | `application/json` | Request body must be JSON. |

#### Request body

| Field | Type | Description |
| --- | --- | --- |
| `url` | `string` | Full HTTP(S) URL for the target SPA using port 80 or 443. Google Trends recommends keyword. For legacy clients, url may be a plain keyword string or a Trends explore URL only when the keyword field is completely omitted; only the q query parameter is read from an explore URL. Explicit keyword:null, a number, or an empty string returns INVALID\_KEYWORD and never falls back to url. When keyword and url are both supplied, url must be a canonical Trends explore URL. SimilarWeb requires a full page URL, not a bare domain. |
| `keyword` | `string` | Recommended for google-trends-explore. Supply 1-5 unique, non-empty comma-separated keywords, each at most 100 Unicode characters. The server builds the canonical Trends URL. Legacy url input is used only when this field is completely omitted; explicit null, numeric, or empty-string values return INVALID\_KEYWORD. If url is also supplied, it must be a canonical Trends explore URL. |
| `mode` | `"inspect" | "extract"` | inspect returns page inspection data; extract returns fields or template output. |
| `template` | `string` | Optional site template id. When the page listing is unavailable, chrome-web-store-app-info attempts a CRX2/CRX3 manifest fallback within the same 60-second total budget. The CRX sub-stage is capped at 20 seconds and reserves at least 15 seconds for the Worker. Only signature-verified CRX2/CRX3 packages whose developer-public-key-derived CRX ID matches the requested extension ID are accepted; bare ZIP files are rejected. HTTP/HTTPS proxies fetch the CRX through the API, while SOCKS proxies continue through the Worker. The fallback does not introduce a new public error code. |
| `parameters` | `object` | Parameters declared by the selected template. |
| `waitUntil` | `"load" | "domcontentloaded" | "networkidle"` | Custom extraction defaults to domcontentloaded. Site templates use the request value first, then the template value, then domcontentloaded.
`domcontentloaded`

Waits for DOMContentLoaded. HTML is parsed without waiting for secondary resources such as images; recommended for HTML extraction.

`load`

Waits for window.load after the page and dependent resources such as images and stylesheets finish loading.

`networkidle`

Waits until there are no network connections for at least 500 ms. Long polling, analytics, or lazy-loaded resources may cause a timeout.

 |
| `waitFor` | `waitFor` | Wait for a selector or text to appear. |
| `actions` | `actions[]` | Optional page actions such as click, fill, scroll, or wait. |
| `fields` | `Record<string, field>` | DOM or network fields to extract in extract mode. |
| `schema` | `Record<string, string | field> | { properties: Record<string, string | field> }` | Legacy selector-schema alias used only when fields is absent; string values are treated as DOM selectors. It does not validate response JSON and does not return schemaValid/schemaErrors. |
| `timeoutMs` | `number` | Must be positive and no greater than 3,600,000; other values use the server default. |
| `viewport` | `{ width: number; height: number }` | Viewport size. |
| `locale` | `string` | Browser locale; may follow trusted proxy metadata when omitted. |
| `timezoneId` | `string` | IANA timezone; may follow trusted proxy metadata when omitted. |
| `geolocation` | `{ latitude: number; longitude: number }` | Optional geolocation coordinates. |
| `proxy` | `proxy` | Custom proxy; cannot be combined with countryCode. |
| `countryCode` | `string` | Managed proxy region. GLOBAL selects a dynamic exit from 15 popular regions. A two-letter region code prefers a trusted proxy with dynamic fallback. Omitted values use a random trusted proxy. Cannot be combined with custom proxy. |
| `userAgentMode` | `"custom" | "random"` | Set to random to let the server select from the User-Agent library; requests without a User-Agent default to random. |
| `userAgentOs` | `"windows" | "macos"` | Operating system used by random mode. Defaults to windows. |
| `fingerprint` | `fingerprint` | Browser fingerprint settings. When omitted, every signal defaults to random while keeping the OS, GPU, CPU, memory, fonts, and device signals coherent. |
| `userAgent` | `string` | Override the default User-Agent. |
| `cookies` | `cookies[]` | Preloaded cookies. google-trends-explore ignores omitted, null, or empty-array values; any non-empty or malformed cookies return 400 INVALID\_TRENDS\_COOKIES. |

### Responses

200

`application/json`Returns inspection results or extracted structured data. Successful Google Trends responses also include top-level source (sunbrowser/cache), cached, stale, collectedAt (RFC3339Nano), and attempts. The Chrome Web Store manifest fallback returns the same fields as page-listing extraction and sets categoryZh=非公开 and categoryEn=Unlisted. The complete CRX response is capped at 32 MiB; the ZIP central directory is capped at 8 MiB; ZIP archives are capped at 8,192 entries; selected manifest.json plus \_locales/\*/messages.json metadata is capped at 128 files and 8 MiB uncompressed in total; manifest.json and each locale messages.json are individually capped at 1 MiB; the largest manifest-selected icon is capped at 2 MiB uncompressed; and the final result is capped at 4 MiB. Icons are accepted only from the signature-verified, extension-ID-matched CRX and only as PNG, JPEG, WebP, or GIF; SVG is rejected.

400

`application/json`Invalid URL, Google Trends keyword, proxy, region, or random User-Agent parameters. Non-empty or malformed cookies on a Google Trends request return INVALID\_TRENDS\_COOKIES; explicit keyword:null, numeric, or empty-string values return INVALID\_KEYWORD.

401

`application/json`Missing or invalid x-api-key.

402

`application/json`Insufficient balance with code INSUFFICIENT\_CREDITS.

404

`application/json`When the Chrome Web Store page listing is unavailable and the CRX manifest fallback cannot recover it, the existing CHROME\_WEB\_STORE\_LISTING\_UNAVAILABLE code remains in use; the fallback adds no new public error code.

422

`application/json`Required fields are missing, template/field configuration is invalid, or the Worker rejected the task.

429

`application/json`Task rate limit or a Google Trends upstream 429.

502

`application/json`Proxy/target failure, invalid Trends data, or exhaustion of all candidate identities.

503

`application/json`Queue, Worker, or managed resources are unavailable, or the Trends identity plan expired.

504

`application/json`Task, navigation, or proxy timeout, or the Trends response was not observed.

500

`application/json`Unclassified task execution failure.

### Request examples

cURL

```bash
curl -sS -X POST "https://api.adscrawl.net/spa-extract" \
  -H "content-type: application/json" \
  -H "x-api-key: YOUR_API_KEY" \
  -d '{"url":"https://example.com/dashboard","mode":"extract","waitUntil":"domcontentloaded","waitFor":{"selector":"h1","timeoutMs":15000},"fields":{"title":{"source":"dom","selector":"h1","parse":"string"}},"countryCode":"GLOBAL","userAgentMode":"random","userAgentOs":"windows"}'
```

Body JSON

```json
{
  "url": "https://example.com/dashboard",
  "mode": "extract",
  "waitUntil": "domcontentloaded",
  "waitFor": { "selector": "h1", "timeoutMs": 15000 },
  "fields": {
    "title": { "source": "dom", "selector": "h1", "parse": "string" }
  },
  "countryCode": "GLOBAL",
  "userAgentMode": "random",
  "userAgentOs": "windows"
}
```

SimilarWeb

```json
{
  "template": "similarweb-overview",
  "url": "https://www.similarweb.com/website/dolphin-anty.com/#overview",
  "countryCode": "GLOBAL"
}
```

Google Trends

```json
{
  "template": "google-trends-explore",
  "keyword": "adspower,playwright"
}
```

Google Trends Legacy

```json
{
  "template": "google-trends-explore",
  "url": "https://trends.google.com/trends/explore?q=adspower%2Cplaywright"
}
```

Chrome Web Store

```json
{
  "template": "chrome-web-store-app-info",
  "url": "https://chromewebstore.google.com/detail/example-extension/aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
  "countryCode": "GLOBAL"
}
```

### Response examples

200 JSON

```json
{
  "mode": "extract",
  "page": {
    "url": "https://example.com/dashboard",
    "title": "Example Dashboard"
  },
  "data": {
    "title": "Example Dashboard"
  },
  "missingFields": []
}
```

200 Google Trends

```json
{
  "mode": "extract",
  "page": {
    "url": "https://trends.google.com/trends/explore?q=adspower%2Cplaywright",
    "title": "Google Trends"
  },
  "data": {
    "averages": [
      { "query": "adspower", "extractedValue": 42 },
      { "query": "playwright", "extractedValue": 71 }
    ],
    "interestOverTime": []
  },
  "missingFields": [],
  "source": "sunbrowser",
  "cached": false,
  "stale": false,
  "collectedAt": "2026-08-03T02:04:05.123456789Z",
  "attempts": 2
}
```

200 Inspect JSON

```json
{
  "mode": "inspect",
  "page": {
    "url": "https://example.com/dashboard",
    "title": "Example Dashboard"
  },
  "candidates": {
    "dom": { "metrics": [], "tables": [] },
    "network": []
  },
  "suggestedPlan": {
    "fields": {},
    "schema": {
      "type": "object",
      "properties": {},
      "additionalProperties": false
    }
  }
}
```