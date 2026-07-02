# SHL Assessment Recommendation Agent

A conversational FastAPI service that helps hiring managers and recruiters select the most appropriate SHL assessments through natural language conversations using the OpenAI GPT API.

---

# Architecture

- **main.py** — FastAPI application exposing `/health` and `/chat`
- **catalog.json** — SHL Individual Test Solutions catalog used for grounded recommendations
- **requirements.txt** — Python dependencies
- **Dockerfile** — Docker container configuration (optional)

---

# API

## GET /health

Returns:

```json
{
  "status": "ok"
}
```

Status Code: **200**

---

## POST /chat

### Request

```json
{
  "messages": [
    {
      "role": "user",
      "content": "I'm hiring a Java developer"
    },
    {
      "role": "assistant",
      "content": "What seniority level?"
    },
    {
      "role": "user",
      "content": "Mid-level with 4 years of experience"
    }
  ]
}
```

### Response

```json
{
  "reply": "Based on your requirements, I recommend the following SHL assessments...",
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

---

## Response Rules

- `recommendations` is an empty array while clarification is required.
- Returns between **1 and 10** recommendations once sufficient hiring information is available.
- `end_of_conversation` becomes **true** only when the user explicitly confirms the conversation is complete.

---

# Environment Variables

| Variable | Description |
|----------|-------------|
| OPENAI_API_KEY | Your OpenAI API Key |

---

# Running Locally

Install dependencies

```bash
pip install -r requirements.txt
```

Set the API key

### Windows (PowerShell)

```powershell
$env:OPENAI_API_KEY="your_openai_api_key"
```

### Linux/macOS

```bash
export OPENAI_API_KEY="your_openai_api_key"
```

Run the server

```bash
uvicorn main:app --reload
```

API will be available at

```
http://localhost:8000
```

Swagger Documentation

```
http://localhost:8000/docs
```

---

# Docker

Build

```bash
docker build -t shl-agent .
```

Run

```bash
docker run -p 8000:8000 -e OPENAI_API_KEY=your_openai_api_key shl-agent
```

---

# Deployment

Deploy easily on

- Render
- Railway
- Fly.io

Configure the environment variable

```
OPENAI_API_KEY
```

before deploying.

---

# Agent Behaviors

| Behavior | Description |
|----------|-------------|
| Clarify | Asks one follow-up question when information is insufficient |
| Recommend | Suggests 1–10 SHL assessments with justification |
| Refine | Updates recommendations based on additional constraints |
| Compare | Compares SHL assessments using catalog information |
| Refuse | Politely declines unrelated or out-of-scope requests |

---

# Catalog Coverage

The catalog includes SHL Individual Test Solutions such as

- Personality & Behaviour
- Ability & Aptitude
- Knowledge & Skills
- Simulations
- Biodata & Situational Judgement
- Competencies
- Development Reports

---

# Design Decisions

1. Stateless architecture with complete conversation history sent in every request.
2. Recommendations are grounded entirely in the local SHL catalog.
3. Structured JSON responses ensure predictable API output.
4. OpenAI GPT is used for conversational reasoning while preventing hallucinated assessment URLs.
5. Automatic JSON parsing and validation before returning responses.

---

# Technologies Used

- FastAPI
- Python
- OpenAI API
- Pydantic
- Uvicorn
- JSON

---

# License

This project is intended for educational and assessment purposes.
