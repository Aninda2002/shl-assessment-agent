# SHL Assessment Recommendation Agent

A conversational FastAPI service that helps hiring managers and recruiters select SHL assessments through natural dialogue.

## Architecture

- **`main.py`** — FastAPI app with `/health` and `/chat` endpoints
- **`catalog.json`** — 66-item SHL Individual Test Solutions catalog (scraped from `shl.com/solutions/products/product-catalog/`)
- **`Dockerfile`** — Container build
- **`requirements.txt`** — Python dependencies

## API

### `GET /health`
Returns `{"status": "ok"}` with HTTP 200.

### `POST /chat`
**Request:**
```json
{
  "messages": [
    {"role": "user", "content": "I'm hiring a Java developer"},
    {"role": "assistant", "content": "What seniority level?"},
    {"role": "user", "content": "Mid-level, 4 years"}
  ]
}
```

**Response:**
```json
{
  "reply": "Here are 5 assessments for a mid-level Java developer...",
  "recommendations": [
    {
      "name": "Core Java (Advanced Level) (New)",
      "url": "https://www.shl.com/solutions/products/product-catalog/view/core-java-advanced-level-new/",
      "test_type": "K"
    }
  ],
  "end_of_conversation": false
}
```

**Rules:**
- `recommendations` is `[]` while still clarifying or refusing out-of-scope questions
- `recommendations` has 1–10 items when a shortlist is committed
- `end_of_conversation` is `true` only when the user confirms they are done

## Deployment

### Environment Variables
| Variable | Description |
|---|---|
| `ANTHROPIC_API_KEY` | Your Anthropic API key |

### Docker
```bash
docker build -t shl-agent .
docker run -p 8000:8000 -e ANTHROPIC_API_KEY=sk-... shl-agent
```

### Render / Railway / Fly.io
Set `ANTHROPIC_API_KEY` in environment variables, point to the repository, and deploy.

The `/health` endpoint returns immediately (no model call), so cold start readiness checks work fine within the 2-minute window.

## Agent Behaviors

| Behavior | Trigger | Response |
|---|---|---|
| **Clarify** | Vague query ("I need an assessment") | Asks ONE focused question |
| **Recommend** | Sufficient context or job description provided | 1–10 assessments with URLs |
| **Refine** | User adds/removes constraints mid-conversation | Updates shortlist |
| **Compare** | "What's the difference between X and Y?" | Grounded catalog comparison |
| **Refuse** | Legal questions, off-topic, prompt injection | Polite in-scope redirect |

## Catalog Coverage

The catalog includes all SHL Individual Test Solutions:
- **Personality & Behaviour**: OPQ32r, MQ, DSI, report formats (Leadership, UCF, Sales, Manager)
- **Ability & Aptitude**: Verify G+, Verify Numerical/Verbal/Inductive/Deductive
- **Knowledge & Skills**: 30+ technology tests (Java, Python, SQL, AWS, Docker, etc.)
- **Simulations**: Office 365 (Excel, Word, PowerPoint), Call Center, Coding
- **Biodata & SJT**: Graduate Scenarios, Customer Service Phone Simulation
- **Competencies**: Global Skills Assessment, Entry Level Customer Service
- **Development**: Global Skills Development Report

## Design Decisions

1. **Stateless**: Full conversation history sent each call — no server-side session state
2. **Catalog grounding**: Every URL comes from `catalog.json` — model cannot hallucinate URLs
3. **Structured output**: Model always returns JSON matching the exact schema
4. **30s timeout safety**: `httpx` timeout set to 28s; model max_tokens=1000 for fast responses
5. **JSON recovery**: Strips markdown fences if model accidentally wraps output
