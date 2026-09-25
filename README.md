# IP-SHAKTI Sahayak

A multilingual AI assistant for **Intellectual Property, Ayurveda, Traditional Knowledge, and related regulatory information**.

The application uses a **text-first RAG architecture**:

- Permanent PDF knowledge documents are converted to `.txt` locally.
- Only the generated text files are deployed.
- ChromaDB stores document embeddings.
- User-uploaded PDF/TXT/DOCX files are converted to text automatically and indexed into a separate ChromaDB collection.
- Gemini is the primary AI provider.
- Groq is used as a fallback provider.
- SerpApi is used to search Ayurveda centres.
- FastAPI serves both the API and the frontend.
- Railway hosts the deployed application.

---

# 1. Current Architecture

```text
                         ┌──────────────────────────┐
                         │        User Browser       │
                         │   frontend/index.html     │
                         └─────────────┬────────────┘
                                       │
                                       │ HTTPS
                                       ▼
                         ┌──────────────────────────┐
                         │        FastAPI API        │
                         │      backend/app.py       │
                         └─────────────┬────────────┘
                                       │
                ┌──────────────────────┼──────────────────────┐
                │                      │                      │
                ▼                      ▼                      ▼
       ┌────────────────┐    ┌──────────────────┐    ┌─────────────────┐
       │  RAG Search     │    │ Document Upload  │    │ Ayurveda Search │
       │ backend/rag.py  │    │ upload_ingest.py │    │ map_service.py  │
       └────────┬───────┘    └─────────┬────────┘    └────────┬────────┘
                │                      │                      │
                ▼                      ▼                      ▼
       ┌────────────────┐    ┌──────────────────┐    ┌─────────────────┐
       │   ChromaDB      │    │ PDF/TXT/DOCX     │    │    SerpApi      │
       │                 │    │ → extracted TXT  │    │ Google Maps API │
       │ ip_sakti_main   │    │ → chunks         │    └─────────────────┘
       │ ip_sakti_uploads│    │ → ChromaDB        │
       └────────┬───────┘    └──────────────────┘
                │
                ▼
       ┌───────────────────────────────┐
       │      Retrieved Context         │
       └──────────────┬────────────────┘
                      │
                      ▼
       ┌───────────────────────────────┐
       │ Gemini Primary / Groq Fallback│
       └──────────────┬────────────────┘
                      │
                      ▼
       ┌───────────────────────────────┐
       │ Multilingual AI Answer         │
       │ English / Bengali / Hindi      │
       └───────────────────────────────┘
```

---

# 2. Permanent Knowledge-Base Flow

The permanent knowledge base does **not** use PDFs directly on Railway.

```text
Original PDF
    │
    ▼
local_pdfs/<category>/
    │
    │ python prepare_data.py
    ▼
data/<category>/<document>.txt
    │
    │ backend/ingest.py
    ▼
Text chunking
    │
    ▼
ChromaDB
    │
    ▼
ip_sakti_main
```

This design avoids Git LFS PDF pointer problems and makes deployment much more reliable.

---

# 3. User Upload Flow

Users can upload:

- PDF
- TXT
- DOCX

The backend automatically processes the uploaded document.

```text
User uploads PDF/TXT/DOCX
        │
        ▼
FastAPI /api/upload
        │
        ▼
uploaded_files/
        │
        ▼
Text extraction
        │
        ▼
uploaded_text/
        │
        ▼
Chunking
        │
        ▼
ChromaDB
        │
        ▼
ip_sakti_uploads
        │
        ▼
Immediately searchable by RAG
```

For PDFs, `pypdf` extracts text.

For DOCX files, `python-docx` extracts paragraph text.

Image-only scanned PDFs require OCR and are not handled by the standard text extraction pipeline.

---

# 4. Project Structure

```text
IP_Shakti_Sahayak/
│
├── backend/
│   ├── __init__.py
│   ├── app.py
│   ├── rag.py
│   ├── ingest.py
│   ├── upload_ingest.py
│   └── map_service.py
│
├── frontend/
│   └── index.html
│
├── data/
│   ├── ayurveda/
│   ├── copyright/
│   ├── international/
│   ├── patents/
│   ├── regulations/
│   ├── trademarks/
│   └── traditional_knowledge/
│
├── local_pdfs/
│   ├── ayurveda/
│   ├── copyright/
│   ├── international/
│   ├── patents/
│   ├── regulations/
│   ├── trademarks/
│   └── traditional_knowledge/
│
├── uploaded_files/
├── uploaded_text/
├── chroma_db/
│
├── prepare_data.py
├── requirements.txt
├── .env
├── .env.example
├── .gitignore
├── railway.toml
├── Procfile
├── .python-version
└── README.md
```

