# Source: https://www.adscrawl.net/docs/remote-cdp

On this page

Use x-api-key to create, list, and delete sessions. The data token embedded in cdpBaseUrl protects discovery and CDP WebSockets; live control uses a single-use controlToken valid for 30 seconds.

POST`/cdp/sessions`

## Create Dedicated CDP Session

### Request

#### Request headers

| Field | Type | Description |
| --- | --- | --- |
| `x-api-key`Required | `string` | Full API key created in the dashboard. |
| `content-type`Required | `application/json` | Request body must be JSON. |

#### Request body

| Field | Type | Description |
| --- | --- | --- |
| `idleTimeoutMs` | `number` | Idle timeout. Values above the server cap are clamped. |
| `maxSessionMs` | `number` | Max session lifetime. Values above the server cap are clamped. |
| `browserSettings` | `browserSettings` | Recommended browser settings object. Legacy calls may place these fields directly at the request-body top level. |

### Responses

201

`application/json`Returns only sessionId, expiresAt, and cdpBaseUrl with its data token. Pass cdpBaseUrl directly to Playwright connectOverCDP; fetch discovery separately when needed.

400

`application/json`Invalid managed region, custom proxy, or random User-Agent parameters.

401

`application/json`Missing or invalid x-api-key.

429

`application/json`The API key already reached its CDP session limit.

502

`application/json`The Worker rejected the payload, returned an oversized result, or failed to start the browser session.

503

`application/json`Total capacity, queueing, or the Worker is unavailable.

504

`application/json`Queueing or browser session startup timed out.

500

`application/json`Unclassified internal error.

### Request examples

cURL

```bash
curl -sS -X POST "https://api.adscrawl.net/cdp/sessions" \
  -H "content-type: application/json" \
  -H "x-api-key: YOUR_API_KEY" \
  -d '{"idleTimeoutMs":600000,"maxSessionMs":3600000,"browserSettings":{"viewport":{"width":1440,"height":900},"countryCode":"GLOBAL","userAgentMode":"random","userAgentOs":"windows"}}'
```

Body JSON

```json
{
  "idleTimeoutMs": 600000,
  "maxSessionMs": 3600000,
  "browserSettings": {
    "viewport": { "width": 1440, "height": 900 },
    "countryCode": "GLOBAL",
    "userAgentMode": "random",
    "userAgentOs": "windows"
  }
}
```

### Response examples

201 JSON

```json
{
  "sessionId": "6c3f7d14-7fe4-4c8e-9f1b-0b6d6f2fa2ef",
  "expiresAt": "2026-04-21T10:30:00.000Z",
  "cdpBaseUrl": "https://api.adscrawl.net/cdp/sessions/6c3f7d14-7fe4-4c8e-9f1b-0b6d6f2fa2ef?token=<data-token>"
}
```

429 JSON

```json
{
  "error": "CDP sessions per API key limit reached"
}
```

GET`/cdp/sessions`

## List CDP Sessions

### Request

#### Request headers

| Field | Type | Description |
| --- | --- | --- |
| `x-api-key`Required | `string` | Full API key created in the dashboard. |

### Responses

200

`application/json`Returns an array of sessions for the current key.

401

`application/json`Missing or invalid x-api-key.

500

`application/json`Failed to read sessions or their data tokens.

### Request examples

cURL

```bash
curl -sS "https://api.adscrawl.net/cdp/sessions" \
  -H "x-api-key: YOUR_API_KEY"
```

### Response examples

200 JSON

```json
{
  "ok": true,
  "data": [
    {
      "sessionId": "6c3f7d14-7fe4-4c8e-9f1b-0b6d6f2fa2ef",
      "expiresAt": "2026-04-21T10:30:00.000Z",
      "cdpBaseUrl": "https://api.adscrawl.net/cdp/sessions/6c3f7d14-7fe4-4c8e-9f1b-0b6d6f2fa2ef?token=<data-token>"
    }
  ]
}
```

DELETE`/cdp/sessions/:sessionId`

## Delete CDP Session

### Request

#### Request headers

| Field | Type | Description |
| --- | --- | --- |
| `x-api-key`Required | `string` | Full API key created in the dashboard. |

#### Path parameters

| Field | Type | Description |
| --- | --- | --- |
| `sessionId`Required | `string` | Session id to terminate. |

### Responses

200

`application/json`Deleted successfully.

401

`application/json`Missing or invalid x-api-key.

403

`application/json`The session does not belong to the current key.

404

`application/json`Session id does not exist.

409

`application/json`The session is already stopping.

410

`application/json`The session expired.

503

`application/json`The assigned Worker is unavailable.

### Request examples

cURL

```bash
curl -sS -X DELETE "https://api.adscrawl.net/cdp/sessions/SESSION_ID" \
  -H "x-api-key: YOUR_API_KEY"
```

### Response examples

200 JSON

```json
{
  "ok": true
}
```

404 JSON

```json
{
  "error": "CDP session not found"
}
```

GET`/cdp/sessions/:sessionId/json/version`

## Read CDP Discovery

### Request

#### Path parameters

| Field | Type | Description |
| --- | --- | --- |
| `sessionId`Required | `string` | Session id returned by create or list. |

#### Query parameters

