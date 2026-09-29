# HasData for Cursor

Public web pages as structured JSON, from inside Cursor. Google and Bing results, Amazon and Walmart listings, Maps and Yelp places, Zillow and Redfin properties, Airbnb and Booking stays, Indeed and Glassdoor postings, TikTok, Instagram, Facebook and YouTube, plus a general page scraper for everything else.

The scraping runs on HasData infrastructure, so there is no headless browser to install and no proxy pool to keep alive.

## Install

Install the plugin from the Cursor marketplace, then set your HasData API key in the environment Cursor starts from:

```bash
export HASDATA_API_KEY=your_key_here
```

Get a key at [hasdata.com](https://hasdata.com). The free tier does not need a card.

The bundled `mcp.json` reads that variable. If the key is missing the server answers 401 on the first call, which is the failure to expect when tools appear but nothing returns.

### Connecting with OAuth instead

The endpoint also speaks OAuth 2.1 with dynamic client registration. To use it, drop the `headers` block from the MCP server entry and let Cursor run the authorisation flow on first connect. The API key route is the default here because it works without a browser round trip.

## What is in the box

**One MCP server** at `https://mcp.hasdata.com/mcp`, exposing every HasData tool.

**A rule per site.** Each one is written from measured payloads and covers the traps that a tool description cannot: which identifiers are marketplace-scoped, which fields are absent rather than empty, which failures answer 200 with an error string inside, and which numbers are strings. They load only when the agent judges them relevant, so the whole set costs nothing until it is needed.

**A command per site**, for the jobs people actually run. Comparable sales from Zillow, a reputation read from Yelp, a hiring map from Indeed, a rate check on Google Hotels, and so on.

**Cross-service skills** for the work that needs several sites at once. Pricing a product across Amazon, Walmart and Google Shopping. Building a local business dossier from Maps, Yelp, Yellow Pages and Facebook. Checking visibility across Google, Bing and DuckDuckGo including the AI answers. Vetting creators across TikTok, Instagram and YouTube.

## Narrowing the tool list

The full endpoint exposes every tool at once. A model choosing among dozens of similarly shaped tools picks wrong more often than one choosing among five, and every tool description occupies context whether or not it gets called.

Add `?apis=` to the URL to filter it down:

```json
{
  "mcpServers": {
    "hasdata": {
      "type": "streamable-http",
      "url": "https://mcp.hasdata.com/mcp?apis=amazon,walmart",
      "headers": { "x-api-key": "${HASDATA_API_KEY}" }
    }
  }
}
```

Services combine comma-separated or by repeating the parameter. An unknown name is an error rather than an empty list, so a typo fails loudly.

The service segment is the same one that appears in every tool name, which follows `hasdata_<service>_<group>_<method>`. It does not always match the product name: Google Search is `google_serp`, flights are `google_travel_flights` and hotels are `google_travel_hotels`.

## Billing

Every successful call spends credits from the connected account. A call that fails validation is not billed. A request that returns an empty result set is a successful call.

## Links

- [HasData](https://hasdata.com)
- [API documentation](https://docs.hasdata.com)
- [Per-service MCP servers](https://hasdata.com/mcp), for clients that only ever need one site

## License

MIT
