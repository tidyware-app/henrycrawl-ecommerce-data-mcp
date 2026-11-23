# HenryCrawl E-commerce Data MCP

Live e-commerce data for AI agents in one MCP server: **Amazon, eBay, 1688 / AliExpress and Rakuten**.
One license key, no scraping setup, no Apify account, no proxies to manage.

- **$19 / month = 3,000 credits.** Amazon and eBay calls cost 1 credit per result with a **20-credit minimum per successful call** (so at most 150 such calls a month); 1688 / AliExpress and Rakuten cost 1 credit per result, minimum 2. Failed or empty calls are free.
- Works with Claude Desktop, Claude Code, Cursor and any MCP client that supports remote (Streamable HTTP) servers.
- Clean, typed rows built for agents: numeric prices, currency, ratings, review counts, item URLs, plus Amazon-specific Prime / sponsored flags.

**Buy a key:** `https://buy.polar.sh/polar_cl_tKZC0cGfoiqq7ub3jwCUZwIVUKzoJmAshlRrg1wyMbh`
**Server URL:** `https://henrycrawl-ecommerce-mcp.henrycrawl.workers.dev/mcp`

## 1. Get your key
1. Open the checkout link above and subscribe ($19 / month).
2. Your **license key** (it starts with `HENRY-`) is shown on the purchase confirmation page and in the Polar customer portal: `https://polar.sh/tidyware/portal` (the Polar organization is named *tidyware* — same seller as HenryCrawl; Polar Software, Inc. is the merchant of record) → sign in with the email you paid with (Polar emails you a 6-character code) → **Benefits → HenryCrawl E-commerce MCP license key → Copy**. The receipt email contains a link to that portal, not the key itself.
3. Cancel any time in the same portal (**Manage subscription**). After cancelling, your key keeps working until the end of the period you already paid for.

## 2. Connect

### Claude Desktop
Settings → Developer → **Edit Config**, add this block to `claude_desktop_config.json`, then **restart Claude Desktop**. Requires Node.js 18+.

```json
{
  "mcpServers": {
    "henrycrawl-ecommerce": {
      "command": "npx",
      "args": [
        "-y", "mcp-remote",
        "https://henrycrawl-ecommerce-mcp.henrycrawl.workers.dev/mcp",
        "--header", "Authorization:${HENRYCRAWL_AUTH}"
      ],
      "env": { "HENRYCRAWL_AUTH": "Bearer YOUR_LICENSE_KEY" }
    }
  }
}
```
(Putting the key in `env` avoids a known issue with spaces in arguments on Windows.)

You can also skip `mcp-remote`: Settings → Connectors → **Add custom connector** and paste `https://henrycrawl-ecommerce-mcp.henrycrawl.workers.dev/mcp?key=YOUR_LICENSE_KEY`. Whether custom connectors are available depends on your Claude plan, and the key sits in the URL — don't share screenshots of that screen.

### Claude Code
```bash
claude mcp add --scope user --transport http henrycrawl-ecommerce https://henrycrawl-ecommerce-mcp.henrycrawl.workers.dev/mcp \
  --header "Authorization: Bearer YOUR_LICENSE_KEY"
```
(`--scope user` makes it available in every project; drop it to add the server to the current project only. Start a new session afterwards.)

### Cursor
`~/.cursor/mcp.json`, then **restart Cursor**:
```json
{
  "mcpServers": {
    "henrycrawl-ecommerce": {
      "url": "https://henrycrawl-ecommerce-mcp.henrycrawl.workers.dev/mcp",
      "headers": { "Authorization": "Bearer YOUR_LICENSE_KEY" }
    }
  }
}
```

### Any other client
Streamable HTTP endpoint `https://henrycrawl-ecommerce-mcp.henrycrawl.workers.dev/mcp`. Send the key as `Authorization: Bearer <key>`, `X-API-Key: <key>`, or `?key=<key>`.

