---
name: price-across-retailers
description: Compare the price of one product across Amazon, Walmart, Google Shopping and a named Shopify storefront. Matches the product on brand and model before comparing, and reports the market and delivery destination behind every figure. Use when the user asks to compare prices, asks whether something is cheaper somewhere else, or needs competitive pricing, a repricing input or a cost check across retailers.
---

# Price across retailers

Four services answer the same question in four shapes. The work is matching the product, not fetching it.

## Ask first

A price has no meaning without a market and a place. Get the product identifier if the user has one, an ASIN, a UPC, a model number or a product URL. Get the country, and get the delivery destination when the answer will be acted on. A comparison that silently mixes a US Amazon price with a Canadian Walmart price is wrong in a way the reader cannot see.

## Gather

Amazon starts with `hasdata_amazon_search_getSearchResults` on the product name, then `hasdata_amazon_product_getProductDetails` on the ASIN with `otherSellers` set when the cheapest offer matters. Pass `domain` and `deliveryZip` deliberately, because both move the price and the seller mix.

Walmart starts with `hasdata_walmart_search_getSearchResults`, then `hasdata_walmart_product_getWalmartProduct` with `itemId` or `url`, and `otherOffers` when a marketplace seller may undercut the shelf price. Read `searchInformation.storeId` out of the response and report it, since an item can be unavailable at that one store while stocked nationally.

Google Shopping is `hasdata_google_serp_shopping_getSearchResults`, which gives the spread across merchants in one call and is the fastest way to find out whether the two retailers above are even the cheap end.

A Shopify storefront is `hasdata_shopify_products_getProducts` with the store domain as `url`. Reach for it only when the user names a store.

## Match the product before comparing

Model numbers match. Titles do not. A Walmart product record carries `brand`, `model`, `upc` and `condition`, which are the fields to align on, while an Amazon ASIN is marketplace-scoped rather than universal.

Do not trust `upc` blindly. Walmart populates it with whatever the seller supplied, and a Dyson vacuum came back with `"626417-01"`, a manufacturer part number rather than a twelve-digit UPC. Check the shape before matching on it, and fall back to `brand` plus `model` when it does not look like a UPC.

When nothing but the title is available, say the match is by title and put the two titles side by side so the reader can judge it.

A pack of two against a single unit is the classic false saving. Check quantity and size before you call anything cheaper.

## Read the prices correctly

Amazon can return a live listing with no usable `currentPrice` when the item is out of stock or sold only by third parties. Treat the price as optional and say it is unavailable rather than printing zero.

Shopify has no product price at all. Each entry in `variants` carries `price` and `compare_at_price` as strings such as `"16.50"`, so parse them to numbers first. A string sort puts `"9.00"` above `"16.00"` and the result looks plausible.

Walmart's `reviews.totalReviews` counts ratings rather than written reviews, which matters when the summary leans on review volume as a proxy for how established a listing is.

## Report

Give the price per retailer with the currency, the market and the destination used, then the gap. Say how the products were matched. If one retailer does not carry the item, that is a finding worth stating rather than a row to leave out.
