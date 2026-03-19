# CorpusGraph — OpenClaw Skill

Document ETL and entity relationship engine for AI agents. This skill gives OpenClaw agents the ability to ingest documents, convert them into searchable structured data, extract entities, and query a relationship graph — capabilities that are impossible for an agent working with raw files alone.

## Why This Exists

An AI agent can't parse a Parquet file, OCR a scanned PDF, extract entity mentions from an email chain, and then map how all those entities connect across hundreds of documents. The formats are too varied, the processing too specialized, and the cross-referencing too large for any context window.

CorpusGraph handles all of that before the agent ever sees the data. It converts documents in 1,000+ formats into searchable, machine-readable data, automatically extracts 30+ entity types, and builds a relationship graph mapping connections across the entire corpus. Structured files (Parquet, CSV, ORC, JSON) become clean JSON arrays. Unstructured files (PDFs, emails, images) become searchable full text with extracted entities. The agent gets structured results through a simple API — not raw files it can't process.

## What the Agent Can Do

- **Ingest** documents in 1,000+ formats through automated ETL
- **Search** across all documents with full-text search and faceted filtering
- **Read structured data** — Parquet, CSV, ORC, JSON files returned as clean JSON arrays
- **Extract entities** — people, organizations, emails, phones, crypto addresses, dates, and 25+ more types
- **Map relationships** — find connection paths between any two entities across the corpus
- **Retrieve evidence** — get the specific documents where two entities co-occur
- **Monitor processing** — real-time pipeline status with per-endpoint readiness indicators

## Self-Teaching

The agent fetches the complete API guide after authenticating. The platform teaches the agent how to use it — no external documentation required.

## Setup

1. **Create an account** at [app1.ingestigate.com/agentic-registration](https://app1.ingestigate.com/agentic-registration) or log in if you already have one.
2. **Generate credentials** at [app1.ingestigate.com/search/agentic-token](https://app1.ingestigate.com/search/agentic-token).
3. **Set the environment variables** from the credential JSON:
   ```bash
   export INGESTIGATE_TOKEN="<access_token from credential JSON>"
   export INGESTIGATE_BASE_URL="<api_base_url from credential JSON>"
   ```

The agent will guide you through this process if you haven't set it up yet.

## Plans

| Plan | Agentic API Calls/Day | Price |
|------|----------------------|-------|
| Trial | 50 | Free for 14 days |
| Starter | 300 | $49/month |
| Professional | Unlimited | $1,999/month |
| Enterprise | Unlimited | Custom |

## Security

- **No persistent API keys.** Short-lived delegated sessions only (30-minute access tokens, 8-hour refresh). When the session ends, the credentials are worthless.
- **Organization-scoped data isolation.** Every agent action is scoped to the user's exact permissions. No cross-organization data leakage.
- **Full audit trail.** Every action the agent takes is traceable to a specific authenticated user.
- **MFA required.** All accounts use multi-factor authentication.
- **Processing-aware responses.** API responses include corpus readiness signals so agents never report conclusions from incomplete data.
- **Air-gapped deployment available** for on-premise and regulated environments.

## Links

- [CorpusGraph](https://ingestigate.com/corpusgraph)
- [Ingestigate Platform](https://ingestigate.com)
- [API Guide](https://app1.ingestigate.com/api/agent/guide) (requires authentication)

## License

This skill wrapper is licensed under [MIT-0](https://opensource.org/license/mit-0) (MIT No Attribution). The Ingestigate platform and API are proprietary.
