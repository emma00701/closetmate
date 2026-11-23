# ClosetMate: AI Wardrobe and Packing Assistant

INFO 5940 Team Project, Fall 2026 · Mid-semester milestone: AI-DLC Inception phase

## Team

| Full name | NetID |
| --- | --- |
| _TODO: Full Name_ | _TODO: netid_ |
| _TODO: Full Name_ | _TODO: netid_ |
| _TODO: Full Name_ | _TODO: netid_ |
| _TODO: Full Name_ | _TODO: netid_ |

## At a Glance

| Item | Choice |
| --- | --- |
| AI coding agent | **Claude Code** (all team members) |
| AI-DLC version | **v1.0.1**: core rules in `CLAUDE.md` (copy of `core-workflow.md`), rule details in `.aidlc-rule-details/` |
| LLM inside the product | **Anthropic Claude API** (hosted), model `claude-opus-5-5`, called with the official `anthropic` Python SDK |
| External tool APIs | **Open-Meteo** forecast, geocoding, and historical climate APIs (free, no key) |
| Product stack | Python 3.12 · uv · FastAPI · Jinja2 + HTMX · SQLite (SQLModel) · pytest |
| How it runs | Locally on each laptop as a web app at `http://127.0.0.1:8000`; no cloud hosting |

## Project Overview

### 1. Problem and rationale for using LLMs

People who own plenty of clothes still struggle to decide what to wear. They forget what they own, guess at the weather, and repeat the same few outfits. The problem peaks at two moments. Before a **trip**, a multi-city itinerary with mixed activities and changing weather leads to overpacking, forgotten essentials, and last-minute purchases. Before a **special occasion** such as a wedding, interview, or gala, an unfamiliar dress code ("cocktail", "business casual", "black tie optional") pushes people to buy something new without checking whether their closet already works. ClosetMate is for busy students and young professionals. It answers three questions from the user's *own* closet: *What should I wear today? What should I pack for this trip? What should I wear, or buy, for this event?*

An LLM is the right tool because every input is unstructured and every decision is a judgment call. Clothing **photos** must become structured data (category, color, formality, warmth), which is a vision task. **Day plans, itineraries, and invitations** are free-form text that must become dates, places, activities, and dress codes. **Dress codes and style** are fuzzy, culturally loaded ideas that rules can't capture but an LLM can interpret and explain. Most importantly, a good recommendation means **combining live facts and making trade-offs**: the forecast in each city, what is clean and available, what was worn yesterday, the dress code, and luggage space. Existing tools each cover only part of this. Weather apps don't know your closet, closet apps make you tag everything by hand and use fixed rules, and general chatbots say "pack a light jacket" without knowing whether you own one.

### 2. Planned LLM integration

ClosetMate uses the LLM in three ways, all through the Anthropic Claude API (`claude-opus-5-5`, which supports images, tool use, and structured outputs; the model ID is a config setting so we can switch).

| Capability | Where it is used | How |
| --- | --- | --- |
| **Vision tagging** | Closet upload | Each photo returns schema-validated tags (category, colors, pattern, formality 1–5, warmth 1–5, seasons) via structured outputs. The user can correct any tag. |
| **Structured extraction** | Trip and occasion planners | Pasted or uploaded (PDF/TXT) itineraries and invitations become validated objects (cities, dates, activities, dress code), which the user confirms before planning. |
| **Tool-using agent** | Daily outfit, trip planner, occasion planner | The model decides which tools to call and in what order, our server runs them, and the model builds the plan from the results. |

**Why an agent and not a single prompt:** the facts the model needs depend on the request. A 3-city trip needs geocoding and weather for each city and date, then a closet search shaped by that weather. Weather for a trip more than 16 days away needs climate averages instead of a forecast. And a swap request needs wear history. One fixed prompt can't know all of this in advance. Our agent loop (built directly on the Messages API, with no agent framework) offers seven read-only tools: `search_closet`, `get_item_details`, `get_wear_history`, and `get_user_profile` over local SQLite, plus `geocode_location`, `get_weather_forecast`, and `get_climate_normals` over Open-Meteo. **Guardrails:** tools can't write data. The agent may only cite clothing IDs returned by `search_closet`, and the server checks every ID before display, so it cannot recommend clothes the user doesn't own. Each plan shows the facts it used (for example, "Boston, Mar 3: 8°C, 70% rain"). Owned items and to-buy items are labeled separately. The agent is capped at 10 tool calls per request. Automated tests use a fake LLM, so they run offline and cost nothing.