### Check that it worked
First check the client sees the server: Claude Code `claude mcp list` (or `/mcp` in a session), Claude Desktop → Settings → Developer, Cursor → Settings → MCP. Then ask your assistant: *"How many credits do I have left?"* It should call `account_usage` (free) and answer `3000` on a fresh key. If you instead see *No HenryCrawl license key*, the header never reached the server — check the config and restart the client.

## 3. Tools

| Tool | What it returns | Max results / call | Credits |
|---|---|---|---|
| `amazon_search` | Keyword search on 9 Amazon marketplaces (US, UK, DE, FR, IT, ES, CA, AU, JP): ASIN, title, brand, price, list price, discount, rating, reviews, Prime, sponsored, URL | 50 | 1 per result, **minimum 20 per successful call** |
| `amazon_product` | 1–5 products by ASIN: current price, list price, rating, reviews | 5 | 1 per result, **minimum 20 per successful call** |
| `ebay_search` | Live listings on 8 eBay sites: item id, title, price, shipping cost / free-shipping flag, condition, bid count for auctions, seller name and feedback, item location, URL | 50 | 1 per result, **minimum 20 per successful call** |
| `ebay_seller` | Active listings of one eBay seller (competitor monitoring; US, UK, DE, AU, CA stores): same fields as `ebay_search` | 50 | 1 per result, **minimum 20 per successful call** |
| `china_sourcing` | Factory / wholesale offers on 1688.com and AliExpress: price in CNY / USD / VND, price tiers, MOQ, units sold, supplier name and province, title translated to English, URL | 50 | 1 per result, minimum 2 |
| `rakuten_search` | Rakuten Ichiba (Japan): name, price in JPY, shop name, rating and review count, reward points, shipping fee, URL | 50 | 1 per result, minimum 2 |
| `account_usage` | Credits used / left this month | — | free |

**Example:** asking Amazon for 3 results costs 20 credits; asking for 50 costs 50. `amazon_product` (1–5 ASINs) therefore costs 20 whenever it returns results. Asking Rakuten for 3 results costs 3.

### Parameters
| Tool | Required | Optional |
|---|---|---|
| `amazon_search` | `query` | `marketplace` — `com` (US, default), `co.uk`, `de`, `fr`, `it`, `es`, `ca`, `com.au`, `co.jp`; country codes (`US`, `UK`, `DE`, `JP`…) work too · `max_results` (1–50, default 20) · `sort` (`relevance`, `price_low`, `price_high`, `newest`, `rating`) · `min_price` · `max_price` · `prime_only` · `exclude_sponsored` |
| `amazon_product` | `asins` (1–5 ASINs) | `marketplace` (same values as above) |
| `ebay_search` | `query` | `site` — `com` (default), `co.uk`, `de`, `com.au`, `ca`, `fr`, `it`, `es`; country codes (`US`, `UK`, `DE`, `AU`…) work too · `max_results` (1–50, default 20) · `sort` (`best_match`, `newly_listed`, `ending_soonest`, `price_lowest`, `price_highest`) · `condition` (`any`, `new`, `open_box`, `certified_refurbished`, `seller_refurbished`, `used`, `for_parts`) · `buying_format` (`any`, `buy_it_now`, `auction`) · `min_price` · `max_price` · `free_shipping` · `exclude_words` |
| `ebay_seller` | `seller` (store username) | `site` — `com` (default), `co.uk`, `de`, `com.au`, `ca` only · `max_results` (1–50, default 20) · `sort` and `condition` (same values as `ebay_search`) |
| `china_sourcing` | `query` | `platforms` (`1688`, `aliexpress`; default both) · `max_results` (1–50, default 20) · `sort` (`relevance`, `price-asc`, `booked` = most ordered) · `ship_to` (US, VN, GB, DE, FR, AU, CA, JP, KR, SG, MY, TH, PH, ID, BR, MX) · `price_min_cny` · `price_max_cny` |
| `rakuten_search` | `query` | `max_results` (1–50, default 20) · `sort` (`standard`, `priceAsc`, `priceDesc`, `newest`, `reviewCount`, `reviewScore`) · `min_price` · `max_price` · `exclude_ads` |

