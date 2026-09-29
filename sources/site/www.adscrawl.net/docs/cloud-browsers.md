# Source: https://www.adscrawl.net/docs/cloud-browsers

On this page

## Create, launch, and retain

Save a profile for later, or create and launch a cloud browser in one request. source distinguishes manual and launch profiles. Both retain settings after stop and support queries, restart, and explicit deletion, separately from temporary /cdp/sessions. Every API key start requires a custom proxy. starting/running/stopping share the user's running allowance. Confirmed stop releases running allowance while saved count stays unchanged.

POST /cloud-browsers/launch creates a profile and waits for running before returning 201. Open runtime.connectUrl for the interactive browser; Location points to the detail API. Closing the viewer does not stop billing. Each launch creates a new profile without an idempotency key: on failure, inspect the returned id and retry stop with a bound; never automatically repeat launch.

## Open the interactive browser

runtime.status is starting, running, stopping, or stopped; starting/stopping still reserve quota. runtimeKind is neko or worker\_cdp. Active sessions include sessionId and expiresAt. Only running may include connectUrl: open it directly for the instance's interactive browser, signed in as the profile owner. neko points to /cloud-browser-runtime/{sessionId}/ on the API origin and omits cdpBaseUrl. Fixed usr/pwd parameters are public viewer protocol values; authorization still uses the session cookie, without read-only cast mode. worker\_cdp uses CDP Studio and may return cdpBaseUrl with a temporary token. Use returned URLs; never construct them or add API keys, cookies, or proxy credentials. If a URL is absent, query the actual status first.

## Authentication and ownership

Use X-API-Key: <API\_KEY> or Authorization: Bearer <SESSION\_JWT> for list, create, launch, detail, start, and stop. Bearer means a login session JWT, not an API key. Session cookies are also supported. Resources belong to the authenticated user. PATCH and DELETE configuration endpoints, viewer access, and join-token remain session-only.

A session-authenticated start also requires apiKeyId for an active, unexpired API key owned by the same user. X-API-Key selects that key automatically; if apiKeyId is also supplied, it must match. Never put API keys or proxy credentials in frontend code, URLs, or logs. Examples use placeholders only.

JSON responses include traceId, also available in the X-Trace-Id response header. Errors carry error and may include code; use the HTTP status and stable code for error handling.

## Proxy rules for API and dashboard

With X-API-Key, every start request must contain a valid top-level proxy object, even if a proxy was saved at creation. Omission, countryCode alone, or source=web cannot replace it. With session authentication (including JWT Bearer), the dashboard can use a valid custom proxy or explicitly select countryCode (a two-letter region, or GLOBAL for a random region). A saved explicit selection may be reused by session starts. Neither entry permits a browser to run without a proxy.

proxy.server must be an http:// or socks5:// URL with an explicit port (1–65535), without embedded credentials, path, query, or fragment. Alternatively use protocol + host + port. Omit both username and password for an unauthenticated proxy, or provide both non-empty values. Empty or malformed proxies are rejected. Send proxy and countryCode separately; passing both returns 400 COUNTRY\_PROXY\_CONFLICT. Start overrides affect this run only and do not update the saved profile.

A managed proxy allocation failure rejects the start; custom proxy failures must not fall back to a direct connection. Fix the proxy or retry after service recovery. Do not remove the proxy to work around an error.

Session JWT: explicit dynamic proxy

```bash
curl --fail-with-body -sS -X POST "https://api.adscrawl.net/cloud-browsers/<CLOUD_BROWSER_ID>/start" \
  -H "Authorization: Bearer <SESSION_JWT>" \
  -H "Content-Type: application/json" \
  -d '{"apiKeyId":"<API_KEY_ID>","countryCode":"GLOBAL"}'
```

## Saved profiles and running limits

