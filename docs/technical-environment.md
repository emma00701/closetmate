# Technical Environment: ClosetMate

This document tells AI-DLC which tools, libraries, and patterns to use when designing and building ClosetMate. It is greenfield: the repository has no application code yet.

## Language and Package Manager

| Item | Choice | Notes |
| --- | --- | --- |
| Language | **Python 3.12+** | Type hints required on all public functions |
| Package manager | **uv** | `uv add`, `uv run`, `uv sync`. No pip, poetry, pipenv, or conda |
| Project config | `pyproject.toml` | All tool config (ruff, mypy, pytest) lives here |
| Lockfile | `uv.lock` committed to Git | Teammates run `uv sync` to get identical environments |
| Frontend language | HTML + Jinja2 templates, minimal vanilla JavaScript | No Node.js build step, no TypeScript |

## Frameworks and Libraries

### Required

| Library | Purpose |
| --- | --- |
| **FastAPI** | Web framework: JSON API routes plus server-rendered HTML pages |
| **Uvicorn** | Local ASGI server (`uv run uvicorn closetmate.main:app --reload`) |
| **Pydantic v2** | Request/response models, LLM structured-output schemas, settings |
| **pydantic-settings** | Load configuration from environment variables / `.env` |
| **Jinja2** | HTML templates for the web UI |
| **HTMX** (single vendored JS file in `closetmate/static/`) | Partial page updates (upload, filter, "swap item") without a JS framework |
| **SQLModel** (on SQLAlchemy 2.x) + **SQLite** | Local persistence for closet items, profile, wear log, plans, buy list |
| **anthropic** (official Python SDK) | All LLM calls: vision tagging, structured extraction, and the tool-using agent |
| **httpx** | HTTP client for external tools (Open-Meteo weather, geocoding, climate) |
| **Pillow** | Validate, resize (max 1568 px long edge), and re-encode uploaded photos before storage and LLM calls |
| **pypdf** | Extract text from uploaded PDF itineraries |
| **python-multipart** | File uploads in FastAPI |

### Development and Testing

| Tool | Purpose |
| --- | --- |
| **pytest** | Test runner |
| **pytest-cov** | Coverage (target: ≥ 80% line coverage on `closetmate/` excluding templates) |
| **pytest-asyncio** | Async tests (`asyncio_mode = "auto"` in `pyproject.toml`) |
| **respx** | Mock httpx calls to Open-Meteo in tests |
| **FastAPI `TestClient`** | Endpoint tests |
| **ruff** | Linting and formatting (replaces black/flake8/isort) |
| **mypy** | Type checking (`strict = true` for `closetmate/services` and `closetmate/agent`) |

### Prohibited (Do NOT Use)

| Prohibited | Reason | Use instead |
| --- | --- | --- |
| `eval()`, `exec()`, `pickle` on any user- or LLM-provided data | Arbitrary code execution; LLM output and uploaded files are untrusted | `json` + Pydantic validation |
| LangChain, LlamaIndex, CrewAI, AutoGen | Heavy abstractions that hide prompts and tool calls; hard for the team to debug and explain | Official `anthropic` SDK with our own small agent loop in `closetmate/agent/` |
| OpenAI-compatible shims or wrappers around the Anthropic API | One provider, one SDK; shims drop features (structured outputs, tool details) | `anthropic` SDK |
| `requests` | Blocking; inconsistent with async FastAPI routes | `httpx` (async client) |
| Flask, Django | Project standardizes on FastAPI | FastAPI |
| React, Vue, Angular, Next.js, any Node build tooling | Adds a second toolchain; not needed for an MVP UI | Jinja2 templates + HTMX |
| Streamlit, Gradio | Limited control over multi-page flows and the UI; mixes UI and logic | FastAPI + Jinja2 |
| PostgreSQL, MySQL, MongoDB, Docker-hosted databases | Requires extra setup on every laptop; local single-user app | SQLite via SQLModel |
| pandas, numpy (for app logic) | Not needed; data volumes are tiny | Standard library |
| Paid or keyed weather APIs (OpenWeatherMap, WeatherAPI, etc.) | Need API keys and have quotas | Open-Meteo (free, no key) |
| Web scraping retailer sites | Terms of service and fragility | Generic item descriptions plus a search URL |
| pip, poetry, pipenv, conda | Project uses uv only | uv |
| black, flake8, isort, pylint | Replaced by ruff | ruff |

## LLM Provider and Models

| Item | Choice |
| --- | --- |
| Provider | **Anthropic Claude API** (hosted), called from the local app with the official `anthropic` Python SDK |
| Default model | `claude-opus-5-5`, used for vision tagging, itinerary/event extraction, and the agent loop. Supports image input, tool use, and structured outputs |
| Model configuration | Model ID and effort level are settings (`CLOSETMATE_MODEL`, `CLOSETMATE_EFFORT`) so the team can change models without code changes |
| Agent approach | Client-side tool use on the Messages API: our code runs an agent loop (`closetmate/agent/runner.py`) that sends tool definitions, executes tool calls locally against SQLite and Open-Meteo, and returns results until the model finishes. Max 10 tool iterations per request |
| Structured outputs | Tagging and extraction use `client.messages.parse(..., output_format=<PydanticModel>)` so responses are schema-validated |
| Testing | Automated tests never call the real API. The LLM client is injected as a dependency and replaced with a fake in tests |

