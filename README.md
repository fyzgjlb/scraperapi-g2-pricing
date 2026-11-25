# Amazon scraping API: what 1,000 product pages really cost, whether you can get full reviews, and which ScraperAPI plan covers your volume

ScraperAPI bills Amazon requests at 5 credits each, so the free plan's 1,000 monthly credits buy 200 product pages, not 1,000. At the other end, the Scaling plan's 5,000,000 credits work out to 1,000,000 Amazon requests for **$475/month**, or $427.50/month on annual billing, which is roughly **$0.48 per 1,000 pages** monthly and $0.43 annual.

Those two conversions decide most of the buying question, and neither is on the pricing page. Below: the credit table, the full plan line-up, an actual monthly bill at three volumes, the Amazon endpoint's request and response shape, and the limits that determine whether this fits your project at all.

> **Quick pick.** Testing a pipeline: the free plan, 1,000 credits/month, 5 concurrent threads, no card. Running 5,000 to 30,000 Amazon pages a day: Scaling at $475/month, or $427.50/month if you pay annually. 👉 [See the current ScraperAPI plans and credit allowances](https://www.scraperapi.com/pricing?fp_ref=coupons)

## What one Amazon page costs in credits

Amazon sits in the e-commerce bucket at **5 credits per request**. A plain page is 1 credit, Google and Bing are 25, LinkedIn is 30. Add-ons stack on top of the base cost:

| Request type | Credits | Notes |
| --- | --- | --- |
| Standard page | 1 | Per request |
| Amazon, Walmart, eBay | 5 | Per request |
| Google / Bing SERP | 25 | Includes subdomains |
| LinkedIn | 30 | Per request |
| Bot-protection bypass | +10 | Cloudflare, DataDome, PerimeterX/HUMAN, Akamai |
| JS rendering (`render=true`) | +10 | Per request |
| Premium proxies (`premium=true`) | +10 | Per request |
| Premium proxies + JS rendering | 25 | Combined, not 20 |
| Screenshot (`screenshot=true`) | +10 | Per request |
| Ultra premium (`ultra_premium=true`) | 30 | Paid plans only |

Two consequences worth internalising before you compare vendors. A "5,000,000 credit" plan is **1,000,000 Amazon requests**, and a pipeline that needs `render=true` on every page turns the same 1,000,000 requests into 15,000,000 credits. Geotargeting, sessions, device type, output format and auto parsing add nothing.

Every response carries its own cost in the `sa-credit-cost` header, and the dashboard has a Domain Cost Estimator plus a `max_cost` cap per scrape. Run 50 pages through your real target URLs before you size a plan, because the estimate you get from the credit table and the number in that header can differ once Amazon serves you a harder page.

## All ScraperAPI plans side by side

| Plan | Credits / month | Price | Concurrent threads | Get it |
| --- | --- | --- | --- | --- |
| Free | 1,000 | $0 | 5 | [Start on the free plan](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Hobby | Shown on pricing page | Shown on pricing page | — | [Check the Hobby tier](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Startup | Shown on pricing page | Shown on pricing page | — | [Check the Startup tier](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Business | Shown on pricing page | Shown on pricing page | — | [Check the Business tier](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Scaling (most popular) | 5,000,000 | $475/mo, or $427.50/mo billed annually ($5,130/yr, saves $570) | 200 | [Open the Scaling plan](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Professional | Shown on pricing page | Shown on pricing page | — | [Check the Professional tier](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | 21,500,000 (+500,000 bonus credits on the current offer) | $1,975/mo, or $1,777.50/mo billed annually ($21,330/yr, saves $2,370) | 500 | [Open the Advanced plan](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | Custom | Custom | Custom | [See enterprise options](https://www.scraperapi.com/pricing?fp_ref=coupons) |

The mid-tier rows are the ones most small teams actually land on, and their allowances move around more than the top two, so confirm the numbers on the page rather than working from a screenshot.

## What actually differs between the tiers

Not "more power". Four concrete things change as you go up.

**Credits per month**, which is the only number that maps directly to how many Amazon pages you can pull. Divide by 5.

**Concurrency.** The free plan allows 5 threads, Scaling 200, Advanced 500. Go over and you get an HTTP 429 telling you to bring concurrency back inside the plan limit. The request is turned away, not queued, and there's no surcharge for the overage. If your code fires parallel requests without a semaphore, you will see 429s long before you run out of credits.

**Geotargeting scope and routing.** Scaling lists global, country-level geotargeting. Advanced adds priority routing. Geotargeting itself doesn't consume credits on any plan.

**What happens when credits run out.** By default scraping pauses until the monthly renewal. Hobby, Startup, Business, Scaling and Professional can switch on auto-upgrade, which bumps you to the next plan the moment you hit the ceiling. Scaling, Professional, Advanced and Enterprise can use pay-as-you-go instead, which continues at a fixed per-credit rate up to a monthly cap you set. If you're running price monitoring where a two-day gap is worse than a bigger bill, that distinction matters more than the sticker price.

There's a **7-day, no-questions-asked refund** on paid plans if the real-world success rate doesn't match what you measured in testing.

## What 5,000 pages a day costs

Credit allowances, converted at 5 credits per Amazon request:

- **5,000 pages/day ≈ 150,000 requests/month ≈ 750,000 credits.** That fits under Scaling's 5,000,000 with room to spare, but you can't buy 750,000 credits à la carte, so you'd be paying the tier that covers it. At Scaling's effective rate the credit burn is about $71/month, which is the number to keep in mind when you look at the smaller tiers.
- **1,000,000 requests/month (~33,000/day) = exactly 5,000,000 credits.** This is Scaling's design point: $475/month, or $427.50/month annual, which is $0.475 versus $0.4275 per 1,000 Amazon pages. The annual commitment saves $570 a year.
- **Same volume with `render=true` on every request = 15,000,000 credits.** Now you're past Scaling and into a bigger tier. If half your pages render and half don't, you're at 10,000,000 and still above Scaling. This is the single most common way an Amazon scraping API bill comes in three times higher than planned.
- **4,300,000 requests/month = 21,500,000 credits** at Advanced's $1,975/month, or $1,777.50 annual. Per thousand pages that's $0.459 monthly, $0.413 annual.

The lesson for anyone sizing a plan: page-count math without a render multiplier is optimistic by up to 3x.

## Inside the Amazon product endpoint

ScraperAPI's structured Amazon product endpoint takes ASINs and returns parsed JSON, and it runs asynchronously with a webhook callback rather than holding a connection open:

python
import requests

url = "https://async.scraperapi.com/structured/amazon/product"
data = {
    "apiKey": "YOUR_KEY",
    "asins": ["B079BLHH67", "B07G98GG51"],
    "country_code": "us",
    "tld": "com",
    "callback": {"type": "webhook", "url": "https://your-app.example/hook"}
}
print(requests.post(url, json=data, headers={"Content-Type": "application/json"}).text)


You get back a job object (`id`, `status: "running"`, `statusUrl`, `asin`, `country_code`, `tld`) and the parsed record arrives at your webhook. A single-ASIN request uses the same endpoint with `asin` instead of `asins`. The `tld` parameter covers 20 marketplaces: `com`, `co.uk`, `ca`, `de`, `es`, `fr`, `it`, `jp`, `in`, `cn`, `sg`, `mx`, `ae`, `br`, `nl`, `au`, `tr`, `sa`, `se`, `pl`.

The parsed output is the product record itself, not a search-results card. A rice cooker listing, for example, comes back with `name`, `brand`, `brand_url`, a `product_information` object holding `asin`, `model_name`, `item_weight`, `capacity`, `voltage` and a `best_sellers_rank` array like `["#14,536 in Kitchen & Dining", "#49 in Rice Cookers"]`, `date_first_available`, plus `full_description` with the A+ content text. For BSR tracking and catalog enrichment, that's the field set you want; for a fast price sweep across search pages, calling the product endpoint per ASIN burns 5 credits a page when a search endpoint would have covered twenty listings at once.

## The three limits that decide the project

These are the questions worth answering before you pay, because none of them live on a pricing page.

**Full review text.** ScraperAPI's Amazon product endpoint returns the product record, review counts included. Whether you get review bodies depends on parameters, and it's worth testing rather than assuming. The industry picture is not encouraging: complete review lists on Amazon have required a logged-in session since late 2024, and one competing Amazon scraper documents that it returns only the top reviews shown on the product page, "usually 8 to 13," plus Amazon's own summary. If your product depends on sentiment analysis over hundreds of reviews per ASIN, budget time to test this first.

**Postal-code pricing.** ScraperAPI's geo controls are `country_code` plus `tld`, and the docs are explicit that scraping amazon.com from Canada to see Canadian shipping information needs both set. That's country-level targeting. A specific ZIP code is a different capability, and it isn't in the documented Amazon parameters. For "what does this cost delivered to 90210" work, you need to confirm support directly. One major competitor states outright that it returns the price a shopper in the site's home country sees and does not support a specified ZIP.

**Async by default, and what that costs you.** Every structured Amazon call is a job with a webhook or a `statusUrl` to poll. That's the right architecture for 100,000 pages a month and a nuisance for a one-off script. Budget for the webhook handler and idempotent writes, or you'll end up polling in a loop and duplicating rows.

> Neither failed-request billing nor the exact pay-as-you-go per-credit rate is stated on the pricing page. Both change your real monthly cost, so get them in writing from support before you commit to an annual plan.

## How the competition bills the same job

Billing units differ enough that per-1,000 comparisons across vendors are usually wrong. This is the honest version:

| Provider | Billing unit | Free allowance | Documented constraint |
| --- | --- | --- | --- |
| ScraperAPI | Credits; Amazon = 5 per request | 1,000 credits/month, no card | 429 when concurrency is exceeded; no queue |
| Oxylabs | Per delivered result, from $0.50 per 1,000 | Free open-source Python tool | Only 4 marketplaces named in the search `domain` parameter |
| Bright Data | Per successful record | 5,000 credits every month, no card | Sync requests return HTTP 202 with a `snapshot_id` after 1 minute |
| ScrapingBee | Credits; 5 per product, 10 per search page | 1,000 credits, no card | Review text "depends on the parameters you set" |
| Apify | Per 1,000 results, listing and details priced separately | — | Top reviews only (8 to 13), no ZIP targeting, buy-box seller only |

Oxylabs quotes "starting from $0.50 per 1,000 results" and charges only for successfully delivered results, which is a different risk model from paying 5 credits per attempt. Bright Data's 5,000 free credits renew monthly rather than once, which makes it a genuine testing allowance. ScrapingBee's 10 credits per search page compounds fast on category sweeps: 20 pages of results is 200 credits, and with 1,000 free credits you get five such sweeps.

ScraperAPI's own position is the middle of that field: cheaper per Amazon page than Oxylabs' published starting rate on the upper tiers, more expensive than the free allowances elsewhere, and with the simplest single credit currency once you internalise the 5x multiplier.

## Which plan to pick

Under 200 Amazon pages a month: stay on the free plan indefinitely. It doesn't expire and doesn't need a card.

Between 200 and roughly 100,000 pages a month: you're in the mid tiers, and the deciding variable is concurrency, not credits. If you run 20 parallel workers, check the thread count on the tier before the allowance.

Around 1,000,000 pages a month, or any workload where a pause in data hurts: Scaling. Annual billing drops it from $475 to $427.50 a month if you're confident about the 12-month horizon, but the 7-day refund window is short, so spend a week on a paid month first. 👉 [Compare the Scaling and Advanced allowances on the pricing page](https://www.scraperapi.com/pricing?fp_ref=coupons)

Above 4,000,000 pages a month: Advanced at 500 threads and priority routing, and at that spend level ask about Enterprise pricing before you check out.

Two things to confirm before you enter a card: whether a request that returns nothing still consumes its 5 credits, and what the pay-as-you-go per-credit rate is on your tier, since that's the number that governs your worst month rather than your average one. 👉 [Start with the free 1,000 credits and price your real volume](https://www.scraperapi.com/pricing?fp_ref=coupons)