---

# 5. ChromaDB Collections

The project uses two collections.

## Main Knowledge Base

```text
ip_sakti_main
```

Contains permanent knowledge extracted from the `.txt` files inside:

```text
data/
```

## User Uploaded Documents

```text
ip_sakti_uploads
```

Contains chunks generated from files uploaded by users.

This keeps permanent reference material separate from temporary/user-provided content.

---

# 6. Requirements

Recommended Python version:

```text
Python 3.12
```

Main packages:

```text
fastapi
uvicorn[standard]
python-dotenv
google-genai
groq
chromadb
pypdf
python-multipart
python-docx
pydantic
requests
```

Install everything with:

```powershell
pip install -r requirements.txt
```

---

# 7. Local Setup on Windows

Open PowerShell.

Move to the project folder:

```powershell
cd D:\IP_Shakti_Sahayak
```

Create a virtual environment:

```powershell
py -m venv venv
```

Activate it:

```powershell
.\venv\Scripts\Activate.ps1
```

If PowerShell blocks activation:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

Then activate again:

```powershell
.\venv\Scripts\Activate.ps1
```

Install dependencies:

```powershell
pip install -r requirements.txt
```

---

# 8. Environment Variables

Create:

```text
.env
```

Example:

```env
GEMINI_API_KEY=your_gemini_key
GROQ_API_KEY=your_groq_key
SERPAPI_API_KEY=your_serpapi_key

STORAGE_ROOT=.

AUTO_INGEST_ON_START=true
REBUILD_MAIN_ON_START=false

CHROMA_BATCH_SIZE=75
```

Do not commit `.env` to GitHub.

---

# 9. Preparing Permanent PDF Documents

Put original PDFs only inside:

```text
local_pdfs/
```

Example:

```text
local_pdfs/
├── patents/
│   ├── patents_act_1970_current.pdf
│   ├── patent_office_manual.pdf
│   └── wipo_patentscope_guide.pdf
│
├── trademarks/
│   ├── trade_marks_act_1999.pdf
│   ├── trade_marks_rules_2017.pdf
│   └── wipo_madrid_system_guide.pdf
│
├── ayurveda/
│   ├── evidence_base_of_ayurveda.pdf
│   ├── evidence_based_ayurvedic_practice.pdf
│   └── ayurveda_science_of_life.pdf
│
└── ...
```

Convert all PDFs to text:

```powershell
python prepare_data.py
```

Generated files will appear inside:

```text
data/
```

Example:

```text
data/patents/patents_act_1970_current.txt
data/ayurveda/evidence_base_of_ayurveda.txt
```

The converter preserves page markers:

```text
===== PAGE 1 =====
...

===== PAGE 2 =====
...
```

This allows source page information to remain available after conversion.

---

# 10. Build the Main ChromaDB Locally

Run:

```powershell
python backend\ingest.py
```

The current ingestion system uses batch indexing.

Default:

```text
CHROMA_BATCH_SIZE=75
```

Typical output:

```text
Found 18 TXT files.
[1/18] Indexing: ...
Added 75 chunks ...
Indexed: document.txt (...)
```

The main collection will be stored inside:

```text
chroma_db/
```

---

# 11. Run Locally

Start FastAPI:

```powershell
uvicorn backend.app:app --reload
```

Open:

```text
http://127.0.0.1:8000
```

API documentation:

```text
http://127.0.0.1:8000/docs
```

Health check:

```text
http://127.0.0.1:8000/health
```

Status:

```text
http://127.0.0.1:8000/api/status
```

---

# 12. Current API Status Format

Example:

```json
{
  "message": "IP-SHAKTI Sahayak API is running",
  "gemini_configured": true,
  "groq_configured": true,
  "serpapi_configured": true,
  "main_chunks": 379,
  "uploaded_chunks": 448
}
```

Meaning:

```text
main_chunks      = Permanent knowledge-base chunks
uploaded_chunks  = User-uploaded document chunks
```

---

# 13. API Endpoints

## Health Check

```text
GET /health
```

Example:

```json
{
  "status": "ok"
}
```

