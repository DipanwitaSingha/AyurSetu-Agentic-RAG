<div align="center">

# 🌿 AyurSetu Agentic RAG

### Multilingual Agentic RAG Assistant for Intellectual Property, Ayurveda & Traditional Knowledge

<img src="https://img.shields.io/badge/Agentic%20RAG-Enabled-6d28d9?style=for-the-badge&logo=openai&logoColor=white">
<img src="https://img.shields.io/badge/FastAPI-Backend-009688?style=for-the-badge&logo=fastapi&logoColor=white">
<img src="https://img.shields.io/badge/ChromaDB-Vector%20DB-7c3aed?style=for-the-badge">
<img src="https://img.shields.io/badge/OCR-Tesseract-f97316?style=for-the-badge">
<img src="https://img.shields.io/badge/GitHub%20Actions-CI%2FCD-2088FF?style=for-the-badge&logo=githubactions&logoColor=white">
<img src="https://img.shields.io/badge/Railway-Deployment-111827?style=for-the-badge&logo=railway&logoColor=white">

**Connecting Traditional Wisdom with Intelligent Retrieval**

</div>

---
## 🏗️ AyurSetu Agentic RAG Architecture

![AyurSetu Agentic RAG Architecture](docs/AyurSetu_Agentic_RAG_Infographic.png)
## 🧠 About the Project

**AyurSetu Agentic RAG** is a multilingual AI assistant that combines **Agentic RAG, vector search, document processing, OCR and large language models**.

The system can answer questions from its indexed knowledge base about:

- 📘 Patents
- ™️ Trademarks
- ©️ Copyright
- 🌍 WIPO and International IP
- 🌿 Ayurveda
- 📜 Traditional Knowledge
- 📄 User-uploaded documents

Instead of sending every question directly to an LLM, the system first decides how the question should be handled. It can perform a simple retrieval or a deeper multi-query search before generating a source-grounded answer.

---

## 🧠 Agentic RAG Type

This project uses a **Controlled Agentic RAG architecture**.

The agent is responsible for:

- understanding the user's question
- identifying the query type
- planning one or more retrieval queries
- searching the permanent and uploaded document collections
- combining relevant context
- sending grounded context to Gemini or Groq
- generating follow-up questions

### Agentic RAG Flow

```text
User Question
      │
      ▼
Agent Router
      │
      ├── Simple Knowledge Query
      ├── Comparison Query
      ├── Multi-query Research
      └── Guided Topic Query
      │
      ▼
Query Planning
      │
      ▼
Vector Retrieval
      │
      ├── ip_sakti_main
      └── ip_sakti_uploads
      │
      ▼
Relevant Context
      │
      ▼
Gemini
      │
      └── Groq Fallback
      │
      ▼
Grounded Answer
      │
      ▼
Sources + Follow-up Questions
```

---

---

## 🧭 Agent Router and Agent Service

### `agent_router.py`

The **Agent Router** is the entry point for Agentic RAG requests. It receives the user question, validates the request, passes it to the agent logic, and returns the final response to the frontend.

```text
Frontend
   ↓
POST /api/agent-search
   ↓
agent_router.py
   ↓
agent_service.py
```

### `agent_service.py`

The **Agent Service** contains the main Agentic RAG logic. It decides how the question should be handled, whether a simple search or multi-query search is required, retrieves relevant ChromaDB context, calls Gemini or Groq, and prepares follow-up questions.

### Agent Flow

```text
User Question
      ↓
Agent Router
      ↓
Agent Service
      ↓
Question Analysis
      ↓
Query Planning
      ↓
Single Query / Multi Query
      ↓
ChromaDB Retrieval
      ↓
Relevant Context
      ↓
Gemini / Groq
      ↓
Grounded Answer
      ↓
Sources + Follow-up Questions
```

For a simple question such as:

```text
What is a patent?
```

the agent may use one direct retrieval query.

For a comparison question such as:

```text
What is the difference between a patent and a trademark?
```

the agent can break the task into multiple searches, retrieve evidence for each topic, and combine the results into one grounded answer.

> **In short:** `agent_router.py` handles the request flow, while `agent_service.py` handles the intelligent planning, retrieval, and answer-generation logic.

## ✨ Main Features

