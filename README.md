# EchoMind

EchoMind is an enterprise-grade intelligent customer support system.

Core flow:

```text
User Request
  -> FastAPI /chat
  -> MemoryManager: Redis + ChromaDB + user profile
  -> IntentRecognizer
  -> AgentOrchestrator: General / Technical / Billing
  -> LLM response
  -> Write back to Redis and ChromaDB
```

## Core Features

- Multi-agent routing: General, Technical, Billing; supports parallel collaboration.
- Hybrid intent recognition.
- Multi-layer memory: Redis working memory, ChromaDB episodic memory and user profile.
- RAG knowledge base: query rewriting, parallel recall, deduplication, LLM reranking.
- MCP tools: cache, retry, circuit breaker, fallback.
- Online monitoring and end-to-end evaluation.
- Dynamic Skills loaded from `skills/`.
- Docker Compose full stack: EchoMind, Redis, ChromaDB, Prometheus, Nginx.

## Project Structure

```text
EchoMind/
├── api/main.py
├── core/intent_recognizer.py
├── agents/agent_orchestrator.py
├── memory/conversation_memory.py
├── mcp/tool_manager.py
├── mcp/knowledge_base.py
├── monitor/performance_monitor.py
├── evaluation/evaluator.py
├── data/demo_docs/
├── docker-compose.yml
├── Dockerfile
├── requirements.txt
└── .env
```

## Prerequisites

- Docker
- Docker Compose
- Anthropic API Key or compatible API Key

## Configuration

```bash
cp .env.example .env
```

Minimum configuration:

```env
ANTHROPIC_API_KEY=your_api_key
```

Optional Anthropic-compatible provider, e.g. DeepSeek:

```env
ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic
ANTHROPIC_MODEL=deepseek-v4-pro
ANTHROPIC_API_KEY=your_deepseek_key
```

Redis and ChromaDB are usually configured by `docker-compose.yml`.

## Quick Start

```bash
docker compose up -d --build
docker compose ps
docker compose logs -f echomind
```

Health check:

```bash
curl http://localhost:8000/health
```

Swagger UI:

```text
http://localhost:8000/docs
```

Default ports:

| Service | Host Port | Purpose |
|---|---:|---|
| EchoMind API | 8000 | Main API |
| Nginx | 80 | Reverse proxy |
| ChromaDB | 8001 | Vector database |
| Redis | 6379 | Working memory |
| Prometheus | 9090 | Monitoring |

## API Overview

| Method | Path | Purpose |
|---|---|---|
| GET | `/health` | Health check |
| POST | `/chat` | Main chat pipeline |
| POST | `/search` | Knowledge base retrieval |
| GET | `/knowledge/stats` | Knowledge base chunk count |
| POST | `/knowledge/upload` | Upload `.txt`, `.md`, `.json` |
| GET | `/monitor` | Agent and tool metrics |
| GET | `/skills` | View loaded Skills |
| POST | `/skills/reload` | Reload Skills |
| POST | `/eval/run` | Run end-to-end evaluation |
| GET | `/docs` | Swagger UI |

## Testing

```bash
# 1. Start services
docker compose up -d --build

# 2. Health check
curl http://localhost:8000/health

# 3. Chat
curl -X POST http://localhost:8000/chat \
  -H "Content-Type: application/json" \
  -d '{"message": "I want to know the refund policy", "user_id": "demo_user", "conv_id": "demo_conv"}'

# 4. Knowledge base stats
curl http://localhost:8000/knowledge/stats

# 5. Import demo knowledge
curl -X POST http://localhost:8000/knowledge/upload \
  -F "file=@data/demo_docs/sample_knowledge.json"

# 6. Search
curl -X POST "http://localhost:8000/search?query=refund%20policy&top_k=3"

# 7. Monitor
curl http://localhost:8000/monitor

# 8. Evaluation
curl -X POST http://localhost:8000/eval/run
```

CLI mode:

```bash
docker compose run --rm echomind python api/main.py --cli
```

## Troubleshooting

- `/health` returns 503: check `ANTHROPIC_API_KEY`, Redis, ChromaDB, and logs.
- ChromaDB connection fails: check `docker compose ps chromadb` and `/api/v1/heartbeat`.
- Redis auth fails: ensure password is `echomind123` in `.env` and Compose.
- `/search` returns no results: check `/knowledge/stats`; upload demo docs if needed.
- User profile or episodic memory missing: wait for async update or reach compression threshold.

## Stop and Cleanup

```bash
docker compose stop
docker compose restart echomind
docker compose down
docker compose down -v
```
