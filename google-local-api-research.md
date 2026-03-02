# Google Local as a Backend for a Custom Guide — API Research

## Short Answer

There is **no dedicated "Google Local" API**. However, the **Google Places API (New)** is the closest equivalent and is the primary way to programmatically access local business/place data from Google. It can absolutely be used as a backend for a custom guide — with some important caveats around caching, display requirements, and cost.

---

## 1. Google Places API (New) — The Primary Option

The [Places API (New)](https://developers.google.com/maps/documentation/places/web-service/op-overview) is Google's current-generation API for local place data. It provides:

### Key Endpoints

| Endpoint | Purpose |
|---|---|
| **Text Search (New)** | Search for places by query string (e.g., "best coffee shops in Austin") |
| **Nearby Search (New)** | Find places within a radius of a location |
| **Place Details (New)** | Get full details for a specific place by Place ID |
| **Place Photos (New)** | Retrieve photos associated with a place |
| **Autocomplete (New)** | Suggest places as the user types |

### Data Available Per Place

- Name, address, phone number, website
- Business hours (including current open/closed status)
- User ratings and reviews (up to 5 reviews per request)
- Photos
- Price level
- Place types/categories
- Geographic coordinates
- Google Maps URL
- Editorial summaries

### How It Works for a Custom Guide

1. **Discovery**: Use Text Search or Nearby Search to find places matching your guide's theme
2. **Details**: Use Place Details to fetch rich information for each place
3. **Display**: Show results on a Google Map with proper attribution
4. **Navigation**: Link users to Google Maps for directions

---

## 2. Pricing (As of March 2025)

Google restructured Maps Platform pricing in March 2025:

### Tiers

| Tier | Free Threshold | Notes |
|---|---|---|
| **Essentials** | 10,000 events/month | Basic place data (IDs, names, locations) |
| **Pro** | Lower free tier | Advanced fields (reviews, photos, hours) |
| **Enterprise** | Lowest free tier | Premium data |

### Cost Control with Field Masks

Requests are billed based on which fields you request, not a flat rate:

- **IDs Only** — cheapest (just Place IDs)
- **Location** — coordinates added
- **Basic** — name, address, hours, etc.
- **Advanced** — reviews, photos, price level
- **Preferred** — most expensive, all fields

**Tip**: Only request the fields you need to minimize cost.

### Subscription Plans

Google now offers Starter, Essentials, and Pro subscription plans as an alternative to pay-as-you-go. Useful if you have predictable usage.

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

### Rate Limits

- Default quota limits apply per project
- Can request quota increases through Google Cloud Console

For full terms: [Google Maps Platform Terms of Service](https://cloud.google.com/maps-platform/terms)

---

## 4. Other Relevant Google APIs

| API | Use Case |
|---|---|
| [Google Business Profile API](https://developers.google.com/my-business) | Manage your **own** business listings (not for reading others' data; requires approval) |
| [Maps JavaScript API](https://developers.google.com/maps/documentation/javascript) | Embed interactive maps in web apps |
| [Geocoding API](https://developers.google.com/maps/documentation/geocoding) | Convert addresses to coordinates and vice versa |
| [Directions API](https://developers.google.com/maps/documentation/directions) | Get routes between places |
| [Routes API](https://developers.google.com/maps/documentation/routes) | Newer routing API with more features |

### Google Local Guides Program

The [Google Local Guides](https://maps.google.com/localguides/) program is a community contribution program — there is **no API** to access Local Guides data or contributions programmatically.

---

## 5. Alternatives to Google Places API

If Google's caching restrictions, display requirements, or pricing are too limiting:

| Alternative | Strengths |
|---|---|
| **[Foursquare Places API](https://foursquare.com/)** | Large POI database, more flexible caching, no map display requirement |
| **[HERE Places API](https://developer.here.com/)** | Strong mapping platform, enterprise-friendly terms |
| **[Yelp Fusion API](https://docs.developer.yelp.com/)** | Rich review data, restaurant/service focused |
| **[OpenStreetMap / Overpass API](https://overpass-api.de/)** | Free, open data, no caching restrictions, community-maintained |
| **[SerpApi (Google Local)](https://serpapi.com/google-local-services-api)** | Scrapes Google Local results into structured JSON |
| **[Outscraper](https://outscraper.com/)** | Scrapes Google Maps data including unlimited reviews |
| **[Local Business Data (RapidAPI)](https://rapidapi.com/letscrape-6bRBa3QguO5/api/local-business-data)** | Real-time Google Maps/POI data via API |

> **Note**: Scraping-based services (SerpApi, Outscraper, etc.) may violate Google's Terms of Service. Use at your own risk.

---

## 6. Recommended Approach for a Custom Guide

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
