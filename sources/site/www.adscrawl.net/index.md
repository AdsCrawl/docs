# Source: https://www.adscrawl.net/

# When scraping gets tough,send in a real {browser}

Connect with Playwright, Puppeteer, or CDP and run jobs in a full browser environment with residential proxy routing—built for dynamic pages, authenticated sessions, and sites that stop basic scrapers.

[Start for free](https://app.adscrawl.net/dashboard/) [Set up with Agent](https://www.adscrawl.net/#setup-title)

HTMLMarkdownJSONScreenshot

https://

![](https://www.adscrawl.net/brand/adspower-mark.png)Powered by **AdsPower****100+** rotating residential proxiesPlaywright · Puppeteer · CDP

![](https://www.adscrawl.net/logo.svg)AdsCrawlDOM ready

![](https://www.adscrawl.net/logo.svg)**AdsCrawl**

LIVE DOM

RenderedSession ready

LIVE PAGE

DOMHTMLJSON

Structured result`JSON`

**1**{**2**  _"url"_: **"https://tiktok.com"**,**3**  _"title"_: **"AdsCrawl browser API"**,**4**  _"format"_: **"html"**,**5**  _"rendered"_: **true****6**}

tiktok.com

## Add browser automation to your app with one {API}

Five APIs cover rendered pages, structured data, screenshots, and live browser control. Pick a workflow and send your first request in minutes.

**HTML**Turn any URL into rendered HTML or clean Markdown.**Scrape**Pull the exact fields you need from dynamic pages.**Screenshots**Capture a full page—or a single element—as a PNG.**CDP**Connect Playwright for multi-step automation and live browser control.**Cloud Browser**Launch an interactive browser with your proxy, cookies, and a reusable profile.

HTML[View API docs](https://www.adscrawl.net/docs/browser-tasks/#html)

cURLCopy

```
curl --fail-with-body -sS \
  -X POST "https://api.adscrawl.net/html" \
  -H "x-api-key: $ADSCRAWL_API_KEY" \
  -H "content-type: application/json" \
  -d '{
    "url": "https://www.adscrawl.net",
    "contentMode": "html",
    "waitUntil": "domcontentloaded"
  }' \
  --output page.html

# Use "contentMode": "markdown" for readable text.
```

## Browse from the region your target site expects.

Route each browser through the region you need, with fewer failures from geo restrictions, IP conflicts, or unavailable proxy nodes.

**One browser. One dedicated network identity.**

Skip the proxy pool and the rotation logic.

_🇺🇸_US_🇧🇷_BR_🇩🇪_DE_🇸🇬_SG_🇯🇵_JP_🇨🇦_CA_🇦🇺_AU

**+**

Loading proxy routes

### Set it up yourself

1. Sign up for an AdsCrawl account.
2. Copy your API key from the dashboard.
3. Choose an API example and send your first request.

[Get your API key](https://app.adscrawl.net/register/)

### Let your coding agent handle it

Recommended

Paste one prompt into your coding agent. It will walk you through sign-up, request access to an API key, and integrate AdsCrawl into your project. You approve access in your browser.

agent setup

`Set up AdsCrawl in this codebase. Use [https://api.adscrawl.net/auth.md](https://api.adscrawl.net/auth.md) to help me create an account and authorize an API key, then follow [https://api.adscrawl.net/docs/agent-quickstart.md](https://api.adscrawl.net/docs/agent-quickstart.md) to complete the integration.`

Copy setup prompt

HTML · Scrape · Screenshots

### Turn any page into usable data

Fetch rendered HTML, pull structured fields from dynamic pages, and capture screenshots for research, monitoring, or AI workflows.

[See how it works](https://www.adscrawl.net/docs/browser-tasks/)

CAPTCHA Solver · 2Captcha

### Solve CAPTCHA challenges automatically

Detect and handle supported challenges such as reCAPTCHA, Turnstile, and AWS WAF in the same browser session—without a manual handoff.

[See how it works](https://www.adscrawl.net/docs/browser-tasks/)

CDP · Playwright

### Automate the entire browser flow

Connect Playwright to a remote browser to navigate, click, fill out forms, and collect results—all in one session.

[See how it works](https://www.adscrawl.net/docs/remote-cdp/)

Cloud Browser

### Pick up where your browser left off

Launch a Cloud Browser with your proxy and cookies, use it in the live viewer, then stop the session and reuse the saved profile later.

[See how it works](https://www.adscrawl.net/docs/cloud-browsers/)

[**AdsCrawl vs. Scrapfly vs. Playwright: Which One Fits Your Stack?**\\ \\ Aug 23, 2026](https://blog.adscrawl.net/adscrawl-vs-scrapfly-vs-playwright) [**AdsCrawl Setup: From API Key to Your First Browser Request**\\ \\ Aug 18, 2026](https://blog.adscrawl.net/adscrawl-setup) [**AdsCrawl vs. Browserless vs. ScrapingBee: Which API Is Right for You?**\\ \\ Aug 23, 2026](https://blog.adscrawl.net/adscrawl-vs-browserless-vs-scrapingbee)

![](https://img.adspower.net/top-browser/70/b32877e4eb8aef4b6108a09758f8a1.png)

### Luis R.

E-commerce

I chose AdsPower to manage my accounts securely, and I've been happy with it so far.

[View original review](https://www.adspower.com/)

![](https://img.adspower.net/top-browser/e6/0bfcaa415ba2f04aba9a3f9da00de4.png)

### Eleanor X

Freelancer

I like the browser fingerprint protection, and the pricing is competitive.

[View original review](https://www.adspower.com/)

![](https://img.adspower.net/top-browser/91/363d355b418dc926ed48d94b7c4542.png)

### Celine Y.

Business Development Manager

I use AdsPower for affiliate marketing, especially its RPA tools for automating repetitive work.

[View original review](https://www.adspower.com/)

[![SOC 2 Type II](https://www.adspower.net/dist/herosoc2.BJht_Qh3.jpg)\\ \\ **SOC 2 Type II**\\ \\ Audited security controls](https://www.adspower.com/) [![ISO 27001](https://www.adspower.net/dist/hero27001.BltaLg8L.jpg)\\ \\ **ISO 27001**\\ \\ Information security management](https://www.adspower.com/) [![ISO 27701](https://www.adspower.net/dist/hero27701.yLgzjAk2.png)\\ \\ **ISO 27701**\\ \\ Privacy information management](https://www.adspower.com/)

Reviews are excerpted or summarized from AdsPower's website.

### How do I get started?

Create an account, copy your API key from the dashboard, and run one of the examples above. Send the key in the x-api-key header, and keep it on your server.

### Which API should I choose?

Use HTML for rendered content, Scrape for structured fields, and Screenshots for page images. Use CDP when you need scripted interaction, or Cloud Browser when you want a reusable profile and a live viewer.

### Can AdsCrawl handle JavaScript-rendered pages?

Yes. AdsCrawl renders the page in a browser before returning the result. Where supported, set waitUntil or wait for a selector so the content you need has time to load.

### Can I get Markdown instead of HTML?

Yes. Set contentMode to markdown on POST /html for clean, readable content. Use html for rendered markup or json for a structured version of the article.

### How do I extract specific fields?

Call POST /spa-extract with mode set to extract, then define the fields you need. DOM fields use CSS selectors, while network fields can read matching JSON responses. Check missingFields in the response to see what was not found.

### Can I capture a full page or just one element?

Yes. POST /screenshot returns a PNG. Set fullPage for the entire page, or pass a selector to capture the first matching element. Save the response as a binary file.

### Can I use Node.js, Python, or Playwright?

Yes. The HTTP APIs work with Node.js fetch, Python requests, cURL, and other HTTP clients. For browser automation, create a CDP session and connect Playwright with the returned webSocketDebuggerUrl.

### What's the difference between CDP and Cloud Browser?

CDP gives your automation a temporary remote browser session. Cloud Browser saves the browser profile and includes a live viewer while it is running. Stop it when you're done; the profile will be ready for your next run.

### Can I use my own proxy?

Yes. APIs that support proxies accept a custom configuration. Every Cloud Browser launch request must include a valid proxy. Put the credentials in the designated fields, not in the proxy server URL.

### Can I use cookies to access signed-in pages?

Yes. Supported tasks and Cloud Browser workflows accept cookies. Use cookies only for accounts you are authorized to access, and check the domain, path, and expiration. For multi-step sign-in flows, use a CDP session.

### Will closing the viewer stop a Cloud Browser?

No. Closing the viewer leaves the Cloud Browser running, so usage charges continue. Call the stop endpoint and confirm that runtime.status is stopped. To release a temporary CDP session, call DELETE /cdp/sessions/:sessionId.

### Why is my request timing out?

The page may be slow, the proxy may be unreachable, or the selector you are waiting for may never appear. Check the error response, verify the URL and proxy, then adjust the wait condition or timeout within the endpoint's documented limits.

### What if I run out of credits or hit a rate limit?

If you get HTTP 402, check your credit balance and plan in the dashboard. For HTTP 429, lower your concurrency and retry with backoff. You can find current plan limits on the pricing page and in your account.

### Can my coding agent set up AdsCrawl for me?

Yes. Copy the setup prompt above into your coding agent. It will guide you through sign-up and integration. Approve the API key request in your browser, then let the agent finish the setup.

Still have questions? [View API docs](https://www.adscrawl.net/docs/) · [Pricing](https://www.adscrawl.net/pricing/)

![AdsCrawl](https://www.adscrawl.net/logo.svg)

## Build on a browser 
you can trust

[Start your first session](https://app.adscrawl.net/register/)