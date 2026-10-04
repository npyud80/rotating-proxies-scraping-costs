# Rotating Proxies for Scraping: Per-Request vs Sticky Rotation, Real Costs, and How to Avoid Mid-Session Bans

The first few hundred requests almost always look fine. Then the same IP has pulled the same category page a few thousand times in an hour, and the responses stop being pages: 403s, a Cloudflare challenge, or a 200 with an empty body that parses to nothing. That is usually where the search for rotating proxies for scraping starts — not at the beginning of a project, but in the middle of one that used to work.

Rotation is one of those things that sounds settled until you try to configure it. New IP every request, or hold one for a session? Pay per gigabyte, or pay per IP? Residential, ISP, or datacenter? Get it wrong and you either burn budget on bandwidth you never used, or you build a traffic pattern that is *easier* to fingerprint than a single static IP would have been.

This guide works through those decisions in the order they actually come up, with the current pricing from 9Proxy as a concrete reference point, since its billing model sits at the unusual end of the market: per-IP with unlimited bandwidth rather than per gigabyte.

## What rotation actually changes

A proxy swaps your IP for someone else's. Rotation spreads your requests across many of those someone-else IPs, so no single address looks like a scraper.

What it fixes:

- Per-IP rate limits, which is the main cause of mid-run failures
- IP-level bans, because a burned address can just be dropped
- Geo-restricted content, since the exit country is selectable
- Parallelism, because subtasks can run through separate network paths instead of queuing behind one address

What it does not fix is worth saying out loud: the IP is one half of your identity, and headers, TLS fingerprint, timezone, and locale are the other half. A residential IP from New York attached to a browser reporting `Europe/London` and `en-GB` is a mismatch that most fraud systems detect quickly. Rotation changes the address, not the rest of the story.

And there is a failure mode people miss. Rotating badly — hundreds of low-quality IPs, each from a mismatched region, changing every few requests — is itself a bot signal. The goal is not maximum rotation. It is a traffic pattern that looks like a lot of ordinary visitors.

## Per-request, sticky, or something in between

The rotation strategy matters more than the provider you pick. There are four patterns that cover nearly everything.

**Per-request rotation.** Every request exits a different IP. Right for stateless pages: SERPs, sitemaps, category listings, public product pages. Wrong the moment cookies, a cart, or a login is involved.

**Sticky sessions.** One IP held for a window — 5, 15, 30 minutes — then replaced. This is what paginated flows, multi-step checkouts, and anything behind a login need. A user whose IP changes between page 1 and page 2 of a search result is not a stealthier user; it is an obvious inconsistency.

**Task-level rotation.** One IP or a small pool per batch: one city, one category, one keyword set. Easy to debug, easy to retry, and it keeps parallel workers from sharing identities.

**Geo-targeted rotation.** The exit region is chosen deliberately and kept consistent with the rest of the fingerprint. Collecting US prices means US exits, US timezone, US locale.

9Proxy implements the first two directly. On the GB-based product, the mode is chosen when you generate the proxy, and session behaviour is controlled through the username rather than by managing a list of endpoints:

text
subaccount-country-us-city-newyork          # rotating, new IP per request
subaccount-country-us-sst-15-ssid-bot01     # sticky for 15 minutes


Rotating mode simply omits the session parameters — no `sst`, no `ssid` — so each request draws a fresh IP from the pool. Sticky mode sets `sst` in minutes, and `ssid` gives each parallel worker its own persistent identity even under the same targeting config. Zip-level and ISP-level filters are available too, though narrowing by state *and* city *and* ISP at once shrinks the available pool and raises the odds of reusing an address you already burned.

## The billing question nobody asks until the invoice

Most residential providers charge by traffic. That model has an obvious trap for scrapers: the cost depends on how much data your target sends back, which you do not control. Product images, fonts, and tracking scripts on a single e-commerce page can add 3–8 MB you never wanted, and at $3–7 per GB that adds up faster than the request volume does.

There are two ways out. One is aggressive resource blocking in the browser (`block_resources: ["image", "font", "media"]`), which stretches the same gigabyte allowance several times over. The other is a provider that does not meter traffic at all.

That second route is 9Proxy's position. IP-based packages give you a set number of residential IPs with unlimited bandwidth on each one — scrape 100 pages or 10,000 through the same IP, the price does not move. The IP stays active from a few hours up to roughly 24 hours once forwarded, and unused IPs never expire, so leftover inventory rolls into the next job rather than disappearing at month end.

The trade-off is real and worth stating plainly: on an IP-based plan you are effectively renting identities, and the rotation granularity is coarser. When you want a brand-new exit for every single request across tens of thousands of addresses, the GB-based product is the better fit — you pay only for the data consumed within the package's 180-day window, and you can generate unlimited endpoints in rotating or sticky mode.

