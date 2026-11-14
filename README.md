# UCP agent profile

A [Universal Commerce Protocol](https://ucp.dev) agent profile for a personal
back-in-stock notifier. It polls one product page and sends a push notification
when that product becomes purchasable. Nothing else.

The profile is served at

**<https://cdn.jsdelivr.net/gh/chajulal-creator/ucp-agent-profile@main/agent.json>**

and that is what this client passes as `meta.ucp-agent.profile` on UCP
requests. Not the GitHub Pages URL — see below.

## What it declares, and what it deliberately does not

| capability | declared | why |
|---|---|---|
| `dev.ucp.shopping.catalog.lookup` | yes | reads one product's availability |
| `dev.ucp.shopping.catalog.search` | yes | resolves a product handle to an id |
| `dev.ucp.shopping.cart` | **no** | it never builds a cart |
| `dev.ucp.shopping.checkout` | **no** | it never checks out |
| `dev.ucp.shopping.order` | **no** | it never places or reads orders |
| `payment_handlers` | **empty** | it never handles payment |

This is a read-only client. It declares the two catalog capabilities it
actually uses and advertises no payment handler, so capability negotiation
cannot hand it a checkout it has no code to perform.

## Why it exists

The storefront refuses roughly 96% of automated reads of a product page with
`429 local_rate_limited`. Measured over 10.3 hours: 610 requests, 23 reads —
3.8% per request, independent of pace and of the client's User-Agent, because
the throttle is per-path-class. The store publishes a UCP service for agents
instead, which is the supported way to ask it a question, and that service
requires a hosted agent profile. This is that file.

## Hosting requirements it has to meet

Per the UCP spec a profile must be served over HTTPS, with no redirect, and
with `Cache-Control: public` and `max-age` of at least 60 seconds. Measured,
for this exact file:

| host | status | content-type | cache-control | accepted? |
|---|---|---|---|---|
| `raw.githubusercontent.com` | 200 | `text/plain` | — | no, wrong type |
| `chajulal-creator.github.io` (Pages) | 200 | `application/json` | `max-age=600` | **no** |
| `cdn.jsdelivr.net/gh/...@main` | 200 | `application/json` | `public, max-age=604800` | **yes** |

**GitHub Pages cannot host a UCP agent profile.** It sends `max-age=600`
without the `public` directive, and Pages does not let you set response
headers, so there is no fix on that side. The validator's verdict is specific
and worth recording for anyone else hitting it:

```
-32001 profile_malformed
"Unable to fetch agent profile: Invalid cache control"
```

jsDelivr serves the same file out of this repo with `public, max-age=604800`
and the right content type, so the profile lives behind that CDN. Pages stays
enabled only because it costs nothing; nothing points at it.

One consequence: jsDelivr caches the `@main` ref hard. An edit here can take
a while to reach the CDN, so treat this file as slow-moving, or purge it.