---

## Application Status

```text
GET /api/status
```

Returns:

- AI-provider configuration status
- SerpApi configuration status
- permanent ChromaDB chunk count
- uploaded-document chunk count

---

## Ask a Question

```text
POST /api/search
```

Example request:

```json
{
  "question": "What is a patent?",
  "language": "English"
}
```

Supported answer languages currently include:

```text
English
Bengali
Hindi
```

---

## Upload a Document

```text
POST /api/upload
```

Supported:

```text
.pdf
.txt
.docx
```

Processing:

```text
Upload
→ extract text
→ save extracted text
→ chunk
→ ChromaDB
→ searchable
```

---

## Search Ayurveda Centres

```text
POST /api/ayurveda-centres
```

Example:

```json
{
  "location": "Kolkata",
  "latitude": null,
  "longitude": null
}
```

This endpoint uses SerpApi.

---

# 14. RAG Search Flow

When the user asks a question:

```text
Question
   │
   ▼
Search ip_sakti_main
   │
   ├──────────────┐
   │              │
   ▼              ▼
Permanent KB   Uploaded KB
                 ip_sakti_uploads
   │              │
   └──────┬───────┘
          ▼
Merge results
          │
          ▼
Sort by Chroma distance
          │
          ▼
Select best chunks
          │
          ▼
Build RAG context
          │
          ▼
Gemini
          │
          ├── if success → answer
          │
          └── if failure → Groq
                              │
                              ▼
                           answer
```

The displayed relevance value is derived from Chroma distance:

```text
relevance = 1 / (1 + distance)
```

It is useful as a relative similarity indicator, but it is not a statistical probability.

---

# 15. AI Provider Flow

Primary:

```text
Gemini
```

Fallback:

```text
Groq
```

Flow:

```text
RAG context
    │
    ▼
Gemini request
    │
    ├── success → response
    │
    └── failure
           │
           ▼
         Groq
           │
           ▼
        response
```

The API response includes:

```json
{
  "provider": "Gemini"
}
```

or:

```json
{
  "provider": "Groq"
}
```

---

# 16. Railway Deployment Architecture

Production architecture:

```text
GitHub Repository
        │
        │ push
        ▼
Railway Deployment
        │
        ├── FastAPI
        ├── frontend/index.html
        ├── data/*.txt
        │
        └── Railway Volume
              │
              ├── chroma_db/
              ├── uploaded_files/
              └── uploaded_text/
```

Original permanent PDFs are **not** deployed.

Only `.txt` versions are deployed.

---

# 17. Railway Start Command

The production start command is:

```bash
uvicorn backend.app:app --host 0.0.0.0 --port $PORT
```

`railway.toml` example:

```toml
[build]
builder = "RAILPACK"

[deploy]
startCommand = "uvicorn backend.app:app --host 0.0.0.0 --port $PORT"
healthcheckPath = "/health"
healthcheckTimeout = 120
restartPolicyType = "ON_FAILURE"
restartPolicyMaxRetries = 10
```

---

# 18. Railway Environment Variables

Configure these in Railway:

```text
GEMINI_API_KEY=...
GROQ_API_KEY=...
SERPAPI_API_KEY=...

STORAGE_ROOT=/app/storage

AUTO_INGEST_ON_START=true
REBUILD_MAIN_ON_START=false

CHROMA_BATCH_SIZE=75
```

Do not store API keys in GitHub.

---

# 19. Railway Persistent Volume

Create a Railway volume mounted at:

```text
/app/storage
```

Runtime data will be stored in:

```text
/app/storage/chroma_db
/app/storage/uploaded_files
/app/storage/uploaded_text
```

This is important because deployed container filesystems may otherwise be replaced during redeployments.

---

# 20. Background Indexing

Permanent knowledge indexing runs in a background thread.

This prevents Railway from waiting several minutes before FastAPI starts.

Old behaviour:

```text
Start container
→ index all documents
→ wait several minutes
→ FastAPI starts
→ Railway may stop container
```

Current behaviour:

```text
Start container
→ FastAPI starts immediately
→ /health works
→ Railway considers app healthy
→ knowledge-base indexing continues in background
```

---

# 21. Important Railway Settings

Keep:

```text
AUTO_INGEST_ON_START=true
REBUILD_MAIN_ON_START=false
```

Do not leave this permanently enabled:

```text
REBUILD_MAIN_ON_START=true
```

