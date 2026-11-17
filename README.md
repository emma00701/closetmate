# ClosetMate: AI Wardrobe and Packing Assistant

INFO 5940 Team Project, Fall 2026

## Team

| Full name | NetID |
| --- | --- |
| _TODO: Full Name_ | _TODO: netid_ |
| _TODO: Full Name_ | _TODO: netid_ |
| _TODO: Full Name_ | _TODO: netid_ |
| _TODO: Full Name_ | _TODO: netid_ |

## Tools and Methodology

- **AI coding agent:** Claude Code
- **Methodology:** AI-Driven Development Lifecycle (**AI-DLC v1.0.1**), with rules installed as `CLAUDE.md` (copy of `core-workflow.md`) and `.aidlc-rule-details/`
- **LLM inside the product:** Anthropic Claude API (hosted), default model `claude-opus-5-5` (vision + tool calling + structured outputs), called from a local Python app
- **Product stack:** Python 3.12 · uv · FastAPI · Jinja2 + HTMX · SQLite (SQLModel) · Open-Meteo weather API · pytest

## Repository Layout

| Path | Contents |
| --- | --- |
| `README.md` | This overview |
| `docs/vision.md` | AI-DLC input: vision document (what and why, MVP in/out, open questions) |
| `docs/technical-environment.md` | AI-DLC input: technical environment document (stack, prohibited libraries, security, example code) |
| `CLAUDE.md` | AI-DLC v1.0.1 core workflow rules for Claude Code |
| `.aidlc-rule-details/` | AI-DLC v1.0.1 detailed stage rules |
| `aidlc-docs/` | Created by AI-DLC during the workflow: `audit.md`, `aidlc-state.md`, and `inception/` artifacts |

## Project Overview

### 1. Problem and why an LLM fits

Every day, people decide what to wear with poor information. They forget what they own, guess at the weather, and fall back on the same few outfits. The problem is worse at two moments. Before a **trip**, a multi-city itinerary with mixed activities and changing weather leads to overpacking, forgotten essentials, and last-minute purchases. Before a **special occasion** like a wedding, interview, or themed party, an unfamiliar dress code ("cocktail", "business casual", "black tie optional") pushes people to buy something new without checking whether their closet already has a workable outfit. ClosetMate is for busy students and young professionals who own enough clothes but lack an easy way to use them well. It answers three questions from the user's *own* closet: *What should I wear today? What should I pack for this trip? What should I wear, or buy, for this event?*

An LLM is the right tool because each step involves messy, unstructured input that rule-based software handles badly. Clothing photos need visual understanding to become structured data (category, color, formality, warmth). Day plans ("lab, then an interview at 2, drinks after"), pasted itineraries, and event invitations are free-form text that must be read and turned into structured dates, places, and activities. Dress codes and style rules are fuzzy, culturally loaded concepts that an LLM can interpret and explain. Finally, a good recommendation means combining several facts (the forecast in each city, what is clean and available, what was worn yesterday, the dress code) and making trade-offs. That kind of multi-step reasoning over tool results is what an LLM agent does well and what a fixed decision tree does poorly. Existing weather apps don't know your closet, closet apps make you tag everything by hand, and generic chatbots give advice like "pack a light jacket" without knowing whether you own one.

### 2. Planned LLM integration

ClosetMate uses the LLM in three ways. **(a) Vision tagging:** when the user uploads a photo, Claude's vision capability returns schema-validated tags (category, colors, pattern, formality 1–5, warmth 1–5, seasons, description) through structured outputs, so the closet is searchable without manual data entry. The user can correct any tag. **(b) Structured extraction:** pasted or uploaded (PDF/TXT) itineraries and event descriptions are turned into validated Pydantic objects (cities, date ranges, activities, dress code), which the user confirms before planning. **(c) A tool-using agent** powers the daily outfit, trip planner, and occasion planner. Our code runs an agent loop on the Claude Messages API: the model decides which tools to call and in what order, our server runs them, and the model uses the results to build the plan. Planned tools: `search_closet`, `get_item_details`, `get_wear_history`, `get_user_profile` (local SQLite), and `geocode_location`, `get_weather_forecast`, `get_climate_normals` (the free Open-Meteo APIs; climate averages cover trips beyond the 16-day forecast window). A trip request might go: extract cities and dates → geocode each city → fetch weather per city and date → search the closet for warm, rain-ready, and dressy items → check re-wear limits → return a day-by-day outfit plan, a consolidated packing list, and a "consider buying" list for real gaps.

