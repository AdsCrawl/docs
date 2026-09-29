# Source: https://www.adscrawl.net/docs/schemas

On this page

## `cloudBrowser.runtime`

runtime.status is starting, running, stopping, or stopped; starting/stopping still reserve quota. runtimeKind is neko or worker\_cdp. Active sessions include sessionId and expiresAt. Only running may include connectUrl: open it directly for the instance's interactive browser, signed in as the profile owner. neko points to /cloud-browser-runtime/{sessionId}/ on the API origin and omits cdpBaseUrl. Fixed usr/pwd parameters are public viewer protocol values; authorization still uses the session cookie, without read-only cast mode. worker\_cdp uses CDP Studio and may return cdpBaseUrl with a temporary token. Use returned URLs; never construct them or add API keys, cookies, or proxy credentials. If a URL is absent, query the actual status first.

| Field | Type | Description |
| --- | --- | --- |
| `runtimeKind` | `"neko" | "worker_cdp"` | Runtime kind. |
| `status`Required | `"starting" | "running" | "stopping" | "stopped"` | runtime.status is starting, running, stopping, or stopped; starting/stopping still reserve quota. runtimeKind is neko or worker\_cdp. Active sessions include sessionId and expiresAt. Only running may include connectUrl: open it directly for the instance's interactive browser, signed in as the profile owner. neko points to /cloud-browser-runtime/{sessionId}/ on the API origin and omits cdpBaseUrl. Fixed usr/pwd parameters are public viewer protocol values; authorization still uses the session cookie, without read-only cast mode. worker\_cdp uses CDP Studio and may return cdpBaseUrl with a temporary token. Use returned URLs; never construct them or add API keys, cookies, or proxy credentials. If a URL is absent, query the actual status first. |
| `sessionId` | `string` | Active session ID. |
| `expiresAt` | `string (RFC3339)` | Active session expiry. |
| `connectUrl` | `string` | Optional direct interactive browser URL; requires the profile owner's login session. |
| `cdpBaseUrl` | `string` | Only worker\_cdp may return this. |

## `cloudBrowser.quotas`

limit is the plan's saved-profile allowance; runningLimit is the user's concurrent running allowance; runningCount includes starting, running, and stopping across all of the user's profiles, pages, API keys, and dashboard sessions, excluding temporary /cdp/sessions. users.cloud\_browser\_running\_limit defaults to 1 for NULL or negative values; 0 blocks new starts and a positive value sets the cap. List and each start read it live; lowering it does not stop existing instances. Global capacity still applies, without the temporary CDP per-key concurrency limit.

| Field | Type | Description |
| --- | --- | --- |
| `limit`Required | `integer` | Saved-profile allowance. |
| `runningLimit`Required | `integer` | User concurrent running allowance. |
| `runningCount`Required | `integer` | Total starting/running/stopping sessions. |

## `browserSettings`

Browser settings for CDP sessions. Omitted regions use a random trusted proxy; explicitly use GLOBAL, a region code, or a custom proxy when needed.

| Field | Type | Description |
| --- | --- | --- |
| `viewport` | `{ width: number; height: number }` | Browser window size. CDP sessions convert it to the default window-size. |
| `locale` | `string` | Browser locale such as en-US. |
| `timezoneId` | `string` | IANA timezone id such as Asia/Shanghai. |
| `geolocation` | `{ latitude: number; longitude: number }` | Optional geolocation coordinates. |
| `proxy` | `proxy` | Custom proxy configuration; cannot be combined with countryCode. |
| `countryCode` | `"GLOBAL" | string` | Managed proxy region. GLOBAL selects a popular region; a specific region prefers trusted proxies with dynamic fallback. Omitted values prefer a random trusted proxy. |
| `userAgent` | `string` | Override the default User-Agent. |
| `userAgentMode` | `"custom" | "random"` | Set to random to let the server select from the User-Agent library; requests without a User-Agent default to random. |
| `userAgentOs` | `"windows" | "macos"` | Operating system used by random mode. Defaults to windows. |
| `fingerprint` | `fingerprint` | CDP fingerprint settings. When omitted, canvas and webGlImage default to real; other omitted signals use a coherent randomized profile. |
| `cookies` | `cookies[]` | Cookies injected before the session starts. |

## `fingerprint`

Every field is optional. Browser-task omissions default to random. CDP browserSettings set omitted canvas and webGlImage to real while generating other signals from a coherent randomized profile. When WebGL is real, WebGPU and hardware cannot be random.

