# AI Code Reviewer

An AI-powered GitHub bot that automatically reviews pull requests. When a PR is opened or updated, the bot reads the diff, runs specialist review agents, and posts inline comments and a summary directly on GitHub. It learns recurring issues per repository over time.

> **Status:** 🚧 In development. See the [Roadmap](#roadmap).

## Features

- [ ] GitHub App with verified webhooks (HMAC SHA-256)
- [ ] Automatic review on PR opened, updated and reopened
- [ ] Inline comments and a summary review posted to the PR
- [ ] Local LLM inference with Llama via Ollama (no API cost, code stays private)
- [ ] Multi-agent review with LangGraph: static analysis, security, style, architecture
- [ ] Duplicate and low-confidence comment filtering
- [ ] Background processing with Redis and Celery, plus SHA-based deduplication
- [ ] Per-repo learning from merged PRs
- [ ] Jupyter notebook support (reviews code cells)
- [ ] LLM tracing with LangFuse
- [ ] Metrics with Prometheus and Grafana

## How it works

```
Developer opens a PR
        │
        ▼
GitHub webhook ──► FastAPI (verify HMAC, parse event)
        │
        ▼
Redis queue ──► Celery worker
        │
        ▼
Fetch diff from GitHub API + load past repo patterns
        │
        ▼
LangGraph agents (Static · Security · Style · Architecture)
        │
        ▼
Merge findings ► remove duplicates ► filter by confidence
        │
        ▼
Post inline comments + summary review to the PR
        │
        ▼
On merge: learner stores frequent issues in PostgreSQL
```

## Tech stack

| Area | Tools |
|---|---|
| API | Python, FastAPI, Pydantic |
| Queue | Redis, Celery |
| AI | Llama (via Ollama), LangGraph |
| Database | PostgreSQL, SQLAlchemy, Alembic |
| Observability | LangFuse, Prometheus, Grafana |
| DevOps | Docker, docker-compose, GitHub Actions |

## Project structure

```
ai-code-reviewer/
├── app/
│   ├── main.py
│   ├── config.py
│   ├── api/          # webhook routes
│   ├── github/       # app auth, GitHub client
│   ├── llm/          # LLM client abstraction (Ollama, fake)
│   ├── review/       # prompts, agents, parsing
│   ├── workers/      # Celery tasks
│   └── db/           # models and sessions
├── tests/
│   └── samples/      # sample flawed code for testing
├── docs/
├── .env.example
├── docker-compose.yml
└── requirements.txt
```

## Getting started

### Prerequisites

- Python 3.11+
- Docker Desktop
- [Ollama](https://ollama.com) with a Llama model pulled
- A GitHub account
- A [smee.io](https://smee.io) channel (for local webhooks)
- A LangFuse account (optional)

### 1. Clone and install

```bash
git clone https://github.com/<your-username>/ai-code-reviewer.git
cd ai-code-reviewer
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

### 2. Create a GitHub App

1. GitHub → Settings → Developer settings → GitHub Apps → New GitHub App
2. Permissions: **Pull requests** (read and write), **Contents** (read), **Metadata** (read)
3. Subscribe to the **Pull request** event
4. Set the webhook URL to your smee.io channel and choose a webhook secret
5. Generate a private key (`.pem`) and note the App ID
6. Install the app on a test repository

### 3. Configure environment

```bash
cp .env.example .env
```

| Variable | Description |
|---|---|
| `GITHUB_APP_ID` | Your GitHub App ID |
| `GITHUB_PRIVATE_KEY_PATH` | Path to the `.pem` file |
| `GITHUB_WEBHOOK_SECRET` | Secret used to verify webhooks |
| `LLM_PROVIDER` | `ollama` or `fake` |
| `OLLAMA_MODEL` | Model name to use |
| `OLLAMA_BASE_URL` | Default `http://localhost:11434` |
| `DATABASE_URL` | PostgreSQL connection string |
| `REDIS_URL` | Redis connection string |
| `LANGFUSE_PUBLIC_KEY` / `LANGFUSE_SECRET_KEY` | Optional tracing keys |

> Never commit `.env` or your private key.

### 4. Run

```bash
docker-compose up -d            # Redis and PostgreSQL
uvicorn app.main:app --reload   # API
celery -A app.workers worker -l info   # worker
npx smee-client --url <your-smee-url> --target http://localhost:8000/webhook
```

### 5. Try it

Open a pull request on the repository where the app is installed. The bot should comment within a minute.

## Testing

```bash
pytest
ruff check .
```

Tests use a fake LLM client, so they run without Ollama.

## Roadmap

- [ ] Week 1: project skeleton, CI, LLM client abstraction
- [ ] Week 2: GitHub App, webhooks, HMAC, authentication
- [ ] Week 3: first AI review, structured output, LangFuse tracing
- [ ] Week 4: Redis/Celery, deduplication, PostgreSQL
- [ ] Week 5: multi-agent LangGraph workflow, learning loop, evaluation
- [ ] Week 6: Docker, monitoring, documentation, demo

## Evaluation

_Results to be added: issues caught, false-positive rate and review time for single-agent vs multi-agent, and for different Llama model sizes._

## Limitations

- Local models are slower and less capable than hosted frontier models
- The bot runs only while the server and webhook tunnel are running
- Reviews are suggestions, not a replacement for human review

## Acknowledgements

Architecture inspired by common production designs for AI code-review bots.

## License

<Choose a license, e.g. MIT>