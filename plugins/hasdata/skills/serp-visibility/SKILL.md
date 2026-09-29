---
name: serp-visibility
description: Check how a site or a query ranks across Google, Bing and DuckDuckGo in one pass, including the AI Overview and the Copilot answer. Use for rank tracking, share-of-voice checks and AI answer monitoring.
---

# Visibility across search engines

Three engines, three notations for the same targeting, and three different definitions of a position. Getting those right is most of the work.

## Ask first

Get the queries and the market. A ranking without a country and a language is answered from wherever the request happens to leave from, which is almost never what the user meant. Get the domain being tracked when the question is about a specific site rather than about who ranks.

## Targeting says the same thing three ways

Google takes `gl`, `hl` and `location`, and `location` is written the way Google writes it, for example `Austin,Texas,United States`.

Bing takes `mkt` for the market, `cc` for the country and `setLang` for the interface language, plus either `location` as free text or `lat` and `lon` as coordinates. Sending both forms silently drops the text, so pick one.

DuckDuckGo takes `kl` in its own notation, such as `us-en` or `de-de`. It also accepts `cc` and `setLang`, but `kl` wins, so setting all three means the last two do nothing.

Use the same market on all three or the comparison is meaningless.

## Gather

Google is `hasdata_google_serp_serp_getSearchResults`. Use `hasdata_google_serp_serp_light_getSearchResults` instead when the task is ten blue links and nothing else, since it is the lighter call.

Bing is `hasdata_bing_serp_getSearchResults`. DuckDuckGo is `hasdata_duckduckgo_serp_getSearchResults`.

Run one call per engine per query and work from the payload. Re-searching to pick up one more field is a second billed call for data you already had.

## A position is not a rank

Bing and DuckDuckGo both restart `position` at 1 in every response, so absolute rank is the number of results already collected plus `position`.

The Bing `first` parameter is a result offset rather than a page number, and it is unreliable. In measurement, `1`, `8` and `21` returned the same results as a call with no offset at all, while `11` returned a different set. An offset Bing quietly ignored looks exactly like a page of fresh results, so compare the links and deduplicate by `link` before trusting page two.

DuckDuckGo pages with `nextPageToken`, passed alone. The token carries both the query and the offset, and sending a fresh `q` alongside it does not error. The token wins silently, so the answer is the previous query next page while the log claims you searched for something else.

Page size is not fixed on any of them. Bing returned six organic results on one call and fourteen on another for the same query, because it decides how much of the page goes to ads and widgets. Never hard-code ten.

## The AI answers

The Google AI Overview usually arrives inline on the search response as `aiOverview`, with `textBlocks` and a `references` array that names the cited sources. When Google returns a collapsed block instead, the response carries a `pageToken`, and that token goes to `hasdata_google_serp_ai_overview_getAiOverviewResponse`. Tokens go stale, so read it out of the response you just received rather than from an earlier run.

The Bing answer is `copilotSearch`, structured as `textBlocks` typed as headings and lists. Its `references` array is often empty, so a citation list can have nothing in it to report, and that is a finding rather than a bug.

The DuckDuckGo answer is `searchAssist`.

Citation in an AI answer is a different question from an organic ranking. Report them as two columns, since a domain often holds one and not the other.

## Absent, not empty

On Bing and DuckDuckGo the `ads`, `copilotSearch` and `searchAssist` keys are absent rather than empty when the page has none, so test for the key before reading it.

DuckDuckGo never returns an empty result set. A nonsense query comes back with ten loosely related entries and nothing in the payload marks them as a miss. Bing does the same, and a deliberately meaningless query still returned a full page of ten organic results, so neither engine will tell you that a query has nothing behind it. Judge relevance yourself and say when the results do not answer the query. Google does return empty sets, and an empty set there is a valid answer rather than an error.

A Bing ad `link` is a click tracker on `bing.com/aclk` with the destination buried inside. Read `displayedLink` for the advertiser domain.

## Report

A row per query with the tracked domain rank on each engine, or a blank where it does not rank, then whether it appears in each engine AI answer. Say which market you used and when the pull happened, because all of this changes daily.