| Field | Type | Description |
| --- | --- | --- |
| `webRtc` | `"forward" | "real" | "disabled"` | Forward uses the proxy exit address; legacy random remains accepted as an alias. |
| `webGl` | `"random" | "real"` | WebGL vendor and renderer metadata. |
| `webGpu` | `"random" | "real" | "disabled"` | Random mode follows the WebGL GPU. |
| `webGlImage` | `"random" | "real"` | WebGL image noise. |
| `canvas` | `"random" | "real"` | Canvas noise. |
| `audioContext` | `"random" | "real"` | Audio fingerprint noise. |
| `clientRects` | `"random" | "real"` | Layout measurement noise. |
| `speechVoices` | `"random" | "real"` | OS-matched speech voice list. |
| `fonts` | `"random" | "real"` | OS-matched font list. |
| `hardware` | `"random" | "real"` | Generates CPU thread count and memory as a pair. |
| `doNotTrack` | `"random" | "enabled" | "disabled"` | Do Not Track preference. |

## `proxy`

Optional proxy configuration. Provide either server or protocol + host + port. Username and password must be supplied together.

| Field | Type | Description |
| --- | --- | --- |
| `server` | `string` | Full proxy URL such as http://host:port or socks5://host:port. Credentials must not be embedded, and server cannot be combined with host. |
| `protocol` | `"http" | "socks5"` | Proxy protocol for the split form. |
| `host` | `string` | Proxy host for the split form. |
| `port` | `number | numeric string` | Port from 1 to 65535 for the split form. |
| `username` | `string` | Proxy username. |
| `password` | `string` | Proxy password. |

## `cookies[]`

Cookies written to the browser context before navigation.

| Field | Type | Description |
| --- | --- | --- |
| `name`Required | `string` | Cookie name. |
| `value`Required | `string` | Cookie value. |
| `domain`Required | `string` | Target domain such as .example.com. |
| `path` | `string` | Path, defaults to /. |
| `secure` | `boolean | string | number` | Whether the cookie is sent only over HTTPS. |
| `httpOnly` | `boolean | string | number` | Whether the cookie is inaccessible to client-side JavaScript. |
| `hostOnly` | `boolean | string | number` | Whether the cookie is host-only. |
| `sameSite` | `string` | SameSite attribute. |
| `session` | `boolean | string | number` | Set true for a session cookie. |
| `expirationDate / expires / expiry` | `number` | Unix expiry timestamp in seconds; all three field names are accepted. |

## `waitFor`

Wait for a visible element or text after SPA navigation and actions. selector and text may be combined.

| Field | Type | Description |
| --- | --- | --- |
| `selector` | `string` | Wait for the first matching element to become visible. |
| `text` | `string` | Wait for the first element containing this text to become visible. |
| `timeoutMs` | `number` | Defaults to 15,000 and never exceeds the remaining task timeout. |

## `field`

SPA extract field definition. DOM fields read element content; network fields read the latest matching JSON response URL.

| Field | Type | Description |
| --- | --- | --- |
| `source`Required | `"dom" | "network"` | Field data source. |
| `selector` | `string` | CSS selector for a DOM field. |
| `value` | `"text" | "html" | "attribute"` | DOM read mode. Defaults to text. |
| `attribute` | `string` | Attribute name used when value=attribute. |
| `urlIncludes` | `string` | Substring used to match a response URL for a network field. |
| `path` | `string` | Network JSON path such as $.data.metrics\[0\].value. |
| `multiple` | `boolean` | Whether a DOM field returns all matching elements. |
| `parse` | `"string" | "number" | "integer" | "boolean" | "json"` | Coerce the extracted value to the selected type. |
| `regex` | `string` | Optional regular expression. Capture group 1 wins when present. |
| `required` | `boolean` | Missing required fields return 422 SPA\_REQUIRED\_FIELDS\_MISSING. |

## `actions[]`

Page interactions executed in array order. Each interactive step is capped at 30 seconds.

| Field | Type | Description |
| --- | --- | --- |
| `wait` | `{ type: "wait"; milliseconds: number }` | Pause for 0-30,000ms. |
| `waitForSelector` | `{ type: "waitForSelector"; selector: string; timeoutMs?: number }` | Wait for the first matching element to become visible. |
| `click` | `{ type: "click"; selector: string }` | Click the first matching element. |
| `fill` | `{ type: "fill"; selector: string; value: string }` | Clear and fill the first matching input. |
| `press` | `{ type: "press"; selector: string; key: string }` | Press a key on the first matching element. |
| `scroll` | `{ type: "scroll"; selector?: string; x?: number; y?: number }` | Scroll an element into view or scroll the page by x/y. y defaults to 800. |

## `Google Trends result metadata`

Top-level collection-source and cache metadata returned by successful google-trends-explore responses in addition to the common SPA result.

| Field | Type | Description |
| --- | --- | --- |
| `source`Required | `"sunbrowser" | "cache"` | Whether the result came from a live SunBrowser collection or the server cache. |
| `cached`Required | `boolean` | Whether this response was served from cache. |
| `stale`Required | `boolean` | Whether the returned cached result is stale. |
| `collectedAt`Required | `string (RFC3339Nano)` | The time at which the result was actually collected. |
| `attempts`Required | `non-negative integer` | The collection attempt count reported with the result. Cache HIT responses use 0; STALE responses use the attempts already made when the Worker reported them. |