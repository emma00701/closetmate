# Vision: ClosetMate, an AI Wardrobe and Packing Assistant

## Executive Summary

ClosetMate is a local web app that turns a person's real closet into a personal stylist and packing planner. The user photographs their clothes once. A vision LLM tags each item (category, color, formality, warmth, and so on), and an LLM agent then uses that digital closet with live weather data and the user's plans to answer three everyday questions: *What should I wear today?*, *What should I pack for this trip?*, and *What should I wear (or buy) for this special occasion?* Every recommendation uses clothes the user already owns, explains its reasoning, and lists only the genuine gaps worth buying. ClosetMate is for busy students and young professionals who own enough clothes but still waste time each morning, overpack for trips, and impulse-buy before events.

## Problem Statement

People make clothing decisions every day with poor information. They forget what they own, can't easily match outfits to the weather or the day's activities, and repeat the same few items. These problems get worse at two moments. **Travel**: a multi-city itinerary with mixed activities (hiking, a business meeting, a dinner) and changing weather leads to overpacking, missing items, or last-minute purchases. **Special occasions**: a wedding, interview, or themed party with a dress code ("cocktail attire", "business casual", "black tie optional") leads people to buy something new without checking whether their closet already has a workable outfit.

Existing tools fall short. Weather apps don't know your closet. Closet-catalog apps require tedious manual tagging and offer rule-based suggestions that ignore context. Generic chatbots give advice like "bring a light jacket" that ignores what you own. Solving this needs a system that can (1) understand photos of clothing, (2) read unstructured plans such as an itinerary, an event invitation, or "office then gym then dinner", (3) gather live facts such as forecasts, and (4) reason across all of it to produce a concrete, explained plan. That combination of perception, language understanding, tool use, and multi-step reasoning is what LLM agents are good at.

## Target Users

| User type | Description | What they need from ClosetMate |
| --- | --- | --- |
| Daily dresser (primary) | Student or young professional with a varied weekly schedule (classes, office, gym, evening plans) | A fast, weather-aware daily outfit from their own closet that fits the day's activities and doesn't repeat yesterday's |
| Frequent traveler | Takes 2–8 trips per year: weekend trips, conferences, multi-city vacations | Upload or paste an itinerary and get a day-by-day outfit plan plus a minimal packing list, with re-wearable pieces and carry-on limits respected |
| Occasion planner | Has an upcoming wedding, interview, gala, or themed event with a dress code | Outfit options from their closet that meet the dress code, and if nothing works, a short, specific list of what to buy |

## Key Success Metrics

| Metric | Target (measured during final demo / team testing) |
| --- | --- |
| Auto-tagging accuracy | ≥ 85% of auto-generated tags (category, primary color, formality) accepted by the user without editing, on a test set of ≥ 40 real clothing photos |
| Daily outfit usefulness | ≥ 4 of 5 daily recommendations rated "would wear" by team testers across ≥ 20 test days with varied weather |
| Itinerary extraction | Destinations, dates, and activities correctly extracted from ≥ 9 of 10 sample itineraries (pasted text and PDF) |
| Packing list quality | Every planned outfit is buildable from the packing list; packing list contains ≥ 25% fewer items than a naive "one outfit per day" list for trips ≥ 5 days |
| Grounding | 100% of recommended "owned" items exist in the user's closet (no hallucinated items); all "buy" items are clearly labeled as not owned |
| Response time | Daily outfit returned in ≤ 20 seconds; trip plan for a 7-day trip in ≤ 60 seconds |

## Full Scope Vision (long term, beyond this course)

- **Closet intelligence**: auto-tagging from photos, background removal, duplicate detection, cost-per-wear tracking, "closet gaps" analysis, laundry status.
- **Daily styling**: weather- and calendar-aware outfits, style learning from feedback, outfit history and rotation, "complete the look" suggestions.
- **Trip planning**: itinerary parsing from text, PDF, email, or booking confirmations; climate-aware planning for far-future trips; capsule packing; luggage constraints; laundry-on-trip planning; shared packing lists for group trips.
- **Occasions**: dress-code interpretation, cultural/venue norms, multiple outfit options, shopping suggestions with real product links and budget filters.
- **Integrations**: calendar sync (Google/Outlook), retailer APIs, virtual try-on images, mobile app with camera capture, sustainability insights (rent/borrow/second-hand).