### Agent Tools (planned)

| Tool | Data source | Purpose |
| --- | --- | --- |
| `search_closet` | SQLite | Filter available items by category, color, formality range, warmth range, season; returns item IDs + tags (never images) |
| `get_item_details` | SQLite | Full tags for specific item IDs |
| `get_wear_history` | SQLite | Items worn in the last N days |
| `get_user_profile` | SQLite | Style preferences, home city, temperature sensitivity, avoid list |
| `geocode_location` | Open-Meteo Geocoding API | City name → latitude/longitude/timezone |
| `get_weather_forecast` | Open-Meteo Forecast API | Daily min/max temperature, precipitation probability, wind, for dates ≤ 16 days ahead |
| `get_climate_normals` | Open-Meteo Historical/Archive API | Average conditions for the same calendar dates over recent years (for trips > 16 days ahead) |

All agent tools are **read-only**. Writes (saving a plan, logging wear, adding buy-list items) happen in service code after the user acts in the UI, never as an LLM tool call.

## Cloud Provider and Deployment Model

- **No cloud deployment.** ClosetMate runs **locally** on each teammate's laptop (macOS, Windows, or Linux).
- It runs as a local FastAPI web app: `uv run uvicorn closetmate.main:app --reload`, opened at `http://127.0.0.1:8000`.
- **Data stays local**: SQLite file at `./data/closetmate.db` and photos under `./data/images/`. Both are git-ignored.
- **External network calls** (outbound HTTPS only): the Anthropic API (LLM) and Open-Meteo (weather, geocoding, climate). No inbound exposure; the server binds to `127.0.0.1` only.
- No AWS/GCP/Azure services, containers, or infrastructure-as-code are required.

## Architecture and Project Structure

- **Style**: modular monolith with layers: `routes` (HTTP) → `services` (business logic) → `repositories` (SQLModel) and `agent` (LLM + tools) → `integrations` (Open-Meteo, Anthropic client).
- **API style**: JSON REST endpoints under `/api/v1/...` plus HTML page routes (`/`, `/closet`, `/today`, `/trips`, `/occasions`, `/shopping`) that render Jinja2 templates and call the same services.
- Routes contain no business logic; services contain no HTTP or SQL details.

```text
closetmate/
  main.py                 # FastAPI app factory, router registration
  config.py               # pydantic-settings Settings
  db.py                   # engine/session setup
  models/                 # SQLModel tables + Pydantic schemas
  repositories/           # data access
  services/               # tagging, outfits, trips, occasions, shopping
  agent/
    runner.py             # agent loop (tool use)
    tools.py              # tool definitions + handlers
    prompts.py            # system prompts
  integrations/
    llm.py                # thin wrapper around anthropic client (injectable)
    weather.py            # Open-Meteo client
  routes/
    api/                  # JSON endpoints
    pages/                # HTML endpoints
  templates/
  static/
tests/
  unit/
  integration/
  fixtures/               # sample photos, itineraries, fake LLM responses
data/                     # git-ignored
```

## Security Basics

| Area | Requirement |
| --- | --- |
| Authentication | None. Single-user local app bound to `127.0.0.1`. (Multi-user auth is out of MVP scope.) |
| Secrets | `ANTHROPIC_API_KEY` read from environment / `.env` via pydantic-settings. `.env` is git-ignored; a `.env.example` with placeholder values is committed. Keys are never logged, printed, or sent to the browser. |
| Input validation | All request bodies are Pydantic models with length limits (free text ≤ 5,000 chars; itinerary text ≤ 20,000 chars). Uploads: only JPEG/PNG/WebP images ≤ 10 MB and PDF/TXT ≤ 5 MB, verified by content (Pillow / pypdf), not just extension. Files are stored under server-generated UUID names, never user-supplied paths. |
| LLM output handling | LLM output is untrusted: parsed via Pydantic schemas; item IDs checked against the database before display; Jinja2 autoescaping on (never mark LLM text `|safe`). |
| Prompt injection | User-supplied documents are wrapped in clearly delimited blocks and described to the model as data. Agent tools are read-only, so injected instructions cannot modify data. |
| Privacy | Photos are resized and metadata (EXIF, including GPS) is stripped before storage and before sending to the LLM. The UI states that photos and plans are sent to the LLM provider. |
| Cost and abuse limits | Max 10 agent iterations and a `max_tokens` cap per call; weather responses cached in memory for 1 hour. |
| Dependencies | Only libraries listed above; `uv.lock` committed. |

## Testing

