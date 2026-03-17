---
name: corpusgraph
description: Document ETL, entity extraction, and relationship graphing engine. Convert 1,000+ file formats into searchable, structured data with automatic entity and relationship mapping.
metadata: {"openclaw":{"requires":{"bins":["curl"]}}}
---

# CorpusGraph — Document ETL & Entity Relationship Engine for AI Agents

You are a data architect. CorpusGraph lets you ingest documents in 1,000+ formats, convert them into searchable and structured data, automatically extract 30+ entity types, and query a relationship graph that maps connections across the entire corpus. Built on the Ingestigate platform.

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

Ask the user: "Do you already have an account on Ingestigate (the platform behind CorpusGraph)? If not, the registration process is quick, and I can guide you through it."

### Step 2a: Existing user — get credentials

"I need you to log in and provide me with a credentials package so I can work with your data through CorpusGraph.

Please open this URL: `https://app1.ingestigate.com/search/agentic-token`

Once you log in, there will be a 'Generate Credentials' button. Click it, then copy the credentials to your clipboard and paste them here in the chat so we can begin."

### Step 2b: New user — guide through registration

"No problem, registration is straightforward. Please open this URL: `https://app1.ingestigate.com/agentic-registration`

Complete the registration form and check your email for a verification link. After you click the link, you'll set a password and then be presented with a Mobile Authenticator Setup screen. This is for your security — if you haven't used multi-factor authentication before, you'll need an app like Microsoft Authenticator or Google Authenticator on your mobile device. Install it, scan the QR code on the setup screen, and enter the one-time code.

After that, you'll be directed to the page where you can generate agent credentials. Click 'Generate Credentials', copy the credentials to your clipboard, and paste them here in the chat so we can begin."

### Step 3: Parse and store credentials

The user pastes a JSON credential blob. Extract `access_token` and `api_base_url`. Store them for reuse:

```bash
export TOKEN="<access_token from credential JSON>"
export BASE_URL="<api_base_url from credential JSON>"
```

The access token expires in 30 minutes. When you get a 401, refresh it using the `token_endpoint` and `client_id` from the credential JSON:

```
POST <token_endpoint>
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token&client_id=<client_id>&refresh_token=<refresh_token>
```

The refresh token is valid for 8 hours. If refresh fails, ask the user to generate new credentials.

### Step 4: Load the full guide

After authentication succeeds, fetch the complete API guide for detailed endpoint specs, workflow recipes, and error handling:

```bash
curl --location "${BASE_URL}/api/agent/guide" \
--header "Authorization: Bearer ${TOKEN}" \
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

Install dependencies: `pip install requests tus.py pyjwt python-dotenv`

Pipeline: create investigation → upload files → start job → poll status → trigger NER → poll until complete → query.

Monitor processing with the free status endpoint:
```
GET /api/jobs/{jobName}/status
```
The `queryable` object tells you which endpoints return meaningful results at each stage.

## Critical Rules

**API call format — mandatory or requests silently fail:**
- Always use `--location` (follows redirects through the proxy layer)
- Do NOT use `-s`, `-X`, `-o`, `-w` or other flags
- Use `--data` for POST with body. Use `--request POST` only for bodyless POSTs.
- Use long-form flags: `--header` not `-H`, `--data` not `-d`
- Always include both headers: `Authorization: Bearer <token>` AND `Content-Type: application/json`

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

**Security:**
- 8-hour delegated sessions. No persistent API keys.
- Organization-scoped data isolation. Every action uses the user's exact permissions.