## MVP Scope: Features IN

If a feature is not listed here, it is not in the MVP.

| # | Feature | Description | Rationale |
| --- | --- | --- | --- |
| 1 | Closet upload with AI auto-tagging | Upload one or more clothing photos (JPEG/PNG/WebP). A vision LLM returns structured tags: category, subcategory, primary/secondary colors, pattern, material guess, formality (1–5), warmth (1–5), suitable seasons, and a short description. Tags are validated against a fixed schema. | Manual tagging is the main reason closet apps get abandoned; this is the core LLM perception feature. |
| 2 | Closet management | Browse the closet as a photo grid; filter by category, color, formality, season; edit any tag; delete items; mark items unavailable (e.g., in the laundry). | Users must be able to correct AI mistakes, and the agent must only use available items. |
| 3 | Style profile | One-time profile: preferred style keywords, home city, temperature sensitivity (runs cold / neutral / runs warm), items or colors to avoid, typical weekly activities. | Personalizes every recommendation; gives the agent stable context. |
| 4 | Daily outfit recommendation (agent) | User picks a date (default today) and optionally types the day's plans ("lab in the morning, interview at 2pm, drinks after"). The agent calls tools to get the weather for the user's city, search the closet, and check recent wear history, then returns 1 primary outfit + 1 alternative, each with a short explanation tied to weather and activities. | The core daily use case; demonstrates agentic tool use. |
| 5 | Outfit feedback and wear log | Thumbs up/down on an outfit, "swap this item" (agent proposes a replacement), and "I wore this" (logs wear date). | Avoids repeats, gives the agent history, and lets users steer results. |
| 6 | Trip planner from itinerary (agent) | User pastes an itinerary or uploads a PDF/TXT. The agent extracts a structured trip (cities, date ranges, planned activities), shows it for the user to confirm or edit, then fetches weather per city/date (forecast if within 16 days, otherwise historical climate averages for those dates) and produces: (a) a day-by-day outfit plan, (b) a consolidated packing list with quantities that re-uses versatile items, and (c) a "consider buying" list for gaps (e.g., no rain jacket for a rainy week). | Main differentiator; multi-step agent reasoning over documents, tools, and closet. |
| 7 | Special occasion planner (agent) | User describes an event (type, dress code if known, date, location, indoor/outdoor, any notes) or pastes the invitation text. The agent interprets the dress code, checks weather for the date/location, and returns up to 3 outfit options from the closet, each rated for dress-code fit, or explains why the closet falls short. | Covers the "special occasion" use case the team identified; tests nuanced language interpretation. |
| 8 | Shopping gap list | All "consider buying" items from trips and occasions are collected into one list. Each entry has a generic description (e.g., "navy unstructured blazer"), why it is needed, which plan it came from, a rough price band, and a retailer-neutral web search link. The user can dismiss items or mark them purchased (then add a photo to the closet). | Turns "what should I buy?" into a concrete, justified, deduplicated list without e-commerce integration. |
| 9 | Saved plans | Trip and occasion plans are saved and can be reopened, regenerated, or deleted. | Users plan trips days in advance and come back to them. |
| 10 | Guardrails and transparency | Every recommendation shows which tools were used (e.g., "Weather: 8°C, 70% rain in Boston on Mar 3") and labels owned vs. to-buy items. Owned items are validated against the database before display. Friendly errors when the LLM or weather API is unavailable. | Builds trust and prevents hallucinated items; important for an agentic product. |

## MVP Scope: Features OUT