### 3. Project scope and feasibility

By **December 16** ClosetMate will be a working local web app with nine MVP features:

1. Photo upload with AI auto-tagging
2. Closet browsing, filtering, and tag editing
3. A style profile (home city, runs cold/warm, styles and colors to avoid)
4. A weather- and plan-aware **daily outfit**, with an alternative and explanations
5. Feedback, "swap this item", and a wear log that prevents repeats
6. A **trip planner** that turns an itinerary into a day-by-day outfit plan, a packing list that re-uses versatile pieces, and a gap list
7. An **occasion planner** that interprets dress codes and suggests up to three closet outfits or explains what is missing
8. A combined **shopping gap list** with reasons and a generic search link
9. Saved plans

**Out of scope:** accounts and login, calendar sync, real retailer APIs or checkout, virtual try-on images, background removal, a mobile app, and cloud hosting (full list with reasons in `docs/vision.md`).

This is more than one feature: it combines vision, document extraction, and a multi-tool agent across three use cases with persistent data. It stays achievable because the three use cases **share one closet database, one weather client, and one agent core**. Only the prompts and output formats differ. The only external API besides the LLM is free and needs no key, and we test against a fake LLM. Our plan:

| Dates | Work |
| --- | --- |
| Nov 17 – Nov 24 | AI-DLC Inception as a team: requirements, stories, application design, units of work (this milestone) |
| Nov 25 – Dec 2 | Construction of foundation units: closet + tagging, profile, weather client, agent core |
| Dec 3 – Dec 9 | Feature units: daily outfit, trip planner, occasion planner, shopping list, UI pages |
| Dec 10 – Dec 16 | Build and test, evaluation on our photo/itinerary test set, demo video and presentation |

If time runs short, we simplify saved plans and the shopping list first. The daily outfit and trip planner are the core of the demo.

### 4. Collaboration plan

- **Meeting frequency and format:** We meet **twice a week** for 1–1.5 hours: **in person** on campus after Tuesday class, and **online over Zoom** on weekends. During Inception, every AI-DLC session is a full-team meeting. One member (the driver) runs Claude Code and shares their screen, and the team agrees on each answer and approval before the driver records it.
- **Communication between meetings:** A team group chat for day-to-day questions (reply within 24 hours), **GitHub Issues** for tasks and decisions, and a shared notes doc for meeting minutes.
- **Sharing the work:** The driver role **rotates by AI-DLC stage**: Requirements Analysis, User Stories, Application Design (led by our most technical members), and Units Generation. Everyone reviews every generated document before we approve it. In Construction, each member **owns one or two units of work**, works on a feature branch, and opens a pull request that at least one teammate reviews before it merges into `main`. API keys stay in each member's local `.env` and are never committed.

## Repository Guide

| Path | What it is | Status |
| --- | --- | --- |
| `README.md` | Team, tools, project overview, collaboration plan | Done |
| `docs/vision.md` | AI-DLC input: vision (users, success metrics, MVP in/out, risks, open questions) | Done |
| `docs/technical-environment.md` | AI-DLC input: stack, LLM and tools, prohibited libraries, security, example endpoint/function/test | Done |
| `CLAUDE.md`, `.aidlc-rule-details/` | AI-DLC v1.0.1 rule files for Claude Code (official release, unmodified) | Done |
| `aidlc-docs/audit.md`, `aidlc-docs/aidlc-state.md` | AI-DLC record of every question, answer, and approval | Created during Inception |
| `aidlc-docs/inception/` | Requirements, user stories and personas, execution plan, application design, units of work | Created during Inception |

Application code will live at the repository root (`closetmate/`, `tests/`) during the final project, never inside `aidlc-docs/`.

## How We Run AI-DLC

From the repository root, in Claude Code:

```text
Using AI-DLC, I want to start a new project. Please read docs/vision.md and docs/technical-environment.md, then begin the AI-DLC workflow.
```

We answer every `[Answer]:` together, request changes when a proposal doesn't fit, reset the agent's context at each approval gate, and stop after **Units Generation** is approved, before Construction.
