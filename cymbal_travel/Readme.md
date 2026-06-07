# 🌍 Cymbal Travel — ADK Multi-Agent Lab
Cymbal Travel is a premier travel agency that curates bespoke vacation experiences. This repository contains the AI agent suite that powers its intelligence pipeline — a collection of specialized agents built on Google's Agent Development Kit that work in concert to keep travel data accurate and up to date.

The pipeline spans three capabilities: a Travel Scout that searches the web for live events and destination updates, a Destination Verifier that validates and structures geographic data for downstream systems, and a Brochure Auditor — a multi-agent pipeline that checks marketing materials for factual errors and rewrites them with corrections applied.

## 📁 Project Structure

```
adk_project/
│
├── app_agent/                    # Root orchestrator
│   ├── __init__.py
│   └── agent.py                  # Registers all sub-agents
│
├── my_google_search_agent/       # Travel Scout
│   ├── __init__.py
│   └── agent.py                  # ← Add google_search tool here
│
├── geo_validator/                # Destination Verifier
│   ├── __init__.py
│   └── agent.py                  # ← Add DestinationSchema + output_schema here
│
├── llm_auditor/                  # Brochure Auditor
│   ├── __init__.py
│   └── agent.py                  # ← Add fixer_agent + SequentialAgent here
│
├── .env                          # API keys (never commit — see .env.example)
├── .env.example                  # Template for required environment variables
├── requirements.txt              # Python dependencies
└── README.md                     # You are here
```

## 🏗 Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Cymbal Travel ADK                       │
│                                                             │
│   User / Downstream System                                  │
│           │                                                 │
│           ▼                                                 │
│   ┌───────────────┐                                         │
│   │   app_agent   │  ◄── Orchestrator / Root Agent         │
│   │  (Dispatcher) │                                         │
│   └───────┬───────┘                                         │
│           │                                                 │
│     ┌─────┼──────────┐                                      │
│     ▼     ▼          ▼                                      │
│ ┌───────┐ ┌────────┐ ┌──────────────────────────────────┐  │
│ │google │ │  geo   │ │         llm_auditor              │  │
│ │search │ │valid.  │ │  ┌────────────┐  ┌────────────┐  │  │
│ │ agent │ │        │ │  │  checker   │─►│   fixer    │  │  │
│ │       │ │ Schema │ │  │   agent    │  │   agent    │  │  │
│ │ Tool: │ │ Output │ │  └────────────┘  └────────────┘  │  │
│ │Google │ │        │ │                                   │  │
│ │Search │ │        │ │   Sequential Multi-Agent Pipeline │  │
│ └───────┘ └────────┘ └──────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

## 🤖 Agent Directory

### 1. `my_google_search_agent` — Travel Scout

**Purpose:** Scours the web for live travel events, advisories, and destination updates for Cymbal Travel's research team.

**How it works (intended):**
- Accepts a natural-language query, e.g. `"festivals in Kyoto next month"`
- Calls the **Google Search tool** via ADK's built-in tool integration
- Returns a structured summary of results

```json
{
  "query": "cherry blossom festivals Japan April 2025",
  "results": [
    { "title": "Ueno Park Sakura Festival", "date": "2025-03-25", "url": "..." },
    ...
  ]
}
```

---

### 2. `geo_validator` — Destination Verifier

**Purpose:** Takes raw destination data (city, country, coordinates, travel advisory level) and returns a **verified, schema-compliant record** that downstream systems can safely ingest.

**How it works (intended):**
- Receives a destination description from the orchestrator
- Validates fields (country code, lat/lng range, advisory level enum)
- Returns a strict JSON object matching `DestinationSchema`

```json
{
  "city": "Paris",
  "country_code": "FR",
  "latitude": 48.8566,
  "longitude": 2.3522,
  "advisory_level": "low",
  "verified": true
}
```

---

### 3. `llm_auditor` — Brochure Auditor