- 🧠 Controlled Agentic RAG
- 🌐 English, Bengali and Hindi
- 📚 ChromaDB vector search
- 📤 PDF, TXT and DOCX upload
- 🔎 Source-grounded answers
- ⚡ Exact and semantic caching
- 🤖 Gemini with Groq fallback
- 🌿 Ayurveda centre search
- 🧾 PDF-to-text conversion
- 🔤 OCR support for scanned documents
- 🚂 Railway deployment
- 🔁 GitHub Actions CI/CD

---

# 🏗️ Full Architecture

```text
                         ┌───────────────────────────┐
                         │       Web Frontend        │
                         │    HTML / CSS / JS        │
                         └─────────────┬─────────────┘
                                       │
                                       ▼
                         ┌───────────────────────────┐
                         │          FastAPI          │
                         │      backend/app.py       │
                         └─────────────┬─────────────┘
                                       │
               ┌───────────────────────┼───────────────────────┐
               │                       │                       │
               ▼                       ▼                       ▼
      ┌─────────────────┐    ┌──────────────────┐    ┌──────────────────┐
      │ Agent Router    │    │ Upload Pipeline  │    │ Ayurveda Centre │
      │ Agent Service   │    │ Background Job   │    │ Search / Map    │
      └────────┬────────┘    └─────────┬────────┘    └──────────────────┘
               │                       │
               ▼                       ▼
      ┌─────────────────┐    ┌──────────────────┐
      │ Query Planning  │    │ Text Extraction │
      │ Multi-query RAG │    │ OCR if required │
      └────────┬────────┘    └─────────┬────────┘
               │                       │
               └───────────┬───────────┘
                           ▼
              ┌──────────────────────────────┐
              │           ChromaDB           │
              │                              │
              │  ip_sakti_main               │
              │  ip_sakti_uploads            │
              └──────────────┬───────────────┘
                             │
                             ▼
              ┌──────────────────────────────┐
              │   Retrieved Source Context   │
              └──────────────┬───────────────┘
                             │
                    ┌────────┴────────┐
                    ▼                 ▼
               ┌─────────┐       ┌─────────┐
               │ Gemini  │       │  Groq   │
               │ Primary │       │Fallback │
               └────┬────┘       └────┬────┘
                    └────────┬─────────┘
                             ▼
              ┌──────────────────────────────┐
              │ Multilingual Final Answer   │
              │ Sources + Follow-up Queries │
              └──────────────────────────────┘
```

---

# 📄 PDF to Text Conversion

Permanent knowledge documents are first converted into text before ChromaDB ingestion.

### Flow

```text
PDF Files
   │
   ▼
local_pdfs/<category>/
   │
   ▼
prepare_data.py
   │
   ▼
Text Extraction
   │
   ▼
data/<category>/*.txt
   │
   ▼
backend/ingest.py
   │
   ▼
Chunking + Embeddings
   │
   ▼
ip_sakti_main
```

Example categories:

```text
local_pdfs/
├── ayurveda/
├── patents/
├── trademarks/
├── copyright/
├── international/
├── regulations/
└── traditional_knowledge/
```

Run conversion:

```powershell
python prepare_data.py
```

Then build the vector database:

```powershell
python -m backend.ingest
```

> For normal text-based PDFs, text can be extracted directly. Image-only or scanned PDFs need OCR before useful text can be indexed.

---

# 🔤 OCR Capability

AyurSetu Agentic RAG supports OCR-related document processing using:

- **Tesseract OCR**
- **PyMuPDF**
- **pytesseract**
- **Pillow**

### OCR Flow

```text
Scanned PDF
    │
    ▼
PDF Page Rendering
    │
    ▼
Image
    │
    ▼
Tesseract OCR
    │
    ▼
Extracted Text
    │
    ▼
Chunking
    │
    ▼
ChromaDB
```

For uploaded documents, the background processing pipeline can extract text and use OCR when required.

Check Tesseract locally:

```powershell
tesseract --version
```

Check Python OCR dependencies:

```powershell
python -c "import fitz; import pytesseract; from PIL import Image; print('OCR dependencies OK')"
```

---

---

## ⚡ Caching Strategy

AyurSetu Agentic RAG uses **two levels of persistent caching** to reduce repeated vector searches and AI calls.

### 1. Exact Cache