We plan to use the **Anthropic Claude API** with the official `anthropic` Python SDK and `claude-opus-5-5` as the default model, because it supports image input, tool use, and structured outputs in one model. The model ID is a configuration setting, so we can switch models without code changes. To keep the agent trustworthy, all agent tools are read-only. The agent may only reference clothing by IDs returned from `search_closet`, and the server checks every ID before showing it, so the product cannot recommend clothes the user doesn't own. Each recommendation shows the facts it used (for example, "Boston, Mar 3: 8°C, 70% chance of rain"). Owned items and to-buy items are labeled separately. Automated tests use a fake LLM client, so they run offline and cost nothing.

### 3. Scope and feasibility

By the final deadline (December 16), ClosetMate will be a working local web app (FastAPI + Jinja2/HTMX + SQLite) with these MVP features: (1) photo upload with AI auto-tagging; (2) closet browsing, filtering, and tag editing; (3) a style profile (home city, temperature sensitivity, style and avoid lists); (4) a weather- and plan-aware **daily outfit** recommendation with an alternative and explanations; (5) feedback, "swap this item", and a wear log to avoid repeats; (6) a **trip planner** that reads an itinerary and produces a day-by-day outfit plan, packing list, and gap list; (7) a **special-occasion planner** that interprets dress codes and suggests up to three closet outfits or explains what is missing; (8) a combined **shopping gap list** with reasons, price bands, and a generic search link; and (9) saved trip and occasion plans. Explicitly out of scope: user accounts, calendar sync, real retailer/product APIs or checkout, virtual try-on images, background removal, a mobile app, and cloud hosting (see `docs/vision.md`).

This is more than a single feature: it combines vision, document extraction, and a multi-tool agent across three use cases with persistent state. It is still achievable because all three use cases share one closet database, one weather integration, and one agent core. Only the prompts and output shapes differ. The external API we depend on (Open-Meteo) is free and needs no key. AI-DLC Inception will break the work into units (for example, closet and tagging, weather integration, agent core, daily outfits, trip planner, occasions and shopping list, and the UI shell) so we can build and test them in parallel during Construction. If time gets tight, the shopping list and saved plans are the first to simplify. The daily outfit and trip planner are the core of the demo.

### 4. Collaboration plan

> _TODO (team): confirm or edit this plan in your first meeting._

- **Meetings:** We meet **twice a week** for about 1–1.5 hours. One meeting is **in person** (on campus, after class) and one is **online over Zoom** with screen sharing. During Inception, every AI-DLC working session is a full-team meeting: one member (the "driver") runs Claude Code and shares their screen, and the whole team discusses each question file and approval before the driver records our shared answer.
- **Communication between meetings:** A team group chat (Slack/Discord/iMessage) for day-to-day questions, with replies expected within 24 hours. GitHub Issues track tasks and decisions, and short meeting notes go in a shared doc.
- **Sharing the work:** The driver role rotates by AI-DLC stage, so every member leads at least one stage: Requirements, User Stories, Application Design (led by our most technical members), and Units Generation. Everyone reviews each generated artifact before approval. In Construction, each member owns one or two units of work, develops on a feature branch, and opens a pull request reviewed by at least one teammate before merging to `main`. API keys stay in each member's local `.env` and are never committed.

## Running AI-DLC

Start Claude Code in the repository root and say:

```text
Using AI-DLC, I want to start a new project. Please read docs/vision.md and docs/technical-environment.md, then begin the AI-DLC workflow.
```

AI-DLC writes its artifacts to `aidlc-docs/`. For this milestone, the team stops after **Units Generation** is approved, before Construction.