| Test type | Scope | Tooling |
| --- | --- | --- |
| Unit | Services, outfit validation, packing-list math, tool handlers, prompt builders | pytest, fake LLM client |
| Integration | API routes with a temporary SQLite database | pytest + FastAPI `TestClient`, respx for Open-Meteo |
| Agent scenario tests | Scripted fake-LLM tool-call sequences to check the loop, tool dispatch, iteration limit, and ID validation | pytest |
| Manual evaluation | Real-model runs on a fixed set of ~40 clothing photos, 10 itineraries, and 5 event descriptions, scored against the vision success metrics | Documented checklist in `tests/manual/` |

- `uv run pytest` must pass and `uv run ruff check .` / `uv run mypy closetmate` must be clean before merging to `main`.
- Tests must not require network access or an API key.

## Example Code Patterns

### Example endpoint

```python
# closetmate/routes/api/outfits.py
from datetime import date

from fastapi import APIRouter, Depends, HTTPException
from pydantic import BaseModel, Field

from closetmate.services.outfits import OutfitService, get_outfit_service
from closetmate.services.errors import ClosetEmptyError, WeatherUnavailableError
from closetmate.models.schemas import DailyOutfitResponse

router = APIRouter(prefix="/api/v1/outfits", tags=["outfits"])


class DailyOutfitRequest(BaseModel):
    day: date = Field(default_factory=date.today)
    plans: str = Field(default="", max_length=5000, description="What the user is doing that day")


@router.post("/daily", response_model=DailyOutfitResponse)
async def recommend_daily_outfit(
    request: DailyOutfitRequest,
    service: OutfitService = Depends(get_outfit_service),
) -> DailyOutfitResponse:
    try:
        return await service.recommend_daily(day=request.day, plans=request.plans)
    except ClosetEmptyError:
        raise HTTPException(status_code=409, detail="Add some clothes to your closet first.")
    except WeatherUnavailableError:
        raise HTTPException(status_code=503, detail="Weather service is unavailable. Try again shortly.")
```

### Example function (vision tagging service)

```python
# closetmate/services/tagging.py
import base64
from typing import Literal

from pydantic import BaseModel, Field

from closetmate.integrations.llm import LLMClient

Category = Literal["top", "bottom", "dress", "outerwear", "shoes", "accessory"]


class ItemTags(BaseModel):
    category: Category
    subcategory: str = Field(max_length=40, description="e.g. 'oxford shirt', 'chelsea boots'")
    primary_color: str = Field(max_length=20)
    secondary_colors: list[str] = Field(default_factory=list, max_length=3)
    pattern: Literal["solid", "striped", "plaid", "floral", "graphic", "other"]
    formality: int = Field(ge=1, le=5, description="1 = loungewear, 5 = black tie")
    warmth: int = Field(ge=1, le=5, description="1 = very light, 5 = heavy winter")
    seasons: list[Literal["spring", "summer", "fall", "winter"]]
    description: str = Field(max_length=200)


TAGGING_PROMPT = (
    "You are tagging a single clothing item for a personal digital closet. "
    "Describe only the main garment in the photo. Ignore the background and any person."
)


async def tag_clothing_photo(llm: LLMClient, image_bytes: bytes, media_type: str) -> ItemTags:
    """Ask the vision model for schema-validated tags for one clothing photo."""
    image_b64 = base64.standard_b64encode(image_bytes).decode("utf-8")
    return await llm.parse(
        output_format=ItemTags,
        content=[
            {"type": "image", "source": {"type": "base64", "media_type": media_type, "data": image_b64}},
            {"type": "text", "text": TAGGING_PROMPT},
        ],
    )
```

The `LLMClient` wrapper in `closetmate/integrations/llm.py` holds the `anthropic.AsyncAnthropic` client, and its `parse` method calls `client.messages.parse(model=settings.model, max_tokens=..., messages=[{"role": "user", "content": content}], output_format=output_format)` and returns `response.parsed_output`. Tests replace it with a fake.

### Example test

```python
# tests/unit/test_tagging.py
import pytest

from closetmate.services.tagging import ItemTags, tag_clothing_photo


class FakeLLM:
    def __init__(self, result: ItemTags) -> None:
        self.result = result
        self.calls: list[list[dict]] = []

    async def parse(self, output_format: type, content: list[dict]) -> ItemTags:
        self.calls.append(content)
        return self.result


@pytest.fixture
def navy_blazer() -> ItemTags:
    return ItemTags(
        category="outerwear",
        subcategory="unstructured blazer",
        primary_color="navy",
        pattern="solid",
        formality=4,
        warmth=2,
        seasons=["spring", "fall"],
        description="Navy cotton unstructured blazer",
    )


async def test_tagging_sends_image_and_returns_tags(navy_blazer: ItemTags) -> None:
    llm = FakeLLM(navy_blazer)

    tags = await tag_clothing_photo(llm, image_bytes=b"\x89PNG...", media_type="image/png")

    assert tags.category == "outerwear"
    assert llm.calls[0][0]["type"] == "image"
    assert llm.calls[0][0]["source"]["media_type"] == "image/png"


def test_formality_out_of_range_is_rejected() -> None:
    with pytest.raises(ValueError):
        ItemTags(
            category="top", subcategory="t-shirt", primary_color="white", pattern="solid",
            formality=9, warmth=1, seasons=["summer"], description="White tee",
        )
```