because it deletes and rebuilds the permanent collection every deployment.

---

# 22. When to Use REBUILD_MAIN_ON_START=true

Use it temporarily when:

- permanent TXT knowledge files were changed
- documents were removed
- chunking logic was significantly changed
- you intentionally want a fresh main collection

Procedure:

```text
1. Set REBUILD_MAIN_ON_START=true
2. Redeploy
3. Wait for indexing to finish
4. Check /api/status
5. Confirm main_chunks > 0
6. Change REBUILD_MAIN_ON_START=false
7. Redeploy
```

---

# 23. Git Configuration

Recommended `.gitignore`:

```gitignore
.env

venv/
.venv/

__pycache__/
*.pyc
*.pyo
*.pyd

.DS_Store

local_pdfs/
*.pdf

chroma_db/
uploaded_files/
uploaded_text/
```

Important:

```text
local_pdfs/
```

must stay local.

Original PDFs should not be committed.

---

# 24. Why PDFs Are Not Stored in GitHub

Previously, large PDFs were tracked through Git LFS.

Railway received Git LFS pointer files like:

```text
version https://git-lfs.github.com/spec/v1
oid sha256:...
size ...
```

Those are not real PDF bytes.

`pypdf` then produced errors such as:

```text
invalid pdf header: b'versi'
EOF marker not found
Stream has ended unexpectedly
```

The permanent solution is:

```text
Original PDF
→ local only
→ convert to TXT
→ commit TXT
→ deploy TXT
```

---

# 25. Frontend API Configuration

Because the frontend and FastAPI backend are deployed together on Railway:

```javascript
const API_URL = "";
```

This makes API calls same-origin.

Examples:

```javascript
fetch(API_URL + "/api/search")
fetch(API_URL + "/api/upload")
fetch(API_URL + "/api/ayurveda-centres")
```

Do not use:

```text
http://127.0.0.1:8000
```

inside the deployed frontend.

---

# 26. Upload Storage

When a user uploads a document, the original file is stored under:

```text
STORAGE_ROOT/uploaded_files/
```

The extracted text is stored under:

```text
STORAGE_ROOT/uploaded_text/
```

On Railway:

```text
/app/storage/uploaded_files/
/app/storage/uploaded_text/
```

The text chunks are stored in:

```text
ip_sakti_uploads
```

---

# 27. Deploy Updates

After modifying the project:

```powershell
git add .
git commit -m "Update IP-SHAKTI Sahayak"
git push
```

Railway automatically redeploys from GitHub if automatic deployment is enabled.

---

# 28. Recommended Deployment Workflow

```text
1. Update code locally
2. Test locally
3. Convert any new permanent PDFs to TXT
4. Run local ingestion test
5. Confirm /api/status
6. Commit only code + TXT knowledge files
7. Push to GitHub
8. Railway redeploys
9. Check /health
10. Check /api/status
11. Test search
12. Test document upload
13. Test Ayurveda centre search
```

---

# 29. Adding a New Permanent Knowledge Document

Example:

```text
new_patent_document.pdf
```

Place it in:

```text
local_pdfs/patents/
```

Run:

```powershell
python prepare_data.py
```

Verify:

```text
data/patents/new_patent_document.txt
```

Push:

```powershell
git add data
git commit -m "Add patent knowledge document"
git push
```

For a complete rebuild:

```text
REBUILD_MAIN_ON_START=true
```

Redeploy once.

Then return:

```text
REBUILD_MAIN_ON_START=false
```

---

# 30. Adding a New User Upload

No code change is required.

The user selects:

```text
PDF / TXT / DOCX
```

and clicks:

```text
Upload & Index
```

The document is automatically converted, indexed, and made searchable.

---

# 31. Troubleshooting

## No PDFs found under local_pdfs

Check:

```powershell
Get-ChildItem .\local_pdfs -Recurse -Filter *.pdf
```

Also ensure:

```text
prepare_data.py
```

and:

```text
local_pdfs/
```

are inside the same project root.

---

## invalid pdf header: b'versi'

The file is a Git LFS pointer, not a genuine PDF.

Check:

```powershell
Get-Content "file.pdf" -TotalCount 1
```

A genuine PDF should begin with:

```text
%PDF-
```

A Git LFS pointer begins with:

```text
version https://git-lfs.github.com/spec/v1
```

Replace it with the real original PDF.

---

## Railway: Application failed to respond