👉 [Compare the current IP-based and GB-based packages](https://bit.ly/9-Proxy)

A practical rule that holds across providers: if your workload is heavy on bytes and light on distinct identities, pay per IP. If it is heavy on distinct identities and light on bytes — one request per product page, no images — pay per GB.

## 9Proxy's current packages in full

9Proxy adjusted IP-based and bundle pricing on 1 June 2026; GB-based rates were left unchanged. Everything below reflects the tiers shown now. These are one-time balance purchases, not subscriptions.

### Residential proxies by IPs (unlimited bandwidth per IP)

| Package | Effective price per IP | Total | Purchase |
| --- | --- | --- | --- |
| 100 IPs | $0.24 | $24 | [Get the 100 IP package](https://bit.ly/9-Proxy) |
| 500 IPs | $0.144 | $72 | [Get 500 IPs](https://bit.ly/9-Proxy) |
| 1,000 IPs (+500 bonus) | $0.084 | $126 | [Get 1,000 IPs plus the 500 bonus](https://bit.ly/9-Proxy) |
| 2,500 IPs | $0.084 | $210 | [Get 2,500 IPs](https://bit.ly/9-Proxy) |
| 5,000 IPs | $0.072 | $360 | [Get 5,000 IPs](https://bit.ly/9-Proxy) |
| 15,000 IPs | $0.048 | $720 | [Get 15,000 IPs](https://bit.ly/9-Proxy) |
| 25,000 IPs | $0.035 | $863 | [Get 25,000 IPs](https://bit.ly/9-Proxy) |
| 50,000 IPs | $0.029 | $1,438 | [Get 50,000 IPs](https://bit.ly/9-Proxy) |

High-volume business tiers run $2,300 for 100,000 IPs ($0.023 each), $4,140 for 200,000 IPs ($0.021), and $8,625 for 500,000 IPs ($0.018).

### Residential proxies by bandwidth (180-day validity)

| Package | Effective price per GB | Total | Purchase |
| --- | --- | --- | --- |
| 5 GB | $3.00 | $15 | [Start with 5 GB](https://bit.ly/9-Proxy) |
| 50 GB (+5 bonus) | $2.10 | $105 | [Get 55 GB](https://bit.ly/9-Proxy) |
| 100 GB | $1.50 | $150 | [Get 100 GB](https://bit.ly/9-Proxy) |
| 200 GB | $1.00 | $200 | [Get 200 GB](https://bit.ly/9-Proxy) |
| 1,000 GB | $0.80 | $800 | [Get 1,000 GB](https://bit.ly/9-Proxy) |
| 2,000 GB | $0.75 | $1,500 | [Get 2,000 GB](https://bit.ly/9-Proxy) |

### Enterprise bandwidth (no expiry)

| Package | Effective price per GB | Total | Purchase |
| --- | --- | --- | --- |
| 3,000 GB | $0.72 | $2,160 | [View the 3,000 GB tier](https://bit.ly/9-Proxy) |
| 6,000 GB | $0.70 | $4,200 | [View the 6,000 GB tier](https://bit.ly/9-Proxy) |
| 10,000 GB | $0.68 | $6,800 | [View the 10,000 GB tier](https://bit.ly/9-Proxy) |

Enterprise plans also remove the 180-day clock and add team mode — one owner plus up to five members sharing bandwidth without expiration, with per-member traffic controls and activity logs.

### Bundles (IPs plus bandwidth)

| Bundle | Contents | Price | Purchase |
| --- | --- | --- | --- |
| Starter | 100 IPs + 5 GB | $30 | [Get the Starter bundle](https://bit.ly/9-Proxy) |
| Popular | 1,500 IPs + 50 GB | $180 | [Get the Popular bundle](https://bit.ly/9-Proxy) |
| Pro | 5,000 IPs + 500 GB | $720 (list $860, ~16% off) | [Get the Pro bundle](https://bit.ly/9-Proxy) |

The bundles are the awkward middle of the market in a good way. A crawl that mostly needs stable session IPs, but occasionally needs to fan out across thousands of rotating exits, usually ends up buying two products from two providers. Here it is one purchase, and the traffic portion keeps its 180-day window while the IPs do not expire.

For context on where that sits: entry-level rotating residential traffic from the larger networks typically lands between $3.75 and $8 per GB at low volume, falling to roughly $2–3 per GB once you commit to hundreds of gigabytes. 9Proxy's $1.50 per GB at 100 GB and $0.75 at 2,000 GB are at the cheaper end of that range, with a pool of 20M+ residential IPs across 90+ countries — noticeably smaller than the 100M–400M pools that Oxylabs, Bright Data, and Decodo advertise. Larger pools matter most at high concurrency, where you want to avoid reusing an address you challenged an hour ago. At moderate concurrency, coverage matters more than raw size, and 90+ countries is enough for most price-monitoring and SERP work.

## What setup actually looks like

The two product lines are configured differently, and the difference affects your deployment more than the pricing does.

GB-based proxies run entirely from the dashboard. You pick the target country — and optionally state, city, ZIP, or ISP — choose rotating or sticky, then batch-generate connection strings in `.txt` or `.csv`, with ready-made code samples for common languages. Authentication is either username/password on sub-users or an IP allowlist, so cloud workers, containers, and headless browsers connect without local software. There is no count of endpoints, because endpoints are generated on demand and only the traffic is deducted.

IP-based proxies require the 9Proxy desktop app, which forwards local ports to allocated residential IPs. Rotation there uses the Auto Rotation Proxy feature: you enable it, set a rotation interval and delay, and optionally limit it to specific days and time windows so IPs are not consumed outside working hours.

> Two things 9Proxy documents about that feature are worth knowing before you set it: every automatic rotation consumes a new IP from your balance, and intervals shorter than 30 seconds are discouraged because connections need time to stabilise before switching. Rotating too aggressively on an IP-based plan burns inventory and produces flakier sessions at the same time.

## Test on a small budget, then scale

Almost nobody regrets spending $15 or $24 on a test. People do regret wiring a proxy layer into a production scheduler on the strength of a provider's marketing page.

A test that takes about twenty minutes:

1. Send one request through the proxy to an IP echo endpoint. Confirm the exit country matches what you configured — if you see your own address, the credentials never reached the client.
2. Run 200–500 requests against your real target, not a friendly test site. Record the share of 403s, 429s, and CAPTCHA pages.
3. Run the same volume with brand-new identities versus sticky ones. If the sticky run fails and the rotating run passes, your flow is carrying state and you have been rotating through it.
4. Block images, fonts, and media, then compare bytes consumed. On metered plans this is usually the single largest cost lever you have.
5. Add jitter to your request timing. Fixed intervals from a fixed pool produce a machine-regular pattern regardless of how many IPs you use.

Only after those numbers look sane should concurrency go up.

👉 [Run the test on the 5 GB starter tier](https://bit.ly/9-Proxy)

## Mistakes that cost the most

**Rotating mid-session.** The classic. Login flows, carts, and paginated results break when the IP changes, and the inconsistency itself triggers security lockouts. Hold one IP for the session, then rotate.

**Over-filtering geography.** Combining state, city, and ISP filters on every request shrinks the eligible pool to a sliver. Target by country unless you specifically need city-level accuracy.

**Using datacenter IPs on hostile targets.** Marketplaces and search engines score hosting ASNs as bot traffic at the network level. Residential exits with clean reputation are what actually pass, which is why residential costs more.

**Metered plans with images enabled.** Same request count, six times the bandwidth. This one mistake turns a $150 monthly bill into $900.

**Treating every failure as an IP problem.** A 403 that survives rotation is usually a fingerprint or header issue. Rotating harder will not fix it.

## Where 9Proxy fits, and where it does not

It is a reasonable pick when bandwidth volume is unpredictable, when you need stable session IPs that do not expire at month end, or when your traffic is spread across many pages per request and per-GB billing would punish you for it. The pricing is transparent, the tiers scale to 500,000 IPs, and the GB product needs no local app, which matters for cloud-only pipelines.

It is a weaker fit if your work lives in the single-digit-GB range on a handful of targets — the 5 GB tier at $3 per GB is a test size, not a production plan, and the pool is smaller than what the enterprise networks sell. If you scrape mobile-first platforms, note that the documented lineup here is residential only.

Third-party coverage is thin but consistent. Geekflare's 2026 review treats it as a budget residential option whose appeal rests on per-IP pricing with unlimited bandwidth, and notes the practical split between IP-based plans for predictable access and GB-based plans for high-rotation, low-payload work. That is a fair summary of the trade you are making.

## Quick answers

**Does rotating proxies for scraping break when the site uses Cloudflare?** Not by itself. Rotation keeps you under per-IP rate limits, but Cloudflare also scores TLS fingerprints and headers. Expect to combine rotation with coherent browser settings.

**How often should IPs rotate?** Per request for independent page fetches. Sticky windows of 5–30 minutes for flows that carry state. Anything under 30 seconds on 9Proxy's IP-based app is explicitly discouraged.

**Is per-IP or per-GB billing cheaper?** Per-GB usually wins when each request returns a small payload and needs a new exit. Per-IP with unlimited bandwidth wins when pages are heavy or you cannot predict volume — that is the case 9Proxy is priced around.

**Do unused IPs expire?** On 9Proxy's IP-based packages, no. GB-based packages carry a 180-day window unless you are on an Enterprise plan, which has no expiry.

**Do I need to install anything?** For GB-based proxies, no — the dashboard generates credentials that work from any client or cloud worker. For IP-based proxies, the desktop app handles local port forwarding and rotation.

Pick the rotation mode from the shape of your requests, pick the billing model from the shape of your payloads, then buy the smallest plan that lets you measure both on your real targets. The rest of the setup follows from there.