| Field | Type | Description |
| --- | --- | --- |
| `token`Required | `string` | Data token already embedded in cdpBaseUrl. Do not replace it with x-api-key. |

### Responses

200

`application/json`Returns Chrome discovery with webSocketDebuggerUrl rewritten to the public session-scoped path.

401

`application/json`The data token is missing or invalid.

404 / 409 / 410

`application/json`The session is missing, stopping, or expired.

502

`application/json`Failed to fetch upstream CDP discovery.

503

`application/json`The session Worker is unavailable.

### Request examples

cURL

```bash
curl -sS "https://api.adscrawl.net/cdp/sessions/SESSION_ID/json/version?token=DATA_TOKEN"
```

### Response examples

200 JSON

```json
{
  "Browser": "Chrome/136.0.0.0",
  "Protocol-Version": "1.3",
  "User-Agent": "Mozilla/5.0 ...",
  "V8-Version": "13.6.233.8",
  "WebKit-Version": "537.36 (@revision)",
  "webSocketDebuggerUrl": "wss://api.adscrawl.net/cdp/sessions/SESSION_ID/devtools/browser/BROWSER_ID?token=<data-token>"
}
```

WS`/cdp/sessions/:sessionId/devtools/browser/:browserId`

## Connect to the CDP Protocol WebSocket

### Request

#### Path parameters

| Field | Type | Description |
| --- | --- | --- |
| `sessionId`Required | `string` | Current CDP session id. |
| `browserId`Required | `string` | Browser target id from discovery. Prefer the returned webSocketDebuggerUrl instead of constructing it manually. |

#### Query parameters

| Field | Type | Description |
| --- | --- | --- |
| `token`Required | `string` | The same data token used for discovery. |

### Responses

101

`WebSocket`Protocol upgrade succeeded; Chrome DevTools Protocol messages are proxied bidirectionally.

401

`application/json`The data token is missing or invalid.

404 / 409 / 410

`application/json`The session is missing, stopping, or expired.

502

`application/json`The Worker CDP connection failed before the 101 upgrade.

503

`application/json`The session Worker is unavailable.

### Request examples

JavaScript

```javascript
const discovery = await fetch(
  "https://api.adscrawl.net/cdp/sessions/SESSION_ID/json/version?token=DATA_TOKEN",
).then((response) => response.json());

const socket = new WebSocket(discovery.webSocketDebuggerUrl);
```

### Response examples

101 Switching Protocols

```text
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
```

POST`/cdp/live-token`

## Issue a Live Control Token

### Request

#### Request headers

| Field | Type | Description |
| --- | --- | --- |
| `x-api-key`Required | `string` | Must own the target session. |
| `content-type`Required | `application/json` | Request body must be JSON. |

#### Request body

| Field | Type | Description |
| --- | --- | --- |
| `sessionId`Required | `string` | Active session id to control live. |

### Responses

200

`application/json`Returns controlUrl containing a single-use controlToken and Unix-millisecond expiresAt. The token expires after 30 seconds.

400

`application/json`sessionId is missing.

401 / 403

`application/json`The API key is invalid or does not own the session.

404 / 409 / 410

`application/json`The session is missing, stopping, or expired.

503

`application/json`The session Worker is unavailable.

500

`application/json`Failed to issue or store the control token.

### Request examples

cURL

```bash
curl -sS -X POST "https://api.adscrawl.net/cdp/live-token" \
  -H "content-type: application/json" \
  -H "x-api-key: YOUR_API_KEY" \
  -d '{"sessionId":"SESSION_ID"}'
```

### Response examples

200 JSON

```json
{
  "ok": true,
  "controlUrl": "wss://api.adscrawl.net/cdp/live/SESSION_ID?controlToken=<single-use-token>",
  "expiresAt": 1785726630000
}
```

WS`/cdp/live/:sessionId`

## Connect to the Live Control WebSocket

### Request

#### Path parameters

| Field | Type | Description |
| --- | --- | --- |
| `sessionId`Required | `string` | Session id in controlUrl. |

#### Query parameters

| Field | Type | Description |
| --- | --- | --- |
| `controlToken`Required | `string` | Issued by /cdp/live-token and consumable only once within 30 seconds. |

### Responses

101

`WebSocket`The server connects to the Worker before upgrading the client WebSocket.

401

`application/json`controlToken is invalid, expired, already used, or does not match sessionId.

404 / 409 / 410

`application/json`The session is missing, stopping, or expired.

502

`application/json`The CDP Worker could not be reached before the 101 upgrade.

503

`application/json`The token store or session Worker is unavailable.

### Request examples

JavaScript

```javascript
const token = await fetch("https://api.adscrawl.net/cdp/live-token", {
  method: "POST",
  headers: {
    "content-type": "application/json",
    "x-api-key": "YOUR_API_KEY",
  },
  body: JSON.stringify({ sessionId: "SESSION_ID" }),
}).then((response) => response.json());

const socket = new WebSocket(token.controlUrl);
```

### Response examples

101 Switching Protocols

```text
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
```

Close reasons

```text
1000 cdp_upstream_closed
1000 idle_timeout
1000 max_timeout
1011 cdp_upstream_disconnected
1011 cdp_upstream_error
```