**Purpose:** A **two-stage multi-agent pipeline** that (1) detects factual errors in marketing brochures and (2) rewrites the brochure with corrections applied.

**Architecture — Sequential Pipeline:**

```
Input brochure text
        │
        ▼
┌───────────────┐
│ checker_agent │  Reads brochure → outputs list of errors
└───────┬───────┘
        │  error_list passed via session state
        ▼
┌───────────────┐
│  fixer_agent  │  Reads brochure + error_list → outputs corrected brochure
└───────────────┘
        │
        ▼
Corrected brochure text
```

_Input:_
> "Visit the Eiffel Tower in London, standing 1,665 feet tall..."

_After checker:_
```json
[
  { "field": "city", "error": "Eiffel Tower is in Paris, not London", "suggestion": "Paris, France" },
  { "field": "height", "error": "Eiffel Tower is 1,083 ft, not 1,665 ft", "suggestion": "1,083 feet" }
]
```

_After fixer:_
> "Visit the Eiffel Tower in **Paris**, standing **1,083 feet** tall..."

---

### 4. `app_agent` — Orchestrator

**Purpose:** The root-level dispatcher that receives inbound requests, determines intent, and routes work to the appropriate specialist agent.

**Role in the pipeline:**
- Exposes a single entry point for all Cymbal Travel AI operations
- Delegates to `my_google_search_agent`, `geo_validator`, or `llm_auditor` based on task type
- Aggregates responses for the calling system

**Configuration:** Wired as the top-level `root_agent` in the ADK `Runner`. Sub-agents are registered as tools or via `AgentTool` wrappers depending on ADK version in use.

---

## ⚙️ Setup & Prerequisites

### Requirements

- Python 3.11+
- A Google Cloud project with the **Gemini API** or **Vertex AI** enabled
- A valid **Google Search API key** + **Custom Search Engine ID** (for `my_google_search_agent`)

### Install

```bash
# 1. Clone the repository
git clone https://github.com/<your-org>/cymbal-travel-adk-lab.git
cd cymbal-travel-adk-lab

# 2. Create and activate a virtual environment
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate

# 3. Install dependencies
pip install -r requirements.txt
```

### `requirements.txt` (minimum)

```
google-adk>=0.4.0
google-generativeai>=0.5.0
pydantic>=2.0.0
python-dotenv>=1.0.0
```

---

## 🔑 Environment Variables

Copy `.env.example` to `.env` and fill in your credentials:

```bash
cp .env.example .env
```

```dotenv
# .env.example

# Gemini / Vertex AI
GOOGLE_API_KEY=your_google_api_key_here
GOOGLE_CLOUD_PROJECT=your_gcp_project_id       # Only needed for Vertex AI

# Google Search (for my_google_search_agent)
GOOGLE_SEARCH_API_KEY=your_search_api_key_here
GOOGLE_SEARCH_ENGINE_ID=your_cx_id_here
```

> **Never commit `.env` to version control.** It is already listed in `.gitignore`.

---

## 🚀 Running the Agents

### Using the ADK CLI (recommended during development)

```bash
# Launch the ADK web UI for interactive testing
adk web

# Or run a specific agent directly
adk run app_agent
```

### Running individual agents for testing

```bash
# Test the Travel Scout
adk run my_google_search_agent --input "beach festivals in Bali in July"

# Test the Destination Verifier
adk run geo_validator --input "Santorini, Greece"

# Test the Brochure Auditor
adk run llm_auditor --input "$(cat sample_brochure.txt)"
```

### Expected test outputs

| Agent | Input | Expected output type |
|---|---|---|
| `my_google_search_agent` | `"surfing events Bali"` | JSON list of search results |
| `geo_validator` | `"Tokyo, Japan"` | `DestinationSchema` JSON object |
| `llm_auditor` | Brochure with deliberate errors | Corrected brochure text |

---
