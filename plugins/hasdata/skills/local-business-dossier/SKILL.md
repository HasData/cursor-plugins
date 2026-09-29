---
name: local-business-dossier
description: Build a profile of one local business or a set of them from Google Maps, Yelp, Yellow Pages and Facebook, with contact details and reputation reconciled across sources. Use for lead research, competitor checks or NAP verification.
---

# Local business dossier

Four directories hold overlapping records for the same business, and they disagree. The value of pulling all four is the disagreement.

## Ask first

Get the business name and the city, or the category and the city when the task is a list rather than one company. Decide up front whether the output is contact details, reputation, or both, because that changes which calls are worth making.

## Gather

Google Maps is `hasdata_google_maps_search_performMapSearch` with `q` and `ll` for the area, then `hasdata_google_maps_place_getPlaceDetails` on the `placeId`. Reviews come from `hasdata_google_maps_reviews_getMapReviews`, which takes `dataId` or `placeId`.

The Maps search answers in two shapes and the key tells you which. A category query such as `coffee in Austin` returns `localResults`, an array of about twenty rows already carrying the phone, address, website, working hours, rating and coordinates, which is often the whole dossier with no second call. A query that Maps resolves to a single entity returns `placeResults`, one object rather than an array, and `localResults` is then absent. Phrasing decides which you get, so test for the key before iterating and never assume an array.

Yelp is `hasdata_yelp_search_getSearchResults`, which needs both `keyword` and `location`, then `hasdata_yelp_place_getPlaceDetails` and `hasdata_yelp_reviews_getPlaceReviews` on the `placeId`. The review tool filters by `rating` and sorts by relevance, date or elite status, so a reputation check does not mean reading everything.

Yellow Pages is `hasdata_yellowpages_search_getSearchResults`, which needs `keyword` and `location` as well, then `hasdata_yellowpages_place_getPlaceDetails` on the listing `url`. Rows land in `organicResults`.

Facebook is `hasdata_facebook_profile_getFacebookProfile` on the page handle, and it is worth one call when the task needs the published phone, the website or how active the page is.

## Reconcile rather than concatenate

The same business carries a different name, a different suite number and a different phone in each directory. Pick the record you trust as the base, then list the fields where the others differ instead of picking silently. A mismatched address across three directories is itself the answer to a NAP audit.

Rating scales are not comparable. A Google rating and a Yelp rating are both out of five and are computed from different populations, so report them side by side and never average them.

Review counts drift daily. Quote them with the date of the pull, or describe the shape rather than the figure.

## Watch for

A query with no real match fails differently in each of the four, and none of them simply hands back an empty list. Maps guesses: a nonsense keyword returned a single unrelated company in `placeResults` rather than nothing at all. Yelp drops the key, so `organicResults` is absent while `ads` still arrives with entries in it, and code that reads the ads as results reports paid listings as matches. Yellow Pages answers 400 with `Invalid api response`, which is a failed lookup rather than a broken key. Test for the key you came for, and judge whether the rows actually match the business you asked about.

Yelp and Yellow Pages both require a location, and a vague one such as a state name returns a wide, low-quality set. Push for a city.

Every successful call spends credits from the connected account, so pull the search once and work from the payload rather than re-searching per field.

## Report

One row per business with the name, address, phone, site, category, rating and review count from each source that has one, then a short note on where the sources disagree and which one you would trust for this purpose.