Price filters are applied at the source, so a tight `min_price` / `max_price` returns fewer rows and costs fewer credits — never below the 20-credit minimum on Amazon and eBay. For `china_sourcing`, prices are returned in CNY, USD and VND while the price filters are in CNY. An unknown `marketplace` or `site` is rejected with a clear error and costs nothing, instead of quietly returning US data.

### Limits
- **3,000 credits per calendar month (UTC).** They reset on the 1st, whatever day you subscribed on; a mid-month subscription gets the full 3,000 for the rest of that month. Unused credits do not roll over.
- **30 calls per minute** per key.
- Calls that return no results, or that fail or time out on our side, cost nothing. If your client disconnects before the server finishes, the call can still complete and be charged — check `account_usage`.
- Amazon / eBay calls take 20–120 s (real browser + residential proxy); 1688 / AliExpress and Rakuten take 10–60 s. Some clients give up after ~60 s: if yours does, ask for fewer results (`max_results` ≤ 10) and run `account_usage` to see whether that call was charged.

### A real row (`amazon_search`, Amazon US)
```json
{
  "asin": "B07FM5HLMF",
  "title": "Rambler 20 oz Stainless Steel Vacuum Insulated Tumbler w/MagSlider Lid",
  "brand": "YETI",
  "priceRaw": "$35.00", "price": 35, "currency": "USD",
  "listPrice": null, "discountPercent": null,
  "rating": 4.8, "reviewCount": 184300, "boughtInPastMonth": 3000,
  "isSponsored": false, "isPrime": false,
  "deliveryText": "FREE delivery Nov 28 - 30",
  "imageUrl": "https://m.media-amazon.com/images/I/61gle6BctvL._AC_UL320_.jpg",
  "productUrl": "https://www.amazon.com/dp/B07FM5HLMF",
  "amazonDomain": "www.amazon.com", "scrapedAt": "2026-11-23T10:38:16.357Z"
}
```
Every call also returns `count`, `credits_charged` and `credits_left`, so your agent can see the cost of what it just did.

## 4. Example prompts
- "Find the 20 cheapest Yeti Rambler 20 oz tumblers on Amazon US and on eBay US (new condition) and show the price gap in a table."
- "Check the current Amazon price and rating of ASINs B0H3CRP1SR and B0CRMZHDG8."
- "Show the 20 newest listings from the eBay seller *overflowoutlet* and point out which ones are under $30."
- "Find 1688 suppliers for silicone phone cases, show unit price in USD and MOQ, and compare with the top Amazon US listings."
- "Search Rakuten for ニンテンドースイッチ and give me the 10 best-reviewed offers."
- "How many credits do I have left?"

## Errors you may see
- *No HenryCrawl license key* → the key never reached the server; check the header and restart your client.
- *License key not found, revoked or expired* → check the key, or renew at the checkout link.
- *Monthly quota used up* → credits reset on the 1st (UTC); you can also buy a second key.
- *Rate limit* → max 30 calls per minute.
- *The data source took too long* → retry with a smaller `max_results`. You are not charged for this error.

## Support and refunds
- **Questions, bugs or refund help: email `support@tidyware.app`** — we reply within 1 business day. No account anywhere needed.
- **Refunds:** Polar Software, Inc. is the merchant of record. Email us, or request the refund yourself from the Polar customer portal (`https://polar.sh/tidyware/portal`). Refunds asked for within 7 days of a charge are given in full. Unused credits at the end of a month are not refunded.
- **Cancelling** never needs us either: do it yourself in the same portal (**Manage subscription**).
- You can also open an issue on any HenryCrawl actor page at `https://apify.com/henrycrawl` (Issues tab) if you already have an Apify account. Using the MCP server itself never needs one.
