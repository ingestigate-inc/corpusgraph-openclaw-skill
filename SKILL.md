---
name: corpusgraph
description: Document ETL, entity extraction, and relationship graphing engine. Convert 1,000+ file formats into searchable, structured data with automatic entity and relationship mapping.
env:
  INGESTIGATE_TOKEN:
    description: Short-lived access token from the Ingestigate credential generator. Expires in 30 minutes; refresh token valid for 8 hours. User generates at https://app1.ingestigate.com/search/agentic-token
    required: true
  INGESTIGATE_BASE_URL:
    description: API base URL from the credential JSON (e.g., https://app1.ingestigate.com). Provided alongside the token.
    required: true
---

# CorpusGraph — Document ETL & Entity Relationship Engine for AI Agents

Act as a data architect. CorpusGraph lets you ingest documents in 1,000+ formats, convert them into searchable and structured data, automatically extract 30+ entity types, and query a relationship graph that maps connections across the entire corpus. Built on the Ingestigate platform.

## When to Use This Skill

Use CorpusGraph when the user asks you to:
- Process a collection of documents into searchable, structured data
- Extract entities (people, organizations, emails, phones, addresses, crypto wallets, and 25+ more types) from files
- Find connections and co-occurrences across a document corpus
- Convert unstructured files (PDFs, emails, images) into queryable text
- Convert structured files (Parquet, ORC, CSV, JSON, XLSX) into clean JSON arrays
- Upload and process new files through an automated ETL pipeline

## Getting Started

### Step 1: Check if the user has credentials

Say this to the user: "Do you already have an account on Ingestigate (the platform behind CorpusGraph)? If not, the registration process is quick, and I can guide you through it."

### Step 2a: Existing user — get credentials

Say this to the user: "I need you to log in and provide me with a credentials package so I can work with your data through CorpusGraph.

Please open this URL: `https://app1.ingestigate.com/search/agentic-token`

Once you log in, there will be a 'Generate Credentials' button. Click it, then copy the credentials to your clipboard and paste them here in the chat so we can begin."

### Step 2b: New user — guide through registration

Say this to the user: "No problem, registration is straightforward. Please open this URL: `https://app1.ingestigate.com/agentic-registration`

Complete the registration form and check your email for a verification link. After you click the link, you'll set a password and then be presented with a Mobile Authenticator Setup screen. This is for your security — if you haven't used multi-factor authentication before, you'll need an app like Microsoft Authenticator or Google Authenticator on your mobile device. Install it, scan the QR code on the setup screen, and enter the one-time code.

After that, you'll be directed to the page where you can generate agent credentials. Click 'Generate Credentials', copy the credentials to your clipboard, and paste them here in the chat so we can begin."

### Step 3: Parse and store credentials

The user pastes a JSON credential blob. Extract `access_token` and `api_base_url`. Store them as environment variables for the duration of this session only:

```bash
export INGESTIGATE_TOKEN="<access_token from credential JSON>"
export INGESTIGATE_BASE_URL="<api_base_url from credential JSON>"
```

**Credential lifecycle:** The access token expires in 30 minutes. The refresh token is valid for 8 hours. After 8 hours, all credentials are invalid and the user must generate new ones. Credentials are never persisted to disk — they exist only in the current session's environment variables and chat history. This is by design: short-lived tokens limit blast radius if exposed.

When you get a 401, refresh the token using the `token_endpoint` and `client_id` from the original credential JSON:

```
POST <token_endpoint>
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token&client_id=<client_id>&refresh_token=<refresh_token>
```

If refresh fails, ask the user to generate new credentials.

### Step 4: Load the full guide

After authentication succeeds, fetch the complete API guide for detailed endpoint specs, workflow recipes, and error handling:

```bash
curl --location "${INGESTIGATE_BASE_URL}/api/agent/guide" \
--header "Authorization: Bearer ${INGESTIGATE_TOKEN}" \
--header 'Content-Type: application/json'
```

Read Sections 1-3 immediately. Reference Section 6 for specific endpoint specs as needed.

## Core Workflows

### Workflow A: Explore an Existing Corpus

**1. See what data is available:**
```
GET /api/discover/collections
```
Returns investigations and jobs with document counts.

**2. Get a corpus overview — entity counts and top entities:**
```
POST /api/dashboard/entity-stats
Body: { "limit": 50 }
```
Returns entity counts by type, top entities ranked by document frequency, and totals. Report this to the user: "Your corpus contains X documents. The system extracted Y entities across Z types. Top entities: [list]."

**3. Search across all documents:**
```
POST /api/search-faceted
Body: { "query": "quarterly revenue", "filters": {}, "page": 1, "pageSize": 10 }
```
Returns results with highlights, facets (people, organizations, locations, file types), and pagination.

**4. Read a document — text content:**
```
POST /api/file-details
Body: { "dataSourceName": "elasticsearch", "jobNames": ["<collection>"], "selectedFile": { "docId": "<docId>" }, "format": "clean_text" }
```
Use for PDFs, emails, DOCX, PPTX, TXT, HTML, and most document types.

**5. Read a document — structured/tabular content:**
```
POST /api/file-details-structured
Body: { "dataSourceName": "elasticsearch", "jobNames": ["<collection>"], "selectedFile": { "docId": "<docId>" } }
```
Use for CSV, Parquet, ORC, JSON files. Returns `rawContent` as parsed JSON arrays — clean key-value pairs the agent can query directly. For XLSX, try this endpoint first; fall back to `file-details` if structured content is empty.

**6. Search entities across the corpus:**
```
POST /api/entities/search
Body: { "query": "acme", "entity_types": ["Organization"], "limit": 50 }
```

**7. Map relationships between entities:**
```
POST /api/graph/paths
Body: { "entities": [{"type":"Person","value":"john doe"},{"type":"Organization","value":"acme corp"}], "maxBridgeNodes": 20 }
```

**8. Retrieve source documents for a connection:**
```
GET /api/graph/edge-evidence?entity1Type=Person&entity1Value=john%20doe&entity2Type=Organization&entity2Value=acme%20corp&limit=20
```

### Workflow B: Upload & Process New Files

Upload requires Python scripts. Fetch them from the API:
```
GET /api/agent/scripts                        — script catalog
GET /api/agent/scripts/create-investigation   — create an investigation first
GET /api/agent/scripts/tus-upload-engine      — core TUS uploader
GET /api/agent/scripts/batch-upload-template  — orchestrator (customize for user's files)
```

Install dependencies: `pip install requests tuspy pyjwt python-dotenv`

Pipeline: create investigation → upload files → start job → poll status → trigger NER → poll until complete → query.

Monitor processing with the free status endpoint:
```
GET /api/jobs/{jobName}/status
```
The `queryable` object tells you which endpoints return meaningful results at each stage.

## Critical Rules

**API call format — mandatory or requests silently fail:**
- Always use `--location` (the API sits behind an authentication reverse proxy that may issue redirects for HTTPS enforcement and path normalization — `--location` ensures these are followed correctly)
- Do NOT use `-s`, `-X`, `-o`, `-w` or other flags
- Use `--data` for POST with body. Use `--request POST` only for bodyless POSTs.
- Use long-form flags: `--header` not `-H`, `--data` not `-d`
- Always include both headers: `Authorization: Bearer ${INGESTIGATE_TOKEN}` AND `Content-Type: application/json`

**Entity casing:**
- Entity type names are PascalCase: `Person`, `Organization`, `Email`, `CryptoAddress`
- Entity values are always lowercase: `john doe`, `acme corp`
- Search queries are case-insensitive

**Anti-hallucination:**
- If a response includes `processing_status.corpus_ready: false`, results may be incomplete. Tell the user.
- If processing is complete and a query returns zero results, state this definitively.
- Only make claims based on data returned by the API. Never guess.

**Choosing the right read endpoint:**
- PDFs, emails, DOCX, PPTX, TXT, HTML → `file-details` with `"format": "clean_text"`
- CSV, Parquet, ORC, JSON → `file-details-structured` (returns JSON arrays)
- XLSX → try `file-details-structured` first, fall back to `file-details`

**Credential handling and security:**
- **No persistent API keys.** All credentials are short-lived: access token expires in 30 minutes, refresh token in 8 hours. After expiry, credentials are worthless.
- **Session-only storage.** Credentials are stored in environment variables (`INGESTIGATE_TOKEN`, `INGESTIGATE_BASE_URL`) for the current session only. They are never written to disk or persisted beyond the session.
- **User-initiated credential flow.** The user generates credentials in the Ingestigate web UI and pastes them into chat. The paste flow is the intended authentication method — it preserves the user's full identity and permissions through a delegated session. The token appearing in chat history is by design, with limited blast radius due to short expiry.
- **Organization-scoped data isolation.** Every API call executes with the user's exact permissions. No cross-organization data access is possible.
- **MFA required.** All Ingestigate accounts require multi-factor authentication.
