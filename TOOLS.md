# Tools Reference

Full documentation for all tools available in the GetExperience MCP Server.

**Booking flow:** `search_experiences` → `get_experience_schedule` → `add_to_checkout` → `create_order`

---

## search_experiences

Search for tours and experiences on GetExperience.com. Returns list of matching experiences with id, title, price, location, duration, rating, photos, booking link.

**Examples:**
- "walking tour in Istanbul" → `location: "Istanbul"`, `categories: ["walking-tour"]`
- "romantic dinner in Paris" → `location: "Paris"`, `categories: ["romantic", "food-and-drink"]`
- "things to do in Bali" → `location: "Bali"`

**Search tips:**
- `location` accepts any city/country in any language — resolved automatically
- `languages` filters tours by guide language (`["es"]` = tours conducted in Spanish)
- `lang` sets the language of titles and descriptions in the response

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `location` | string | no | City, country, or region in any language (e.g. "Istanbul", "Bali", "Paris") |
| `categories` | string[] | no | Activity type filter (see categories below) |
| `dateFrom` | string | no | Start date (YYYY/MM/DD) |
| `dateTo` | string | no | End date (YYYY/MM/DD) |
| `minPrice` | number | no | Minimum price in USD (not cents). E.g. 50 = $50 |
| `maxPrice` | number | no | Maximum price in USD (not cents). E.g. 200 = $200 |
| `guestsCount` | number | no | Number of guests |
| `languages` | string[] | no | Guide language filter (en, es, de, ru, fr, it, tr, ar, zh, ja, ko, pt) |
| `lang` | string | no | Response language for titles/descriptions (en, es, de, ru) |
| `perPage` | number | no | Results per page (default: 25, max: 100) |
| `page` | number | no | Page number (default: 1) |
| `responseFormat` | string | no | `"markdown"` (default) or `"json"` |
| `includeStructuredData` | boolean | no | Include `[GXP_STRUCTURED]` block (default: true) |

### Categories

**Top-level** (recommended for most queries):

animals, nature-and-outdoors, art-and-culture, food-and-drink, entertainment, sightseeing, shopping, sports, wellness, rides, transfers, cruises, stopover-tours, multicity-tours, romantic, children, winter-holiday

**Subcategories** (use for specific/niche searches):

| Top-level | Subcategories |
|-----------|---------------|
| animals | beekeeping, birdwatching, horseback-riding-tour, whale-watching |
| nature-and-outdoors | wildlife, outdoor-hiking, water, night-sky |
| art-and-culture | art, society, science, literature, history |
| food-and-drink | cooking-and-food-making, food-and-market-tours, beer, wine, coffee |
| entertainment | gaming, music, dance, comedy, circus, theater, nightlife |
| sightseeing | walking-tour, museum-tour, photography-tour, extraordinary-tour |
| shopping | shopping-tour, factories-tour, outlet-tours, fashion-events-tour |
| sports | water-sports, mountain-sports, adrenaline-sports, outdoor-sports, snow-sports |
| wellness | yoga, spa, mindfulness, holistic-health, beauty, bodywork |
| rides | cycling-tour, boat-tour, helicopter-tour, hot-air-balloon-tour |
| transfers | rentacar |

### Response structure

```json
{
  "tool": "search_experiences",
  "items": [
    {
      "id": 123,
      "title": "Boat Tour in Istanbul",
      "excerpt": "Short description...",
      "description": "Full description...",
      "price": "$45.00",
      "priceCents": 4500,
      "location": "Istanbul, Turkey",
      "durationMinutes": 120,
      "rating": 4.8,
      "reviewsCount": 42,
      "mainPhoto": "https://...",
      "photos": ["https://...", "https://..."],
      "link": "https://getexperience.com/experiences/123"
    }
  ],
  "total": 25,
  "page": 1
}
```

### Displaying results as cards

Every markdown response includes a `[GXP_STRUCTURED]` block with structured JSON — use it to render rich experience cards in your UI:

```javascript
const match = text.match(/\[GXP_STRUCTURED\]([\s\S]*?)\[\/GXP_STRUCTURED\]/);
const data = match ? JSON.parse(match[1].trim()) : null;
// data.items[] — each item has: id, title, price, priceCents, location,
//   durationMinutes, rating, reviewsCount, mainPhoto, photos[], link
```

To get pure JSON without markdown: set `responseFormat: "json"`.

---

## get_experience_details

Get full details of a specific experience by ID. Call after `search_experiences` — use `id` from search results.

Call when the user wants to learn more about a tour before booking, or to confirm full description, photos, pricing and availability options.

Returns: full description, all photos, price breakdown, duration, guide languages, group size limits, categories, rating, booking link.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | yes | Experience ID from search results |
| `lang` | string | no | Response language (en, es, de, ru) |
| `responseFormat` | string | no | `"markdown"` (default) or `"json"` |
| `includeStructuredData` | boolean | no | Include `[GXP_STRUCTURED]` block (default: true) |

### Response structure

```json
{
  "tool": "get_experience_details",
  "item": {
    "id": 123,
    "title": "Boat Tour in Istanbul",
    "excerpt": "Short description...",
    "description": "Full description with all details...",
    "price": "$45.00",
    "priceCents": 4500,
    "location": "Istanbul, Turkey",
    "durationMinutes": 120,
    "languages": ["en", "ru"],
    "rating": 4.8,
    "reviewsCount": 42,
    "categories": ["sightseeing", "rides"],
    "peopleMin": 1,
    "peopleMax": 10,
    "photos": ["https://...", "https://..."],
    "link": "https://getexperience.com/experiences/123"
  }
}
```

---

## get_experience_schedule