Check Railway logs.

Make sure indexing runs in the background and the health route can respond immediately.

Verify:

```text
GET /health
```

returns:

```json
{
  "status": "ok"
}
```

---

## Upload failed

Check:

```text
POST /api/upload
```

through:

```text
/docs
```

Also confirm:

```text
python-multipart
pypdf
python-docx
chromadb
```

are installed.

---

## Main chunks are 0

Check:

```text
/api/status
```

If:

```json
"main_chunks": 0
```

verify that `.txt` files exist inside:

```text
data/
```

Then run locally:

```powershell
python backend\ingest.py
```

On Railway, temporarily set:

```text
REBUILD_MAIN_ON_START=true
```

Redeploy once.

---

## Uploaded chunks do not increase

Confirm:

```text
/api/upload
```

returns:

```json
{
  "searchable": true,
  "chunks_added": 10
}
```

Then check:

```text
/api/status
```

---

# 32. Current Working Production State

A healthy deployed application should show:

```json
{
  "message": "IP-SHAKTI Sahayak API is running",
  "gemini_configured": true,
  "groq_configured": true,
  "serpapi_configured": true,
  "main_chunks": 379,
  "uploaded_chunks": 448
}
```

Chunk counts will change when knowledge files or user uploads change.

---

# 33. Security Notes

Never commit:

```text
.env
API keys
passwords
private credentials
```

Store production secrets using Railway environment variables.

For user uploads, consider adding:

- file-size limits
- file-name sanitization
- MIME-type validation
- upload quotas
- automatic deletion policies

for a production-grade system.

---

# 34. RAG Behaviour

The AI prompt instructs the model to answer only from retrieved documents.

The system:

```text
User Question
→ semantic search
→ best Chroma chunks
→ context creation
→ Gemini
→ Groq fallback
→ response + retrieved sources
```

If the retrieved context is insufficient, the assistant should state that instead of inventing information.

---

# 35. Multilingual Support

The frontend allows:

```text
English
Bengali
Hindi
```

Retrieval still searches the Chroma knowledge base, while the answer-generation prompt instructs the AI to respond in the selected language.

---

# 36. Ayurveda Centre Search

The Ayurveda centre feature is independent from ChromaDB.

```text
User Location
→ FastAPI
→ SerpApi
→ Google Maps results
→ centre cards
→ map display
→ directions
```

Returned information can include:

- centre name
- address
- rating
- review count
- phone
- type
- website
- latitude
- longitude
- place ID
- directions

---

# 37. Production Summary

Final production architecture:

```text
                    GitHub
                      │
             code + data/*.txt
                      │
                      ▼
                   Railway
                      │
        ┌─────────────┴─────────────┐
        │                           │
        ▼                           ▼
     FastAPI                    Frontend
        │
        ├───────────┬───────────┬──────────────┐
        │           │           │              │
        ▼           ▼           ▼              ▼
     ChromaDB     Gemini       Groq          SerpApi
        │
   ┌────┴────┐
   │         │
   ▼         ▼
Main KB   Upload KB
379+      448+
chunks    chunks
```

Local-only permanent source documents:

```text
local_pdfs/
```

Deployed permanent documents:

```text
data/*.txt
```

Persistent Railway runtime data:

```text
/app/storage/chroma_db
/app/storage/uploaded_files
/app/storage/uploaded_text
```

---

# 38. Useful Commands

Activate environment:

```powershell
.\venv\Scripts\Activate.ps1
```

Convert permanent PDFs:

```powershell
python prepare_data.py
```

Rebuild local ChromaDB:

```powershell
python backend\ingest.py
```

Start local server:

```powershell
uvicorn backend.app:app --reload
```

Git push:

```powershell
git add .
git commit -m "Update IP-SHAKTI Sahayak"
git push
```

---

# 39. Project Purpose

IP-SHAKTI Sahayak is intended as an educational and knowledge-assistance platform.

It combines:

- Intellectual Property information
- Ayurveda knowledge
- Traditional Knowledge
- International IP resources
- User-provided document search
- Multilingual AI responses
- Ayurveda centre discovery

AI-generated responses should be treated as educational information. Legal or medical decisions should be verified with qualified professionals or official authorities.

---

## IP-SHAKTI Sahayak

**Multilingual AI Assistant for Intellectual Property & Ayurveda**

FastAPI + ChromaDB + Gemini + Groq + SerpApi + Railway