If the same question is asked again in the same language, the system first checks the exact cache.

```text
User Question
      ↓
Exact Cache Lookup
      ├── HIT  → Return cached answer
      └── MISS → Continue to semantic cache
```

This avoids unnecessary embedding generation, ChromaDB search and LLM calls for repeated questions.

### 2. Semantic Cache

If there is no exact match, the system creates the query embedding and checks whether a very similar question was answered before.

```text
New Question
      ↓
Create Query Embedding
      ↓
Semantic Cache Lookup
      ├── Similar cached question found
      │       ↓
      │   Return cached answer
      │
      └── No suitable match
              ↓
          ChromaDB Retrieval
              ↓
          Gemini / Groq
```

Successful responses are stored so they can be reused after later requests.

### Why caching is useful

- ⚡ Faster repeated responses
- 💰 Fewer AI API calls
- 🔎 Fewer unnecessary vector searches
- 📉 Lower processing load
- 💾 Persistent reuse across application restarts

---

## 📚 How Large PDFs and Many Pages Are Processed

Large PDFs are **not processed as one huge block**. The project processes documents in smaller stages so that large files are easier to extract, index and search.

### Permanent Knowledge Documents

```text
Large PDF
   ↓
Page-by-Page Text Extraction
   ↓
TXT File with Page Markers
   ↓
Page Separation
   ↓
Sentence-aware Chunking
   ↓
Small Text Chunks
   ↓
Batch Insert into ChromaDB
```

Each chunk keeps metadata such as:

```text
source
category
page
chunk number
```

This allows the RAG system to retrieve a small number of relevant sections instead of loading the entire PDF into the AI prompt.

### Chunking

Long page text is divided into smaller overlapping chunks.

Example:

```text
Page 25
   ↓
Chunk 1
Chunk 2
Chunk 3
...
```

Overlap helps preserve context when an important sentence or explanation crosses a chunk boundary.

### Batch Processing

Chunks are written to ChromaDB in batches rather than one at a time.

```text
Extract Pages
    ↓
Create Chunks
    ↓
Collect Batch
    ↓
ChromaDB Upsert
    ↓
Next Batch
```

This is more suitable for documents containing hundreds of pages.

---

## 📤 How Large Uploaded PDFs Are Handled

Uploaded files use a background document-processing workflow.

```text
User Uploads PDF
      ↓
File Saved
      ↓
Persistent Upload Job Created
      ↓
Background Worker
      ↓
Page Processing
      ↓
Text Extraction / OCR
      ↓
Chunking
      ↓
ChromaDB Upload Collection
      ↓
Document Becomes Searchable
```

The upload API returns a **job ID**, so the frontend does not need to wait for the entire document to finish processing.

The application can report processing information such as:

```text
current page
total pages
progress percentage
elapsed time
generated chunks
searchable status
```

This is especially useful for large PDFs with many pages.

### Why background processing is useful

- 📄 Large uploads do not block the main API request
- 🔄 Processing continues in the background
- 📊 The frontend can show live indexing progress
- 🔤 OCR can be used for pages where normal text extraction is insufficient
- 🔎 The document becomes searchable after indexing is completed

---

## 🔤 Large Scanned PDF + OCR Flow

For scanned/image-based pages, text extraction may require OCR.

```text
Large Scanned PDF
       ↓
Process Page
       ↓
Check Extracted Text
       ↓
Insufficient / No Text?
       │
       ├── No  → Use extracted text
       │
       └── Yes → Render page as image
                    ↓
                Tesseract OCR
                    ↓
                Extract text
       ↓
Combine Page Text
       ↓
Chunk
       ↓
ChromaDB
```

This lets the system handle a mixture of:

- normal searchable PDF pages
- scanned pages
- image-based documents
- large multi-page uploads

> Permanent-source PDF preprocessing and uploaded-document processing are separate workflows. Large uploaded documents use the background worker and progress-tracking pipeline, while permanent documents are prepared and then indexed into the main collection.

## 🛠️ Technology Stack