Get available time slots for a specific experience on a specific date. Required step before booking.

**CRITICAL:** Copy `configId`, `startAt`, `durationMinutes` EXACTLY from the chosen slot — pass these unchanged to `add_to_checkout`. Do not modify or recalculate these values.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | yes | Experience ID |
| `date` | string | yes | Date in YYYY/MM/DD or YYYY-MM-DD format (must be a future date) |
| `responseFormat` | string | no | `"markdown"` (default) or `"json"` |
| `includeStructuredData` | boolean | no | Include `[GXP_STRUCTURED]` block (default: true) |

### Response structure

```json
{
  "tool": "get_experience_schedule",
  "experienceId": 123,
  "date": "2026/03/20",
  "firstDate": "2026/03/20",
  "hasPublic": true,
  "hasPrivate": true,
  "hasChildPrice": false,
  "hasBabiesPrice": false,
  "slots": [
    {
      "configId": 456,
      "startAt": "2026-03-20T10:00:00.000Z",
      "durationMinutes": 120,
      "onDemand": false,
      "priceAdult": 4500,
      "priceAdultFormatted": "$45.00",
      "priceChild": null,
      "priceChildFormatted": null,
      "pricePrivateGroup": 18000,
      "pricePrivateGroupFormatted": "$180.00"
    }
  ]
}
```

---

## add_to_checkout

Add an experience to the cart. Call `get_experience_schedule` first and take `configId`, `startAt`, `durationMinutes` from the chosen slot — pass them here EXACTLY as returned.

Save the returned `sessionId` — you will need it for `get_checkout` and to add more items to the same cart.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `experienceId` | number | yes | Experience ID |
| `scheduleConfigId` | number | yes | `configId` from schedule slot — do not modify |
| `startAt` | string | yes | `startAt` from schedule slot (ISO datetime) — do not modify |
| `durationMinutes` | number | yes | `durationMinutes` from schedule slot — do not modify |
| `adults` | number | no | Number of adults (default: 1) |
| `children` | number | no | Number of children (default: 0) |
| `babies` | number | no | Number of babies (default: 0) |
| `isPrivate` | boolean | no | Private group booking (default: false) |
| `currency` | string | no | Currency code: USD, EUR, etc. (default: USD) |
| `sessionId` | string | no | Omit to create new cart. Reuse to add more items to same cart |
| `responseFormat` | string | no | `"markdown"` (default) or `"json"` |
| `includeStructuredData` | boolean | no | Include `[GXP_STRUCTURED]` block (default: true) |

### Response structure

```json
{
  "tool": "add_to_checkout",
  "sessionId": "uuid-...",
  "checkoutId": 789,
  "totalPriceFormatted": "$90.00",
  "totalPrice": 9000
}
```

---

## get_checkout

Get current cart contents by `sessionId`. Use to confirm what's in the cart before placing the order, or to retrieve `checkoutId` if it was not saved after `add_to_checkout`.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `sessionId` | string | yes | Session ID from `add_to_checkout` |
| `responseFormat` | string | no | `"markdown"` (default) or `"json"` |
| `includeStructuredData` | boolean | no | Include `[GXP_STRUCTURED]` block (default: true) |

### Response structure

```json
{
  "tool": "get_checkout",
  "checkoutId": 789,
  "sessionId": "uuid-...",
  "totalPriceFormatted": "$90.00",
  "totalPrice": 9000,
  "items": [
    {
      "experienceId": 123,
      "title": "Boat Tour in Istanbul",
      "priceFormatted": "$45.00",
      "adults": 2,
      "children": 0,
      "babies": 0
    }
  ]
}
```

---

## create_order

Place the final booking order. Last step in the booking flow.

### Two payment modes

**B2B — `paymentSystem: "internal"`** (default)

Instant booking without guest payment. Use for partner integrations where payment is handled separately (invoicing, prepaid balance, etc.).
- Requires: `X-Api-Key` (set as header at connection, or pass as `apiKey` param)
- Requires: `user` object with guest details (`firstName`, `lastName`, `email`, `tel`, `countryCode`)
- Result: order confirmed immediately, host notified by email, booking is active

**B2C — `paymentSystem: "stripe"`**

Guest pays by card. Returns a payment link — pass it to the user to complete payment.
- `X-Api-Key` not required
- `user` object not required (guest fills in details on the payment page)
- Result: order created, payment link returned

*(B2C is coming soon)*

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `checkoutId` | number | yes | Checkout ID from `add_to_checkout` or `get_checkout` |
| `paymentSystem` | string | no | `"internal"` (default, B2B) or `"stripe"` (B2C) |
| `user` | object | no | Guest details (required for B2B) |
| `user.firstName` | string | — | Guest first name |
| `user.lastName` | string | — | Guest last name |
| `user.email` | string | — | Guest email address |
| `user.tel` | string | — | Guest phone number |
| `user.countryCode` | string | — | Two-letter country code (e.g. US, DE, RU) |
| `apiKey` | string | no | API key for B2B. Can also be set as `X-Api-Key` header at connection time |
| `responseFormat` | string | no | `"markdown"` (default) or `"json"` |
| `includeStructuredData` | boolean | no | Include `[GXP_STRUCTURED]` block (default: true) |

### Response structure

```json
{
  "tool": "create_order",
  "orderId": 999,
  "status": "new",
  "priceFormatted": "$90.00",
  "link": "https://getexperience.com/orders/999",
  "paymentSystem": "internal",
  "paymentData": null
}
```

For Stripe, `paymentData` contains the payment link:

```json
{
  "paymentData": {
    "url": "https://checkout.stripe.com/..."
  }
}
```
