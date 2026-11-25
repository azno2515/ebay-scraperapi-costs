# eBay Search Scraper: Buy, Build or Rent One? What Each Route Costs, Which URL Filters Fail Silently, and Why Your Server's Country Changes the Results (5 Credits per eBay Page, All 8 Plans Compared)

Two questions decide this: do you need eBay sold prices and sell-through rates as a finished report, or do you need raw eBay pages that you parse yourself? The first answer points to a dedicated eBay scraper. The second points to a scraping API like ScraperAPI, where an eBay request costs **5 credits** against 1 for an ordinary page. A third option, an MIT-licensed Scrapy project, costs nothing but asks you to keep it running.

Whichever route you pick, three things on eBay's side decide whether the numbers coming back are right: how the search URL is built, which country the request originates from, and whether you asked for active or sold listings. This article covers what ScraperAPI charges per eBay page and per plan, the URL filters that quietly return unfiltered listings while still billing you, the country gap that hides roughly a third of sold results, and the situations where you shouldn't pay for anything.

## The short version

> At 5 credits per eBay request, the free plan's 1,000 credits cover 200 pages a month, which is enough to test an idea and not enough to monitor a category. The 7-day trial gives 5,000 credits (about 1,000 eBay pages) with no card required, and **Scaling** is the tier built for continuous work: 5,000,000 credits, 200 concurrent threads, $475/month or $427.50/month on annual billing.

If you're still deciding whether eBay scraping is viable at all, spend the trial before you spend money: 👉 [Start the 7-day trial and price out a real eBay job](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## What one eBay page actually costs

ScraperAPI does not bill every site the same, and eBay sits in the more expensive band. The published credit table:

| Request type | Credits |
| --- | --- |
| Standard page | 1 |
| E-commerce page (Amazon, Walmart, eBay) | 5 |
| Google or Bing SERP | 25 |
| LinkedIn | 30 |
| Bot-protection bypass | +10 |
| JS rendering (`render=true`) | +10 |
| Premium proxies (`premium=true`) | +10 |
| Premium proxies plus JS rendering | 25 total |
| Ultra premium (`ultra_premium=true`, paid plans only) | 30 |
| Ultra premium plus JS rendering | 75 |
| Geotargeting, sessions, device type, output format, auto parsing | 0 |

The add-ons stack. A plain eBay search page is 5 credits, the same page with JS rendering is 15, and premium proxies plus rendering lands at 25. Two things follow from that. Geotargeting costs nothing extra, which matters because country selection changes results (more on that below). And any filter you get wrong is billed at full price, because eBay returns an unfiltered page instead of an error.

ScraperAPI reports each response's cost in an `sa-credit-cost` header, offers a Domain Cost Estimator in the dashboard, and accepts a `max_cost` value per scrape so a single request can't run past your limit.

## ScraperAPI's plans, and which tier fits eBay work

ScraperAPI lists eight tiers. Free and Scaling are the two whose figures appear on its pricing and billing pages; the tiers between and above them step the monthly credit allowance, concurrency and price upward, and each is listed on the pricing page.