| Feature | Reason for exclusion | Target phase |
| --- | --- | --- |
| Multi-user accounts, login, and authentication | Product runs locally for one user on a laptop; auth adds work without demo value | Phase 2 |
| Calendar sync (Google/Outlook) | OAuth setup is time-consuming; typed day plans cover the use case for the MVP | Phase 2 |
| Real product search, prices, or affiliate/retailer APIs | API access, costs, and terms of service; generic descriptions plus a search link are enough | Phase 2 |
| Purchasing/checkout | Out of product scope and raises payment and security concerns | Not planned |
| Virtual try-on / AI-generated outfit images | Expensive, slow, and image generation is not needed to solve the core problem | Phase 3 |
| Automatic background removal / photo editing | Nice to have; vision tagging works on ordinary photos | Phase 2 |
| Mobile app or camera capture | Local web app in a desktop/mobile browser is enough for the MVP | Phase 3 |
| Parsing emails, booking confirmations, or images of itineraries | Pasted text and PDF/TXT upload cover the core flow | Phase 2 |
| Size/fit recommendations and body measurements | Sensitive data and needs retailer size data | Phase 3 |
| Luggage weight/volume optimization | Item counts and a carry-on vs. checked flag are enough for MVP | Phase 2 |
| Laundry-on-trip planning beyond a simple "re-wear allowed" rule | Adds complexity to the planner | Phase 2 |
| Social sharing, friends' closets, group packing lists | Not core to a single user's problem | Phase 3 |
| Cloud deployment / hosting | Course requires only a local demo | Not planned for course |
| Fine-tuning or training custom models | Hosted foundation models with good prompts and tools are enough | Not planned |

## Risks

| Risk | Likelihood | Impact | Mitigation |
| --- | --- | --- | --- |
| Vision model mis-tags items (e.g., navy vs. black, wool vs. acrylic) | Medium | Medium | Fixed tag schema, user can edit every tag, test set of real photos to measure accuracy |
| Agent recommends items the user doesn't own (hallucination) | Medium | High | Agent may only reference items by ID returned from the closet search tool; server validates every ID before display |
| Weather API unavailable or trip dates beyond forecast range | Medium | Medium | Use climate averages for dates beyond 16 days; label the data source; cache results; degrade gracefully with a "weather unavailable" note |
| Hosted LLM cost or rate limits during development | Medium | Medium | Configurable model name, cache tagging results, small test closets, per-request token limits, mocked LLM in automated tests |
| Privacy: clothing photos and travel plans are sent to a hosted LLM provider | Low | Medium | Disclose in the UI; photos stay on the local disk; send only resized images; no personal identifiers in prompts |
| Prompt injection via pasted itinerary or invitation text | Low | Medium | Treat user documents as data inside delimited blocks; tools are read-only except explicit, user-confirmed writes |
| Scope creep from three use cases | Medium | High | All three use cases share one closet, one weather tool, and one agent core; strict IN/OUT tables |

## Open Questions

1. **Outfit composition rules**: What makes a valid outfit (e.g., top + bottom or one-piece + shoes, optional outerwear and accessories)? Should rules be hard-coded, left to the LLM, or both (LLM proposes, code validates)?
2. **Trips beyond the forecast window**: For trips more than 16 days away, is a historical climate average for those dates acceptable, and should the plan be flagged for a refresh closer to departure?
3. **Re-wear and laundry assumptions for trips**: How many times can each category be re-worn on a trip by default (e.g., jeans 3×, t-shirts 1×)? Should the user be able to set "laundry available on day N"?
4. **Luggage constraint**: Should the trip planner take a carry-on-only flag that tightens the packing list?
5. **How many outfit options**: 1 primary + 1 alternative for daily outfits, and up to 3 for occasions. Is that the right balance between choice and speed/cost?
6. **Buy list detail**: Should "consider buying" items include a price band and a generic web search link, or only a description? How do we avoid sounding like advertising?
7. **Itinerary formats**: Is pasted text + PDF/TXT enough for the MVP, or do we need image/screenshot itineraries (would reuse the vision model)?
8. **Feedback learning**: Should thumbs up/down only affect the next request (short-term context), or update the style profile over time?
9. **Unavailable items**: Should "in the laundry" status reset automatically after N days?
10. **Cost limits**: What is an acceptable maximum number of LLM calls/tokens per recommendation for our budget?
