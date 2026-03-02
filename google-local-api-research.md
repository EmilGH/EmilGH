# Google Local as a Backend for a Custom Guide — API Research

## Short Answer

There is **no dedicated "Google Local" API**. However, the **Google Places API (New)** is the closest equivalent and is the primary way to programmatically access local business/place data from Google. It can absolutely be used as a backend for a custom guide — with some important caveats around caching, display requirements, and cost.

---

## 1. Google Places API (New) — The Primary Option

The [Places API (New)](https://developers.google.com/maps/documentation/places/web-service/op-overview) is Google's current-generation API for local place data. It provides:

### Key Endpoints

| Endpoint | HTTP Method | URL | Purpose |
|---|---|---|---|
| **Text Search (New)** | POST | `places.googleapis.com/v1/places:searchText` | Search by query string (e.g., "best coffee shops in Austin") |
| **Nearby Search (New)** | POST | `places.googleapis.com/v1/places:searchNearby` | Find places within a radius by type |
| **Place Details (New)** | GET | `places.googleapis.com/v1/places/{placeId}` | Full details for a known place |
| **Place Photos** | GET | `places.googleapis.com/v1/{photoReference}/media` | Retrieve photos for a place |
| **Autocomplete (New)** | POST | `places.googleapis.com/v1/places:autocomplete` | Predictive place search |
| **Places Aggregate** | POST | `areainsights.googleapis.com/v1:computeInsights` | Aggregate counts/IDs matching criteria in an area |

### Data Available Per Place

- Name, address, phone number, website
- Business hours (including current open/closed status)
- User ratings and reviews (up to 5 reviews per request)
- Photos
- Price level
- Place types/categories
- Geographic coordinates
- Google Maps URL
- Editorial summaries and AI-powered (Gemini) summaries
- Deep links to Google Maps (`googleMapsLinks` — Preview)
- Moved-place indicators (`movedPlace` / `movedPlaceId`)
- Relational location info (`addressDescriptor` — nearby landmarks, containing areas)

### How It Works for a Custom Guide

1. **Discovery**: Use Text Search or Nearby Search to find places matching your guide's theme
2. **Details**: Use Place Details to fetch rich information for each place
3. **Display**: Show results on a Google Map with proper attribution
4. **Navigation**: Link users to Google Maps for directions

---

## 2. Pricing (As of March 2025)

Google restructured Maps Platform pricing in March 2025:

### Free Monthly Thresholds

| Tier | Free Requests/Month | Notes |
|---|---|---|
| **Essentials** | 10,000 | Basic place data (IDs, names, locations) |
| **Pro** | 5,000 | Advanced fields (display name, business status, etc.) |
| **Enterprise** | 1,000 | Reviews, hours, ratings, phone, website |

### Approximate Pay-As-You-Go Pricing (per 1,000 requests)

| SKU | ~Cost per 1K |
|---|---|
| Place Details Essentials (IDs Only) | Free |
| Place Details Essentials | ~$5 |
| Place Details Pro | ~$17 |
| Place Details Enterprise | ~$20–25 |
| Place Details Enterprise + Atmosphere | ~$25–32 |
| Text Search Pro | ~$32 |
| Nearby Search Pro | ~$32 |
| Autocomplete (per request) | ~$2.83 |
| Autocomplete (session-based) | Bundled with Place Details |
| Place Photos | ~$7 |

Volume discounts scale from 100K to 5M+ monthly events (up to ~90% off at highest volumes).

### Cost Control with Field Masks

Every request **requires** a field mask via the `X-Goog-FieldMask` header. You are billed at the **highest tier** you touch — if you request one Essentials field and one Enterprise field, you pay the Enterprise rate.

**Field tier examples:**
- **Essentials**: `addressComponents`, `formattedAddress`, `location`, `types`
- **Pro**: `displayName`, `businessStatus`, `googleMapsUri`, `primaryType`
- **Enterprise**: `currentOpeningHours`, `internationalPhoneNumber`, `priceLevel`, `rating`, `websiteUri`
- **Enterprise + Atmosphere**: `reviews`, `editorialSummary`, `generativeSummary`, `delivery`, `dineIn`

**Tip**: Only request the fields you need to minimize cost.

### Subscription Plans

Google offers Essentials (~$275/mo for 100K calls) and Pro (~$1,200/mo for 250K calls) subscription plans. Enrollment window: Nov 2025 – March 2026.

### Cost Example for a Guide App

10,000 monthly users × 3 sessions × (5 searches + 3 detail views):
- Text Search: 150K requests × $0.032 = **~$4,800/mo**
- Place Details: 90K requests × $0.017 = **~$1,530/mo**
- **Total: ~$6,330/mo** (before volume discounts)

For full pricing details: [Google Maps Platform Pricing](https://mapsplatform.google.com/pricing/)

---

## 3. Critical Restrictions for a Custom Guide

### Caching / Storing Results

This is the biggest limitation for a custom guide backend:

- **You CANNOT cache or store** most Google Places data (names, addresses, reviews, photos, hours, etc.)
- **You CAN store Place IDs** indefinitely (refresh every 12 months)
- **You CAN cache lat/lng** for up to 30 days
- Everything else must be fetched live from the API each time

This means your guide **cannot pre-build a database** of places from Google. You must query the API in real time.

### Display Requirements

- Results **must be displayed on a Google Map** (not a competing map like Mapbox or OpenStreetMap)
- You must show Google attribution and third-party data provider credits
- You must display review author information and links

### Rate Limits & Result Limits

- **Nearby/Text Search**: 20 results per request, hard cap of 60 per query (via `nextPageToken` pagination)
- **Autocomplete**: Up to 5 predictions per request
- **Places Aggregate**: Returns place IDs only when count is 100 or fewer; default 1,200 QPM
- Rate limits are per method, per project, per minute — adjustable in the Cloud Console

For full terms: [Google Maps Platform Terms of Service](https://cloud.google.com/maps-platform/terms)

---

## 4. Other Relevant Google APIs

| API | Use Case | Pricing Tier |
|---|---|---|
| [Google Business Profile API](https://developers.google.com/my-business) | Manage your **own** business listings (requires approval) | Free (restricted) |
| [Maps JavaScript API](https://developers.google.com/maps/documentation/javascript) | Embed interactive maps in web apps | Essentials (~$7/1K loads) |
| [Maps SDK for Android/iOS](https://developers.google.com/maps/documentation/android-sdk) | Native mobile map rendering | Free (unlimited) |
| [Geocoding API](https://developers.google.com/maps/documentation/geocoding) | Convert addresses to coordinates and vice versa | Essentials (~$5/1K) |
| [Directions API](https://developers.google.com/maps/documentation/directions) | Get routes between places | Essentials–Enterprise |
| [Routes API](https://developers.google.com/maps/documentation/routes) | Newer routing API with more features | Essentials–Enterprise |
| [Address Validation API](https://developers.google.com/maps/documentation/address-validation) | Verify and standardize addresses | Pro (~$17/1K) |
| [Places Aggregate API](https://developers.google.com/maps/documentation/places-aggregate) | Count/identify places matching criteria in an area | Enterprise |

### Places Aggregate API (Notable for Guides)

Answers questions like "How many 5-star restaurants are within 2km of this location?" Supports filtering by type, status, price level, and ratings. Useful for heatmaps, density analysis, and competitive landscape features.

### Google Local Guides Program

The [Google Local Guides](https://maps.google.com/localguides/) program is a community contribution program — there is **no API** to access Local Guides data or contributions programmatically. The only data export option is [Google Takeout](https://takeout.google.com/) (manual bulk export of your own contributions).

---

## 5. Alternatives to Google Places API

If Google's caching restrictions, display requirements, or pricing are too limiting:

### Commercial Alternatives

| Alternative | Strengths |
|---|---|
| **[Foursquare Places API](https://foursquare.com/)** | 105M+ POIs, 190 countries; rich foot-traffic data; powers Apple Maps; flexible caching |
| **[HERE Places API](https://developer.here.com/)** | 120M+ POIs; strong routing integration; enterprise-friendly terms |
| **[Yelp Fusion API](https://docs.developer.yelp.com/)** | Best-in-class review data; free tier (5,000 calls/day) |
| **[TomTom Search API](https://developer.tomtom.com/)** | ~$0.50/1K requests — up to 40× cheaper than Google |
| **[Geoapify Places API](https://www.geoapify.com/)** | 500+ categories; **allows caching/storing data**; works with any map provider |

### Free / Open-Source

| Alternative | Strengths |
|---|---|
| **[OpenStreetMap / Overpass API](https://overpass-api.de/)** | Free, open data (ODbL license), no caching restrictions |
| **[Mapbox](https://www.mapbox.com/)** | Beautiful map rendering; good geocoding; freemium |

### Scraping-Based Services

| Alternative | Strengths |
|---|---|
| **[SerpApi (Google Local)](https://serpapi.com/google-local-services-api)** | Structured Google Local results as JSON |
| **[Outscraper](https://outscraper.com/)** | Google Maps data including unlimited reviews |
| **[Local Business Data (RapidAPI)](https://rapidapi.com/letscrape-6bRBa3QguO5/api/local-business-data)** | Real-time Google Maps/POI data |

> **Warning**: Scraping-based services violate Google's Terms of Service. Use at your own risk.

---

## 6. Recommended Approach for a Custom Guide

### Cost Reduction Strategies (if using Google)

1. **Use field masks aggressively** — stay in Essentials tier when possible
2. **Use Autocomplete sessions** — Autocomplete portion becomes free when bundled with Place Details
3. **Cache Place IDs** indefinitely (explicitly allowed)
4. **Use IDs Only SKUs** when you only need to identify places
5. **Implement client-side caching** of UI state to reduce redundant API calls
6. **Consider subscription plans** for predictable usage patterns

### Option A: Google-Centric (Simplest, Most Data)

- Use **Places API (New)** for all place data
- Display on **Google Maps**
- Store only **Place IDs** in your database
- Fetch details live on each page load
- **Pros**: Richest data, most up-to-date, trusted source
- **Cons**: Per-request cost, can't pre-cache, must use Google Maps

### Option B: Hybrid (More Flexibility)

- Use **Foursquare** or **HERE** for base place data (more flexible caching)
- Supplement with your own curated content (descriptions, tips, categories)
- Use **OpenStreetMap** for map display
- **Pros**: Can build a local database, no Google Maps requirement
- **Cons**: Less data than Google, may need multiple data sources

### Option C: Fully Open (Most Control)

- Use **OpenStreetMap** data (free, cacheable, open license)
- Add your own curated content and reviews
- Community-maintained data (coverage varies by region)
- **Pros**: Free, full control, no vendor lock-in
- **Cons**: Less business data (hours, reviews, photos) than Google

### Option D: Hybrid (Best of Both Worlds — Recommended)

- **OpenStreetMap** or **Mapbox** for map rendering (free/cheap)
- **Foursquare** or **Geoapify** for POI discovery and basic details (cacheable)
- **Google Places API** only for high-value enrichment (reviews, photos, real-time hours) when a user explicitly taps into a place detail
- **Yelp Fusion** to supplement review data
- **Pros**: Cost-effective at scale, cacheable base data, no vendor lock-in on the map
- **Cons**: More complex architecture, data consistency across sources

---

## Summary

| Question | Answer |
|---|---|
| Is there a "Google Local" API? | No dedicated API by that name |
| Can you use Google data for a custom guide? | Yes, via the **Places API (New)** |
| Can you cache/store the data? | **No** (except Place IDs and lat/lng temporarily) |
| Must you use Google Maps for display? | **Yes**, if using Google Places data |
| Is there a free tier? | Yes — 10,000 events/month on Essentials |
| Are there alternatives? | Yes — Foursquare, HERE, Yelp, OpenStreetMap, and scraping services |