| Plan | Monthly allowance | Price (USD) | Billing | Get it |
| --- | --- | --- | --- | --- |
| Free | 1,000 credits, 5 concurrent threads | $0 | Monthly | [Start on the free plan](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Hobby | Tier allowance listed on the pricing page | Listed on the pricing page | Monthly | [See Hobby pricing](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Startup | Tier allowance listed on the pricing page | Listed on the pricing page | Monthly | [See Startup pricing](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Business | Tier allowance listed on the pricing page | Listed on the pricing page | Monthly | [See Business pricing](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Scaling | 5,000,000 credits, 200 concurrent threads | $475/month, or $427.50/month billed annually ($5,130 a year) | Monthly or annual | [See Scaling pricing](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Professional | Tier allowance listed on the pricing page | Listed on the pricing page | Monthly or annual | [See Professional pricing](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Advanced | Tier allowance listed on the pricing page | Listed on the pricing page | Monthly or annual | [See Advanced pricing](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Enterprise | Custom volume | Contact sales | Custom | [Ask about custom volume](https://www.scraperapi.com/pricing/?fp_ref=coupons) |

At Scaling's 5,000,000 credits, an all-eBay workload works out to about 1,000,000 page requests a month. Annual billing on that tier saves $570 against twelve monthly payments.

A few mechanics decide whether a plan fits:

- Auto-upgrade (moves you to the next tier when credits run out) is available on Hobby, Startup, Business, Scaling and Professional.
- Pay-as-you-go past your limit, at a fixed per-credit rate up to a cap you set, is available on Scaling, Professional, Advanced and Enterprise.
- Hitting the concurrency limit returns a 429 instead of an extra charge. The request is refused, not queued.
- New accounts get a 7-day trial with 5,000 credits and no card, and paid plans carry a 7-day no-questions-asked refund.

## The eBay URL is where most scrapers get it wrong

eBay search takes two kinds of input. You can send a plain keyword, optionally with condition, price, sort, listing type, free shipping, seller and category filters. Or you can paste a search URL you built on eBay itself, which carries item location, brand and other item specifics, several conditions at once, accepts-offers, and seller include or exclude lists.

Output comes in three modes: active listings, sold and completed listings with real sold prices, or both. Sold history mostly reaches back about 90 days, and a parameter such as `soldWithinDays` sets an exact window when 90 days is too blunt. That 90-day default is why "average sold price" figures from different tools rarely agree.

The filter checks below were recorded by the maintainers of a dedicated eBay scraper sold on Apify, and they are worth reading before you build anything, because each count shows what eBay actually did rather than what the documentation promised.

| Goal | Parameter | What the check found |
| --- | --- | --- |
| Items located in one country | `LH_LocatedIn=1&_salic=45` (45 = China) | 212 of 212 results located in China |
| Only domestic items | `LH_LocatedIn=1&_salic=77` on ebay.de | 212 of 212 domestic, against 21 of 212 foreign without it |
| Brand or other item specific | `Brand=Nike` on ebay.com, `Marke=Nike` on ebay.de | Nike in 212 of 213 titles, against 9 of 213 unfiltered |
| Two conditions at once | `LH_ItemCondition=1000%7C3000` | Only Brand New and Pre-Owned survived; New (Other) dropped |
| Accepts offers | `LH_BO=1` | 213 of 213 rows flagged as accepting offers |
| Free shipping | `LH_FS=1` | 213 of 213 rows with free shipping |
| Seller whitelist | `_fss=1&LH_SpecificSeller=1&_saslop=1&_sasl=seller1%2Cseller2` | Only those two sellers |
| Seller blacklist | Same with `_saslop=2` | None of the three excluded sellers, who held 37 of 213 rows |
| Sold listings plus filters | `Brand=Nike&LH_ItemCondition=1000` in sold mode | 200 of 200 sold rows Brand New |

## Three filters that fail without an error

**A top-level category needs a keyword.** eBay won't enumerate a whole top-level category. `_sacat=1` (Collectibles) on ebay.com returns no listings on every attempt, and an unknown category number ends the same way. Add `_nkw=...` or use a subcategory. A scraper that treats an empty page as a successful run will hand you a zero-row dataset and a bill.

**Item-specific field names follow the site's language.** On ebay.de, `Marke=Nike` filtered correctly (Nike in 210 of 211 titles) while `Brand=Nike` changed nothing at all. eBay drops filters it doesn't recognise without an error message, so a mistyped parameter returns the unfiltered result set and you pay for it. If you're scraping several marketplaces, build the field names per site.

**`LH_PrefLoc` is ignored on mobile search.** URLs copied from a phone can carry it. Replace it with `LH_LocatedIn=1&_salic=<code>` or your location filter silently disappears.

## Your server's location decides how many results you get

On ebay.com, a connection from outside the United States returned **393 of the 594 sold results** that a US connection received. Nothing in the response explains what's missing; the pages look complete. For sold-price research, that gap is the difference between a usable median and a guess.

Two habits prevent it. Match the regional domain to the marketplace setting, since the domain picks the site while the connection country follows the marketplace field, so ebay.de and ebay.co.uk jobs should run as separate jobs. And check a known quantity first: if a search returns 212 results from your server and 240 from a colleague's, you have a geography problem, not a data problem.

ScraperAPI's credit table prices geotargeting at zero extra credits, and the Scaling tier advertises global, country-level targeting: 👉 [Check which plans include country-level geotargeting](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## Three routes, side by side

|  | Dedicated eBay scraper | MIT-licensed Scrapy project | Scraping API plus your parser |
| --- | --- | --- | --- |
| Setup | Minutes, keyword or URL in | Python environment, proxy account, deployment | API key, HTTP client, parser |
| Fields out of the box | Titles, prices with currency, condition, shipping cost, seller name and feedback, bid counts, item specifics, sold price and date | Search spider covers listings, pricing, auction data, seller info; product spider reaches 80+ fields | Raw HTML, or structured JSON on the eBay endpoints |
| Analytics | Avg, median, min and max sold price, recommended price, sell-through rate, units sold per day | You compute them | You compute them |
| Maintenance | Vendor handles eBay changes | Yours, permanently | Vendor handles pages, you handle parsing |
| Cost model | Per result, plus platform proxy and compute | $0 licence, paid proxy, your time | Credits per page |

The open-source route is real and worth naming precisely. One Scrapy project ships search and product spiders, user-agent rotation, request throttling, session handling, price normalisation, CSV and JSON export, and support for US, UK, DE, CA, AU, FR, IT, ES. A run looks like `scrapy crawl ebay_search -a query="iPhone 15" -a max_results=100 -a condition="new"`, and it accepts `-a min_price=500 -a max_price=1500 -a sort="PricePlusShippingLowest" -a site="UK"`. It also depends on a third-party proxy service whose free tier allows 1,000 requests, which tells you the proxy is not optional. Its published benchmarks, roughly 100 search listings a minute, 30 detailed products a minute, over 90% success and over 95% field extraction accuracy, are the author's own figures, not an independent test. The licence is MIT, and the disclaimer puts compliance with eBay's terms and applicable law on whoever runs it.

The dedicated scraper route has boundaries too. The actor whose filter checks are quoted above doesn't support store-category parameters, `LH_PrefLoc`, distance or ZIP-radius sorting, or sold-date ranges, and it covers 17 marketplaces while a separate actor covers 12. Those are different products; don't average them into a claim about the category.

## The eBay endpoints ScraperAPI already has

This is where a general scraping API stops being just a proxy. ScraperAPI exposes an async eBay search endpoint that takes a `query` (or a `queries` array), `country_code`, `tld`, `sort_by` and a webhook `callback`, and returns JSON or CSV. `sort_by` accepts `ending_soonest`, `newly_listed`, `price_lowest`, `price_highest`, `distance_nearest` and `best_match`. Filters include `condition` (multiple values, comma-separated and URL-encoded) and `buying_format` with `buy_it_now`, `auction` and `accepts_offers`. A `show_only` list covers `returns_accepted`, `authorized_seller`, `completed_items`, `sold_items`, `sale_items`, `listed_as_lots`, `authenticity_guarantee` and more.

There's a product endpoint as well, returning a parsed object with seller name, seller URL, review count, review percentage and top-rated flag, price value and currency, image array, available quantity, units sold, shipping cost, return policy, condition, brand, model and an item-specifics array keyed by label.

For search result pages and item pages, that removes the HTML parser from your to-do list. For seller feedback pages, store fronts and the long tail of eBay page types, you're back to raw HTML and your own selectors. `tld` defaults to `com`, and `country_code` affects the language and currency of the returned page, so a German job means `tld=de` with a matching country code rather than a US request against a German URL.

## Who should not buy this

If what you want is a comps table and a sell-through percentage, buy the dedicated scraper. ScraperAPI returns pages and structured listings, not a recommended price, and you'd be writing the median-of-sold-prices and sell-through logic yourself.

If you already write Scrapy and your volume is a few hundred pages a month, the MIT project costs nothing but your evenings, and 1,000 free proxy requests will get you through development.

If you need someone to tell you that your specific use complies with eBay's terms, none of these three will do it. The open-source project says so in its own disclaimer, and a paid API changes the hosting bill, not the legal question.

## Choosing, and two things to check before you pay

Match the route to the output, not to the pitch:

- **A one-off pull of sold prices for a product list**: a dedicated scraper on its smallest plan, because the analytics come included.
- **Ongoing monitoring across several eBay sites into your own schema**: a scraping API, starting on the 7-day trial, then sized by arithmetic rather than guesswork. Pages per run × runs per day × 30 × 5 credits gives your monthly credit floor at one credit per standard page and five per eBay page.
- **Zero budget plus Scrapy experience**: the MIT project, with a proxy budget and a maintenance calendar.

Two checks before you pay. First, count the detail pages: every item page you open to get item specifics, seller ratings or return policy is another 5-credit request, and adding `render=true` makes each one 15. A 200-result search that enriches every listing costs 1,200 credits, not 1,000. Second, confirm your target marketplaces are covered by the geotargeting you're paying for, because the 393-versus-594 gap on ebay.com is what an unmatched country looks like from the inside: plausible, complete and short.

👉 [Compare every ScraperAPI tier and start with 5,000 free trial credits](https://www.scraperapi.com/pricing/?fp_ref=coupons)
