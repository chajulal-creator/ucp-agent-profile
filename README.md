# UCP agent profile

A [Universal Commerce Protocol](https://ucp.dev) agent profile for a personal
back-in-stock notifier. It polls one product page and sends a push notification
when that product becomes purchasable. Nothing else.

The profile is served at
**<https://chajulal-creator.github.io/ucp-agent-profile/agent.json>** and is
what this client passes as `meta.ucp-agent.profile` on UCP requests.

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

Per the UCP spec, a profile must be served over HTTPS, with no redirect, and
with `Cache-Control: public` and `max-age` of at least 60 seconds. GitHub Pages
serves this file as `application/json` with a public `max-age`, which is why it
is published here rather than from a raw file URL (`raw.githubusercontent.com`
serves `text/plain`, which the validator rejects).