GET /cloud-browsers returns limit (saved profile capacity), runningLimit (this user's simultaneous runtime capacity), and runningCount (all starting, running, and stopping cloud browser sessions for the user, across every page and API key). A saved, stopped profile uses no running slot. Cloud browsers share these slots between dashboard and API; temporary Remote CDP sessions are separate.

The per-user running limit defaults to 1. An unset (NULL) or negative override uses 1; 0 prohibits new starts; a positive integer is the cap. Limits are read live without caching. Lowering a limit does not close existing sessions; it blocks later starts until capacity is available. Starting reserves a slot atomically; failed starts and completed stops or cleanup release it. A stopping session still occupies its slot.

Exceeding the running limit returns 409 CLOUD\_BROWSER\_CONCURRENCY\_LIMIT. This differs from reaching the saved profile limit (409 Cloud Browser limit reached). Starting still requires a paid plan and at least 1 credit. Cloud browsers consume 1 credit per started minute from successful start until stop; repeated stop notifications do not charge twice.

Raising the user limit does not raise the per-API-key CDP limit or global runtime capacity. Those can still reject a start with 429 CDP sessions per API key limit reached or 503 CDP session capacity exhausted.

## Create → start → query → stop

Creation only saves a profile and returns its id. Start is separate and waits for runtime confirmation before returning 200 running. Display runtime.status exactly as returned by queries: starting → running → stopping → stopped. A 202 stop means shutdown is pending; poll detail until stopped. Starting an already active profile returns 409; stopping an already stopped profile returns 200 with runtime.status=stopped.

Run the following in bash with curl and jq installed. Replace all placeholders with your own test credentials and reachable proxy. The bounded polling loop waits up to about two minutes; a timeout is not proof of shutdown. Keep the profile id and query or stop it again if needed. JSON examples below are illustrative responses, not a live API execution.

cURL lifecycle (bash + jq)

```bash
set -euo pipefail
export ADSCRAWL_API_KEY="<API_KEY>"
API_BASE="https://api.adscrawl.net"

# 1. Save a profile; this does not start a browser.
created=$(curl --fail-with-body -sS -X POST "$API_BASE/cloud-browsers" \
  -H "X-API-Key: $ADSCRAWL_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"remark":"API example","browserSettings":{"viewport":{"width":1440,"height":900}}}')
CLOUD_BROWSER_ID=$(printf '%s' "$created" | jq -er '.id')

# 2. Send proxy again on EVERY start. Replace placeholder credentials.
curl --fail-with-body -sS -X POST "$API_BASE/cloud-browsers/$CLOUD_BROWSER_ID/start" \
  -H "X-API-Key: $ADSCRAWL_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"proxy":{"server":"http://proxy.example.com:8080","username":"<PROXY_USERNAME>","password":"<PROXY_PASSWORD>"}}'

# 3. Read the actual state, with a bounded wait for running/stopped.
wait_for_state() {
  local expected="$1" state attempt
  for attempt in {1..60}; do
    state=$(curl --fail-with-body -sS "$API_BASE/cloud-browsers/$CLOUD_BROWSER_ID" \
      -H "X-API-Key: $ADSCRAWL_API_KEY" | jq -er '.runtime.status') || return 1
    printf 'runtime.status=%s\n' "$state"
    if [ "$state" = "$expected" ]; then return 0; fi
    case "$state" in
      starting|running|stopping|stopped) ;;
      *) return 1 ;;
    esac
    if [ "$expected" = running ] && [ "$state" != starting ]; then return 1; fi
    sleep 2
  done
  return 1
}
if ! wait_for_state running; then
  printf '%s\n' 'Browser not ready; requesting stop for cleanup.' >&2
fi

# 4. Inspect account-wide limits (not just the current page).
curl --fail-with-body -sS "$API_BASE/cloud-browsers?page=1&pageSize=10" \
  -H "X-API-Key: $ADSCRAWL_API_KEY" | jq '{limit,runningLimit,runningCount}'

# 5. A 202 response means stopping, not stopped. Confirm via detail.
curl --fail-with-body -sS -X POST "$API_BASE/cloud-browsers/$CLOUD_BROWSER_ID/stop" \
  -H "X-API-Key: $ADSCRAWL_API_KEY"
wait_for_state stopped
```

GET`/cloud-browsers`

## List Cloud Browsers and Quotas

### Request

#### Request headers

| Field | Type | Description |
| --- | --- | --- |
| `x-api-key`With API key | `string` | Use a full API key for API calls. These endpoints also accept a dashboard session cookie or Authorization: Bearer <access-token>; choose one authentication method. Resources are scoped to their owner. |

#### Query parameters

| Field | Type | Description |
| --- | --- | --- |
| `page` | `integer` | Defaults to 1, capped at 10000; invalid or non-positive values use the default. |
| `pageSize` | `integer` | Defaults to and is capped at 10; invalid or non-positive values use the default. |

### Responses

200

`application/json`limit is the plan's saved-profile allowance; runningLimit is the user's concurrent running allowance; runningCount includes starting, running, and stopping across all of the user's profiles, pages, API keys, and dashboard sessions, excluding temporary /cdp/sessions. users.cloud\_browser\_running\_limit defaults to 1 for NULL or negative values; 0 blocks new starts and a positive value sets the cap. List and each start read it live; lowering it does not stop existing instances. Global capacity still applies, without the temporary CDP per-key concurrency limit.

200

`application/json`runtime.status is starting, running, stopping, or stopped; starting/stopping still reserve quota. runtimeKind is neko or worker\_cdp. Active sessions include sessionId and expiresAt. Only running may include connectUrl: open it directly for the instance's interactive browser, signed in as the profile owner. neko points to /cloud-browser-runtime/{sessionId}/ on the API origin and omits cdpBaseUrl. Fixed usr/pwd parameters are public viewer protocol values; authorization still uses the session cookie, without read-only cast mode. worker\_cdp uses CDP Studio and may return cdpBaseUrl with a temporary token. Use returned URLs; never construct them or add API keys, cookies, or proxy credentials. If a URL is absent, query the actual status first.

401

`application/json`Authentication is missing, invalid, or expired.

503

`application/json`CDP\_WORKER\_UNAVAILABLE, or Cloud browser runtime is unavailable without a code. A missing original node, changed bootId, or temporary loss of contact does not confirm a stop; the session and quota remain reserved and connection URLs may be omitted.

500

`application/json`Internal error: an internal service failure.

### Request examples

cURL

```bash
curl --fail-with-body --silent --show-error --max-time 65 \
  -X GET 'https://api.adscrawl.net/cloud-browsers?page=1&pageSize=10' \
  -H 'x-api-key: <api-key>'
```

### Response examples

200

```json
{
  "ok": true,
  "data": [
    {
      "id": "<browser-id>",
      "source": "manual",
      "deleteOnStop": false,
      "remark": "work profile",
      "browserSettings": {
        "viewport": {
          "width": 1440,
          "height": 900
        }
      },
      "proxyDisplayIp": null,
      "proxyDisplayRegion": null,
      "lastOpenedAt": null,
      "updatedAt": "2026-09-07T08:00:00.000Z",
      "runtime": {
        "runtimeKind": "neko",
        "status": "stopped"
      }
    }
  ],
  "pagination": {
    "page": 1,
    "pageSize": 10,
    "total": 1,
    "totalPages": 1
  },
  "limit": 10,
  "runningLimit": 1,
  "runningCount": 0
}
```

POST`/cloud-browsers`

## Create a Cloud Browser Profile

### Request

#### Request headers

| Field | Type | Description |
| --- | --- | --- |
| `x-api-key`With API key | `string` | Use a full API key for API calls. These endpoints also accept a dashboard session cookie or Authorization: Bearer <access-token>; choose one authentication method. Resources are scoped to their owner. |
| `content-type`Required | `application/json` | The body must be one JSON object, at most 1 MiB. |

#### Request body

| Field | Type | Description |
| --- | --- | --- |
| `remark` | `string` | Optional remark, at most 255 Unicode characters after trimming. |
| `browserSettings` | `object` | Optional saved settings such as viewport, locale, timezoneId, proxy or countryCode, cookies, and fingerprint. Defaults to {}; if supplied it must be an object. Creation only saves configuration and does not start a runtime. Saving a proxy does not replace the top-level proxy on the next API key start. Preset cookies must be an array; they are stored encrypted and restored for owner reads. |

### Responses

201

`application/json`Returns {ok: true, id}; the profile is stopped and consumes no running quota. Free plans allow one saved profile; see list.limit for the current allowance. The Node.js request example contains the complete lifecycle.

400

`application/json`Empty body, null, arrays, scalars, invalid field types, an oversized remark, Invalid cookies, INVALID\_PROXY, INVALID\_COUNTRY\_CODE, or COUNTRY\_PROXY\_CONFLICT. {} is a valid create request.

401

`application/json`Authentication is missing, invalid, or expired.

409

`application/json`Cloud Browser limit reached: the saved-profile limit is reached, without a code. This differs from running quota errors.

500

`application/json`Internal error: an internal service failure.

### Request examples

cURL

```bash
curl --fail-with-body --silent --show-error --max-time 65 \
  -X POST 'https://api.adscrawl.net/cloud-browsers' \
  -H 'x-api-key: <api-key>' \
  -H 'content-type: application/json' \
  --data '{
  "remark": "work profile",
  "browserSettings": {
    "viewport": {
      "width": 1440,
      "height": 900
    }
  }
}'
```

JSON

```json
{
  "remark": "work profile",
  "browserSettings": {
    "viewport": {
      "width": 1440,
      "height": 900
    }
  }
}
```

Node.js Lifecycle

```javascript
// Node.js 20+, save as .mjs. Replace proxy placeholders before running.
const baseUrl = "https://api.adscrawl.net";
const apiKey = process.env.ADSCRAWL_API_KEY;
if (!apiKey) throw new Error("ADSCRAWL_API_KEY is required");
const proxy = {
  "server": "http://proxy.example.com:8080",
  "username": "<proxy-user>",
  "password": "<proxy-password>"
};

async function request(method, path, body, timeoutMs = 65000) {
  const res = await fetch(baseUrl + path, {
    method,
    headers: { "x-api-key": apiKey, "content-type": "application/json" },
    body: body === undefined ? undefined : JSON.stringify(body),
    signal: AbortSignal.timeout(timeoutMs),
  });
  const data = await res.json();
  if (!res.ok) {
    const error = new Error(data.error || "HTTP " + res.status);
    error.status = res.status;
    error.code = data.code;
    error.id = data.id;
    throw error;
  }
  return data;
}

async function stopAndWait(id) {
  const path = "/cloud-browsers/" + encodeURIComponent(id);
  // At most 30 attempts, 2 seconds apart; each request times out after 10 seconds.
  for (let attempt = 0; attempt < 30; attempt++) {
    try {
      const stopped = await request("POST", path + "/stop", undefined, 10000);
      if (stopped.runtime.status === "stopped") return;
      const current = await request("GET", path, undefined, 10000);
      if (current.runtime.status === "stopped") return;
    } catch (error) {
      if (error.code !== "CDP_SESSION_STARTING" &&
          !(error.status >= 500) && error.name !== "TimeoutError") throw error;
    }
    await new Promise((resolve) => setTimeout(resolve, 2000));
  }
  throw new Error("Stop unconfirmed; quota is still reserved. Retry stop for " + id);
}
const { id } = await request("POST", "/cloud-browsers", {
  "remark": "work profile",
  "browserSettings": {
    "viewport": {
      "width": 1440,
      "height": 900
    }
  }
});
try {
  // Include a valid proxy again on EVERY start, even for an existing profile.
  await request("POST", "/cloud-browsers/" + encodeURIComponent(id) + "/start", { proxy });
  const current = await request("GET", "/cloud-browsers/" + encodeURIComponent(id));
  console.log({ id, source: current.source, status: current.runtime.status, connectUrl: current.runtime.connectUrl });
  // Open connectUrl in a browser signed in as the profile owner before continuing.
  const { limit, runningLimit, runningCount } = await request("GET", "/cloud-browsers");
  console.log({ limit, runningLimit, runningCount });
} finally {
  // Also attempt cleanup after a failed start: a runtime may still need recovery.
  await stopAndWait(id);
}
console.log("Stopped; the saved profile remains", id);
```

### Response examples

201

```json
{
  "ok": true,
  "id": "<browser-id>"
}
```

POST`/cloud-browsers/launch`

## Create and Launch a Cloud Browser

### Request

#### Request headers

| Field | Type | Description |
| --- | --- | --- |
| `x-api-key`With API key | `string` | Use a full API key for API calls. These endpoints also accept a dashboard session cookie or Authorization: Bearer <access-token>; choose one authentication method. Resources are scoped to their owner. |
| `content-type`Required | `application/json` | The body must be one JSON object, at most 1 MiB. |

#### Request body

| Field | Type | Description |
| --- | --- | --- |
| `proxy`Required | `object` | Required on every request with every authentication method. Every API key start requires an explicit, valid top-level proxy. Saved browserSettings.proxy, a previous run's proxy, countryCode, or a forged source cannot replace it. Only http or socks5 are accepted: {server: "http://proxy.example.com:8080"} or {protocol: "socks5", host: "proxy.example.com", port: 1080}. Ports are integers or integer strings from 1-65535. server must include a port, without URL credentials, query, or fragment, and cannot be combined with host. Optional username/password must be supplied together as non-empty strings, outside the URL. Proxy failure must never fall back to a direct connection. |
| `tabs` | `(string | {url: string, active?: boolean})[]` | Defaults to \[\], leaving the runtime's initial page without injecting tabs. Up to 8 HTTP(S) URLs, each at most 16384 bytes, without URL credentials, control characters, or surrounding whitespace. active defaults to false; at most one may be true. The first tab is activated when none is selected. |
| `cookies` | `cookie[]` | Defaults to \[\] with no preset cookies. Up to 10000 entries within the 1 MiB request cap. name/domain are non-empty strings; value is a string defaulting to empty; path defaults to /; secure/httpOnly/session are booleans; expires uses Unix seconds; sameSite is Strict/Lax/None. Existing snapshot compatibility fields are accepted; expired entries are filtered and unknown fields or invalid types rejected. Cookies are stored separately with encryption and are readable by their owner. |
| `fingerprint` | `object` | Defaults to webRtc=forward and random for webGl, webGpu, webGlImage, canvas, audioContext, clientRects, speechVoices, fonts, hardware, and doNotTrack. Partial input fills remaining defaults; allowed modes and combinations match the start endpoint. Optional hardwareConcurrency/deviceMemory are integers 1-64; the server generates the runtime seed. |
| `apiKeyId`With session/Bearer | `string (UUID)` | Session/Bearer callers must select their own active key. API key authentication selects the current key automatically; any supplied id must match. |

### Responses

201

`application/json`Returns {ok: true, id, source: launch, deleteOnStop: false, runtime} and Location: /cloud-browsers/<id> only after running is confirmed. runtime.connectUrl opens the interactive browser directly; Location is the query endpoint. source only identifies origin. Manual stop, expiry, and confirmed cleanup of failed startup retain settings and cookie/tab snapshots. The profile still counts toward saved allowance and can be queried, restarted, or explicitly deleted. Closing the viewer does not stop the browser.

201

`application/json`runtime.status is starting, running, stopping, or stopped; starting/stopping still reserve quota. runtimeKind is neko or worker\_cdp. Active sessions include sessionId and expiresAt. Only running may include connectUrl: open it directly for the instance's interactive browser, signed in as the profile owner. neko points to /cloud-browser-runtime/{sessionId}/ on the API origin and omits cdpBaseUrl. Fixed usr/pwd parameters are public viewer protocol values; authorization still uses the session cookie, without read-only cast mode. worker\_cdp uses CDP Studio and may return cdpBaseUrl with a temporary token. Use returned URLs; never construct them or add API keys, cookies, or proxy credentials. If a URL is absent, query the actual status first.

201

`application/json`Uses the existing paid-plan, saved limit, user runningLimit, and global capacity rules. starting/running/stopping reserve running allowance. Billing is one credit per started minute from successful start to confirmed stop. Confirmed stopped releases running allowance; saved count stays unchanged until explicit DELETE /cloud-browsers/{id}. Requires Cloud Runtime to be enabled.

400

`application/json`PROXY\_REQUIRED, INVALID\_PROXY, INVALID\_TABS, INVALID\_COOKIES, INVALID\_FINGERPRINT\_SETTINGS. Explicit null, unknown fields (including countryCode/source/browserSettings), invalid JSON, and incorrect types are rejected before profile creation.

401

`application/json`Authentication is missing, invalid, or expired.

402

`application/json`PAID\_PLAN\_REQUIRED or INSUFFICIENT\_CREDITS; no profile is created.

403

`application/json`The selected key belongs to another user or differs from the authenticated key; no profile is created.

409

`application/json`Cloud Browser limit reached: saved allowance full, no creation. CLOUD\_BROWSER\_CONCURRENCY\_LIMIT: running allowance full or zero. If a profile was created, the response includes id, source: launch, deleteOnStop: false, deleted: false, and the actual runtime; the profile remains saved.

502/503/504/500

`application/json`Uses start errors and capacity retry metadata. Errors after profile creation include id, runtime, and Location, plus sessionId once reserved. Unconfirmed cleanup remains stopping and reserves quota. Disabled Cloud Runtime returns 503 without Worker dispatch. Startup waits up to 60 seconds; cleanup confirmation may take up to 120 more seconds.

502/503/504/500

`application/json`Each call creates a new profile without an idempotency key; never automatically repeat launch. On an error with id, inspect it and retry the same stop with a bound until stopped. The profile remains with deleted: false; unconfirmed cleanup also reserves running allowance. If the response is lost, inspect the list and quota first; do not assume nothing was created or that it stopped.

### Request examples

cURL

```bash
curl --fail-with-body --silent --show-error --max-time 200 \
  -X POST 'https://api.adscrawl.net/cloud-browsers/launch' \
  -H 'x-api-key: <api-key>' \
  -H 'content-type: application/json' \
  --data '{
  "proxy": {
    "server": "http://proxy.example.com:8080",
    "username": "<proxy-user>",
    "password": "<proxy-password>"
  },
  "tabs": [
    "https://example.com"
  ],
  "cookies": [
    {
      "name": "sid",
      "value": "<cookie-value>",
      "domain": "example.com",
      "path": "/",
      "secure": true
    }
  ],
  "fingerprint": {
    "canvas": "real"
  }
}'
```

JSON

```json
{
  "proxy": {
    "server": "http://proxy.example.com:8080",
    "username": "<proxy-user>",
    "password": "<proxy-password>"
  },
  "tabs": [
    "https://example.com"
  ],
  "cookies": [
    {
      "name": "sid",
      "value": "<cookie-value>",
      "domain": "example.com",
      "path": "/",
      "secure": true
    }
  ],
  "fingerprint": {
    "canvas": "real"
  }
}
```

Minimal

```json
{
  "proxy": {
    "server": "http://proxy.example.com:8080",
    "username": "<proxy-user>",
    "password": "<proxy-password>"
  }
}
```

Node.js Lifecycle

```javascript
// Node.js 20+, save as .mjs. Replace proxy placeholders before running.
const baseUrl = "https://api.adscrawl.net";
const apiKey = process.env.ADSCRAWL_API_KEY;
if (!apiKey) throw new Error("ADSCRAWL_API_KEY is required");
const proxy = {
  "server": "http://proxy.example.com:8080",
  "username": "<proxy-user>",
  "password": "<proxy-password>"
};

async function request(method, path, body, timeoutMs = 65000) {
  const res = await fetch(baseUrl + path, {
    method,
    headers: { "x-api-key": apiKey, "content-type": "application/json" },
    body: body === undefined ? undefined : JSON.stringify(body),
    signal: AbortSignal.timeout(timeoutMs),
  });
  const data = await res.json();
  if (!res.ok) {
    const error = new Error(data.error || "HTTP " + res.status);
    error.status = res.status;
    error.code = data.code;
    error.id = data.id;
    throw error;
  }
  return data;
}

async function stopAndWait(id) {
  const path = "/cloud-browsers/" + encodeURIComponent(id);
  // At most 30 attempts, 2 seconds apart; each request times out after 10 seconds.
  for (let attempt = 0; attempt < 30; attempt++) {
    try {
      const stopped = await request("POST", path + "/stop", undefined, 10000);
      if (stopped.runtime.status === "stopped") return;
      const current = await request("GET", path, undefined, 10000);
      if (current.runtime.status === "stopped") return;
    } catch (error) {
      if (error.code !== "CDP_SESSION_STARTING" &&
          !(error.status >= 500) && error.name !== "TimeoutError") throw error;
    }
    await new Promise((resolve) => setTimeout(resolve, 2000));
  }
  throw new Error("Stop unconfirmed; quota is still reserved. Retry stop for " + id);
}
let id;
try {
  // One call creates AND starts. Never automatically repeat this POST.
  const launched = await request("POST", "/cloud-browsers/launch", {
    proxy,
    tabs: ["https://example.com"],
    // cookies and fingerprint omitted: use their defaults.
  }, 195000);
  id = launched.id;
  const current = await request("GET", "/cloud-browsers/" + encodeURIComponent(id));
  console.log({ id, status: current.runtime.status });
} catch (error) {
  id = id || error.id;
  if (!id) console.error("No profile id received; inspect /cloud-browsers before retrying launch.");
  throw error;
} finally {
  // Errors after profile creation also return id; unconfirmed cleanup reserves quota.
  if (id) await stopAndWait(id);
}
console.log("Stopped; the saved profile remains", id);
```

### Response examples

201

```json
{
  "ok": true,
  "id": "<browser-id>",
  "source": "launch",
  "deleteOnStop": false,
  "runtime": {
    "runtimeKind": "neko",
    "status": "running",
    "sessionId": "<session-id>",
    "expiresAt": "2026-09-07T09:00:00.000Z",
    "connectUrl": "https://api.adscrawl.net/cloud-browser-runtime/<session-id>/?usr=adscrawl&pwd=adscrawl"
  }
}
```

504 stopping

```json
{
  "error": "Cloud browser runtime request timed out",
  "code": "CLOUD_RUNTIME_TIMEOUT",
  "id": "<browser-id>",
  "source": "launch",
  "deleteOnStop": false,
  "deleted": false,
  "runtime": {
    "runtimeKind": "neko",
    "status": "stopping",
    "sessionId": "<session-id>",
    "expiresAt": "2026-09-07T09:00:00.000Z"
  }
}
```

GET`/cloud-browsers/{id}`

## Get Cloud Browser Status

### Request

#### Request headers

| Field | Type | Description |
| --- | --- | --- |
| `x-api-key`With API key | `string` | Use a full API key for API calls. These endpoints also accept a dashboard session cookie or Authorization: Bearer <access-token>; choose one authentication method. Resources are scoped to their owner. |

#### Path parameters

| Field | Type | Description |
| --- | --- | --- |
| `id`Required | `string (UUID)` | The persistent cloud browser id returned by create or list, distinct from the runtime sessionId. |

### Responses

200

`application/json`Returns the profile directly, without a data or ok wrapper: id, remark, browserSettings, nullable proxyDisplayIp/Region and lastOpenedAt, updatedAt, and runtime. Proxy credentials are removed from responses; the owner can read saved cookies.

200

`application/json`runtime.status is starting, running, stopping, or stopped; starting/stopping still reserve quota. runtimeKind is neko or worker\_cdp. Active sessions include sessionId and expiresAt. Only running may include connectUrl: open it directly for the instance's interactive browser, signed in as the profile owner. neko points to /cloud-browser-runtime/{sessionId}/ on the API origin and omits cdpBaseUrl. Fixed usr/pwd parameters are public viewer protocol values; authorization still uses the session cookie, without read-only cast mode. worker\_cdp uses CDP Studio and may return cdpBaseUrl with a temporary token. Use returned URLs; never construct them or add API keys, cookies, or proxy credentials. If a URL is absent, query the actual status first.

200

`application/json`source is manual or launch; launch identifies the create-and-launch API, not a deletion policy. The backend handles legacy deleteOnStop flags and now returns false. Stop preserves configuration, snapshots, and source, still counting toward saved allowance. Previously deleted records are not restored and may return deleted: true, stopped, and empty settings; only actually deleted records disappear from the list.

401

`application/json`Authentication is missing, invalid, or expired.

404

`application/json`The profile does not exist or belongs to another user.

503

`application/json`CDP\_WORKER\_UNAVAILABLE, or Cloud browser runtime is unavailable without a code. A missing original node, changed bootId, or temporary loss of contact does not confirm a stop; the session and quota remain reserved and connection URLs may be omitted.

500

`application/json`Internal error: an internal service failure.

### Request examples

cURL

```bash
curl --fail-with-body --silent --show-error --max-time 65 \
  -X GET 'https://api.adscrawl.net/cloud-browsers/<browser-id>' \
  -H 'x-api-key: <api-key>'
```

### Response examples

stopped

```json
{
  "id": "<browser-id>",
  "source": "manual",
  "deleteOnStop": false,
  "remark": "work profile",
  "browserSettings": {
    "viewport": {
      "width": 1440,
      "height": 900
    }
  },
  "proxyDisplayIp": null,
  "proxyDisplayRegion": null,
  "lastOpenedAt": null,
  "updatedAt": "2026-09-07T08:00:00.000Z",
  "runtime": {
    "runtimeKind": "neko",
    "status": "stopped"
  }
}
```

stopping

```json
{
  "id": "<browser-id>",
  "source": "manual",
  "deleteOnStop": false,
  "remark": "work profile",
  "browserSettings": {
    "viewport": {
      "width": 1440,
      "height": 900
    }
  },
  "proxyDisplayIp": null,
  "proxyDisplayRegion": null,
  "lastOpenedAt": null,
  "updatedAt": "2026-09-07T08:00:00.000Z",
  "runtime": {
    "runtimeKind": "neko",
    "status": "stopping",
    "sessionId": "<session-id>",
    "expiresAt": "2026-09-07T09:00:00.000Z"
  }
}
```

launch stopped

```json
{
  "id": "<browser-id>",
  "source": "launch",
  "deleteOnStop": false,
  "remark": "work profile",
  "browserSettings": {
    "viewport": {
      "width": 1440,
      "height": 900
    }
  },
  "proxyDisplayIp": null,
  "proxyDisplayRegion": null,
  "lastOpenedAt": null,
  "updatedAt": "2026-09-07T08:00:00.000Z",
  "runtime": {
    "runtimeKind": "neko",
    "status": "stopped"
  }
}
```

POST`/cloud-browsers/{id}/start`

## Start a Cloud Browser

### Request

#### Request headers

| Field | Type | Description |
| --- | --- | --- |
| `x-api-key`With API key | `string` | Use a full API key for API calls. These endpoints also accept a dashboard session cookie or Authorization: Bearer <access-token>; choose one authentication method. Resources are scoped to their owner. |
| `content-type`Required | `application/json` | The body must be one JSON object, at most 1 MiB. |

#### Path parameters

| Field | Type | Description |
| --- | --- | --- |
| `id`Required | `string (UUID)` | The persistent cloud browser id returned by create or list, distinct from the runtime sessionId. |

#### Request body

| Field | Type | Description |
| --- | --- | --- |
| `proxy`With API key | `object` | Every API key start requires an explicit, valid top-level proxy. Saved browserSettings.proxy, a previous run's proxy, countryCode, or a forged source cannot replace it. Only http or socks5 are accepted: {server: "http://proxy.example.com:8080"} or {protocol: "socks5", host: "proxy.example.com", port: 1080}. Ports are integers or integer strings from 1-65535. server must include a port, without URL credentials, query, or fragment, and cannot be combined with host. Optional username/password must be supplied together as non-empty strings, outside the URL. Proxy failure must never fall back to a direct connection. |
| `apiKeyId`With session/Bearer | `string (UUID)` | Required with dashboard session/Bearer authentication; select your own active key. API key authentication uses the current key automatically; if apiKeyId is also supplied it must match. |
| `countryCode` | `string (session/Bearer only)` | Dashboard callers may explicitly select GLOBAL or a supported two-letter region such as FR, preferring a trusted proxy in that region with dynamic fallback. An empty string is not a selection. Merged saved settings and overrides must contain a proxy or non-empty countryCode for dashboard starts too. It cannot replace proxy for API key calls; never send both in one request. The selected override clears the opposite saved route only in the runtime snapshot, without updating the profile. |
| `cookies` | `cookie[]` | Optional array override for this run, taking precedence over saved snapshots without updating profile configuration. |
| `fingerprint` | `object` | Optional per-run override merged by field. webRtc: forward|real|disabled (legacy random becomes forward); webGpu: random|real|disabled; doNotTrack: random|enabled|disabled; webGl, webGlImage, canvas, audioContext, clientRects, speechVoices, fonts, hardware: random|real. Legacy hardwareConcurrency/deviceMemory accept integers 1-64. Unknown, invalid, or conflicting combinations return INVALID\_FINGERPRINT\_SETTINGS. |

### Responses

200

`application/json`Waits for startup confirmation and returns {ok: true, runtime} only at running, never a successful starting state. Requires an active paid plan. Billing runs from successful start to stop at one credit per started minute, rounded up. See get for runtime fields; neko omits cdpBaseUrl.

400

`application/json`PROXY\_REQUIRED for a missing top-level proxy; INVALID\_PROXY for an invalid proxy; COUNTRY\_PROXY\_CONFLICT for proxy and countryCode together; INVALID\_COUNTRY\_CODE, INVALID\_FINGERPRINT\_SETTINGS, or Invalid cookies. Invalid body/cookies/fingerprint types, missing dashboard apiKeyId, or a missing/inactive/expired selected key also return 400.

401

`application/json`Authentication is missing, invalid, or expired.

402

`application/json`PAID\_PLAN\_REQUIRED: no active paid plan. INSUFFICIENT\_CREDITS: less than one credit available (includes balance and requiredCredits).

403

`application/json`apiKeyId differs from the authenticated key, or the selected key belongs to another user.

404

`application/json`The profile does not exist or belongs to another user.

409

`application/json`CLOUD\_BROWSER\_CONCURRENCY\_LIMIT: the user allowance is full or zero. Browser is already running (without a code): this profile has an active session. limit is the plan's saved-profile allowance; runningLimit is the user's concurrent running allowance; runningCount includes starting, running, and stopping across all of the user's profiles, pages, API keys, and dashboard sessions, excluding temporary /cdp/sessions. users.cloud\_browser\_running\_limit defaults to 1 for NULL or negative values; 0 blocks new starts and a positive value sets the cap. List and each start read it live; lowering it does not stop existing instances. Global capacity still applies, without the temporary CDP per-key concurrency limit.

502

`application/json`CLOUD\_RUNTIME\_AUTH\_FAILED, CLOUD\_RUNTIME\_NOT\_FOUND, CLOUD\_RUNTIME\_HTTP\_ERROR, CLOUD\_RUNTIME\_INVALID\_RESPONSE, or CLOUD\_RUNTIME\_CONTROLLER\_FAILED: runtime rejection or invalid response.

503

`application/json`CLUSTER\_NO\_CAPACITY: online nodes are full, with nullable nextAvailableAt and retryAfterMs. CDP session capacity exhausted (without a code): global capacity is full. Other codes include DYNAMIC\_PROXY\_NOT\_CONFIGURED, MANAGED\_PROXY\_UNAVAILABLE, CLOUD\_RUNTIME\_CONFIG\_INVALID, and CLOUD\_RUNTIME\_UNREACHABLE. Proxy failure never connects directly; inspect the profile before retrying.

503

`application/json`CDP\_WORKER\_UNAVAILABLE, or Cloud browser runtime is unavailable without a code. A missing original node, changed bootId, or temporary loss of contact does not confirm a stop; the session and quota remain reserved and connection URLs may be omitted.

504

`application/json`CLOUD\_RUNTIME\_TIMEOUT: startup timeout does not prove the runtime is absent. Inspect its status and retry stop when needed; stopping and quota remain until cleanup is confirmed.

500

`application/json`Internal error: an internal service failure.

### Request examples

cURL

```bash
curl --fail-with-body --silent --show-error --max-time 65 \
  -X POST 'https://api.adscrawl.net/cloud-browsers/<browser-id>/start' \
  -H 'x-api-key: <api-key>' \
  -H 'content-type: application/json' \
  --data '{
  "proxy": {
    "server": "http://proxy.example.com:8080",
    "username": "<proxy-user>",
    "password": "<proxy-password>"
  }
}'
```

JSON

```json
{
  "proxy": {
    "server": "http://proxy.example.com:8080",
    "username": "<proxy-user>",
    "password": "<proxy-password>"
  }
}
```

SOCKS5

```json
{
  "proxy": {
    "protocol": "socks5",
    "host": "proxy.example.com",
    "port": 1080,
    "username": "<proxy-user>",
    "password": "<proxy-password>"
  }
}
```

Bearer Region

```bash
curl --fail-with-body --silent --show-error --max-time 65 \
  'https://api.adscrawl.net/cloud-browsers/<browser-id>/start' \
  -H 'Authorization: Bearer <access-token>' \
  -H 'content-type: application/json' \
  --data '{
  "apiKeyId": "<api-key-id>",
  "countryCode": "GLOBAL"
}'
```

### Response examples

200 neko

```json
{
  "ok": true,
  "runtime": {
    "runtimeKind": "neko",
    "status": "running",
    "sessionId": "<session-id>",
    "expiresAt": "2026-09-07T09:00:00.000Z",
    "connectUrl": "https://api.adscrawl.net/cloud-browser-runtime/<session-id>/?usr=adscrawl&pwd=adscrawl"
  }
}
```

400

```json
{
  "error": "API key starts require an explicit proxy in every request",
  "code": "PROXY_REQUIRED"
}
```

409

```json
{
  "error": "Cloud browser running limit reached",
  "code": "CLOUD_BROWSER_CONCURRENCY_LIMIT"
}
```

503

```json
{
  "error": "Cloud browser cluster has no available capacity",
  "code": "CLUSTER_NO_CAPACITY",
  "nextAvailableAt": null,
  "retryAfterMs": null
}
```

POST`/cloud-browsers/{id}/stop`

## Stop a Cloud Browser

### Request

#### Request headers

| Field | Type | Description |
| --- | --- | --- |
| `x-api-key`With API key | `string` | Use a full API key for API calls. These endpoints also accept a dashboard session cookie or Authorization: Bearer <access-token>; choose one authentication method. Resources are scoped to their owner. |

#### Path parameters

| Field | Type | Description |
| --- | --- | --- |
| `id`Required | `string (UUID)` | The persistent cloud browser id returned by create or list, distinct from the runtime sessionId. |

### Responses

200

`application/json`runtime.status=stopped: stop is confirmed or no active session existed, releasing running allowance. Both manual and launch profiles retain settings, snapshots, and source and still reserve saved allowance. Profiles can restart and are removed only by explicit deletion. Repeated stop returns stopped without duplicate billing. source identifies origin and deleteOnStop is false; only previously deleted records return deleted: true. sessionId is omitted when no active session existed.

202

`application/json`runtime.status=stopping: another stop is in progress; completion is unconfirmed and quota remains reserved. Poll GET details with a bound; retry the same stop if it stays stopping, until stopped is confirmed. The create endpoint's Node.js lifecycle includes polling and retries.

401

`application/json`Authentication is missing, invalid, or expired.

404

`application/json`The profile does not exist or belongs to another user.

409

`application/json`CDP\_SESSION\_STARTING: startup is still in progress; wait and retry stop.

503

`application/json`CDP\_WORKER\_UNAVAILABLE, or Cloud browser runtime is unavailable without a code. A missing original node, changed bootId, or temporary loss of contact does not confirm a stop; the session and quota remain reserved and connection URLs may be omitted.

500

`application/json`Internal error: a failed or timed-out stop is not success. The session may remain stopping and reserve quota; inspect it and retry stop after reachability is restored. Losing the original node identity must not redirect deletion to a new node or justify a fabricated stopped state.

### Request examples

cURL

```bash
curl --fail-with-body --silent --show-error --max-time 65 \
  -X POST 'https://api.adscrawl.net/cloud-browsers/<browser-id>/stop' \
  -H 'x-api-key: <api-key>'
```

### Response examples

200 stopped

```json
{
  "ok": true,
  "runtime": {
    "status": "stopped",
    "sessionId": "<session-id>"
  }
}
```

202 stopping

```json
{
  "ok": true,
  "runtime": {
    "status": "stopping",
    "sessionId": "<session-id>"
  }
}
```

409

```json
{
  "error": "CDP session is starting; retry after startup completes",
  "code": "CDP_SESSION_STARTING"
}
```

503

```json
{
  "error": "Cloud browser runtime is unavailable"
}
```

200 launch retained

```json
{
  "ok": true,
  "source": "launch",
  "deleteOnStop": false,
  "runtime": {
    "status": "stopped",
    "sessionId": "<session-id>"
  }
}
```