| Area | Technology |
|---|---|
| Frontend | HTML, CSS, JavaScript |
| Backend | FastAPI |
| Agentic Layer | Agent Router + Agent Service |
| AI | Google Gemini + Groq |
| Vector Database | ChromaDB |
| Embeddings | ChromaDB embedding function |
| PDF Processing | pypdf + PyMuPDF |
| OCR | Tesseract + pytesseract + Pillow |
| Document Upload | PDF / TXT / DOCX |
| Cache | SQLite + Semantic Cache |
| Deployment | Docker + Railway |
| CI/CD | GitHub Actions |
| Version Control | Git + GitHub |
| Language | Python |

---

# 🔁 GitHub Actions CI/CD Pipeline

GitHub Actions is used to automate project validation and deployment-related checks.

### CI/CD Flow

```text
Developer Changes Code
        │
        ▼
git push origin main
        │
        ▼
GitHub Repository
        │
        ▼
GitHub Actions
        │
        ├── Checkout Source Code
        ├── Setup Python
        ├── Install Dependencies
        ├── Run Syntax / Validation Checks
        └── Validate Build
        │
        ▼
Main Branch Updated
        │
        ▼
Railway Deployment
        │
        ▼
Docker Build
        │
        ▼
Health Check
        │
        ▼
Live Application
```

Workflow files are stored under:

```text
.github/workflows/
```

This provides a repeatable CI/CD pipeline whenever code is pushed to GitHub.

---

## 📂 Important Project Structure

```text
IP_Shakti_Sahayak/
├── .github/
│   └── workflows/
├── backend/
│   ├── app.py
│   ├── agent_router.py
│   ├── agent_service.py
│   ├── rag.py
│   ├── ingest.py
│   ├── upload_ingest.py
│   ├── worker.py
│   ├── semantic_cache.py
│   └── map_service.py
├── frontend/
│   └── index.html
├── data/
├── local_pdfs/
├── app/
│   └── storage/
├── prepare_data.py
├── requirements.txt
├── Dockerfile
├── railway.toml
└── README.md
```

---

# 💻 Download Project from GitHub to Local PC

Open **PowerShell**.

### 1. Go to the drive/folder where you want the project

```powershell
cd D:\
```

### 2. Clone the GitHub repository

```powershell
git clone https://github.com/amitava-0304/IP_Shakti_Sahayak.git
```

### 3. Enter the project directory

```powershell
cd D:\IP_Shakti_Sahayak
```

If the project already exists locally, update it with:

```powershell
git pull origin main
```

---

# 🧰 Local Setup

## 1. Create virtual environment

```powershell
python -m venv venv
```

## 2. Activate it

```powershell
venv\Scripts\Activate.ps1
```

If PowerShell blocks activation:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
venv\Scripts\Activate.ps1
```

Then:

```powershell
venv\Scripts\Activate.ps1
```

## 3. Install dependencies

```powershell
python -m pip install --upgrade pip
pip install -r requirements.txt
```

## 4. Create `.env`

Create:

```text
D:\IP_Shakti_Sahayak\.env
```

Example:

```env
GEMINI_API_KEY=your_gemini_api_key
GROQ_API_KEY=your_groq_api_key
SERPAPI_API_KEY=your_serpapi_api_key

STORAGE_ROOT=D:/IP_Shakti_Sahayak/app/storage
```

---

# 📚 Prepare and Index Knowledge Base

If PDFs need conversion:

```powershell
python prepare_data.py
```

Then create/rebuild ChromaDB:

```powershell
python -m backend.ingest
```

---

# ▶️ Run Locally

From:

```text
D:\IP_Shakti_Sahayak
```

run:

```powershell
python -m uvicorn backend.app:app --reload
```

Open:

```text
http://127.0.0.1:8000
```

FastAPI documentation:

```text
http://127.0.0.1:8000/docs
```

---

## 🔍 Example Questions

```text
What is a patent?
```

```text
What is the Madrid System?
```

```text
What is the difference between a patent and a trademark?
```

```text
What is traditional knowledge?
```

```text
Explain Ayurveda in Bengali.
```

---

# 🚀 Push Changes to GitHub

```powershell
git add .
git commit -m "Update AyurSetu Agentic RAG"
git pull --rebase origin main
git push origin main
```

---

## 🌐 Live Application

**AyurSetu Agentic RAG**

https://web-production-596c7.up.railway.app/

---

<div align="center">

## 🌿 AyurSetu Agentic RAG

**Connecting Traditional Wisdom with Intelligent Retrieval**

</div>
