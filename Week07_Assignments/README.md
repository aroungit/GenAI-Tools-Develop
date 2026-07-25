# RAG MongoDB Demo - Intelligent Test Case & User Story Search System

## 📋 Project Overview

A full-stack **Retrieval-Augmented Generation (RAG)** application that enables intelligent search and retrieval of test cases and user stories using vector embeddings, keyword search (BM25), hybrid search, and AI-powered reranking. Built with React frontend and Node.js/Express backend, integrated with MongoDB Atlas Vector Search and AI models (Mistral AI for embeddings, Groq for reranking).

## 🎯 Key Features

- **📊 Data Management**: Excel to JSON conversion for test cases and user stories
- **🔢 Vector Embeddings**: Batch embedding generation using Mistral AI
- **🔍 Multi-Search Capabilities**:
  - Vector Search (semantic similarity)
  - BM25 Search (keyword-based)
  - Hybrid Search (combined scoring)
  - AI-Powered Reranking
- **⚙️ Query Processing**: 
  - Abbreviation expansion
  - Synonym mapping
  - Query normalization
- **📝 Results Processing**: 
  - Deduplication
  - Summarization
  - Schema-based formatting
- **🎨 Modern UI**: Material-UI based responsive interface
- **📈 Real-time Progress**: Job tracking for long-running operations

---

## 🛠️ Tech Stack

### **Backend**
| Technology | Purpose | Version |
|------------|---------|---------|
| **Node.js** | Runtime environment | Latest |
| **Express.js** | Web framework | 4.18.2 |
| **MongoDB Atlas** | Vector database & storage | 6.8.0 |
| **Mistral AI** | Embedding generation | 0.4.0 |
| **Groq SDK** | Reranking & summarization | 0.5.0 |
| **Multer** | File upload handling | 1.4.5 |
| **XLSX** | Excel file processing | 0.18.5 |
| **Axios** | HTTP client | 1.12.2 |
| **dotenv** | Environment configuration | 16.4.5 |
| **CORS** | Cross-origin resource sharing | 2.8.5 |
| **Concurrently** | Run multiple processes | 8.2.2 |

### **Frontend**
| Technology | Purpose | Version |
|------------|---------|---------|
| **React** | UI framework | 19.1.1 |
| **Material-UI (MUI)** | Component library | 7.3.2 |
| **MUI X Data Grid** | Data tables | 8.13.1 |
| **React Router** | Navigation | 7.9.3 |
| **Notistack** | Notifications | 3.0.2 |
| **Axios** | API communication | 1.12.2 |
| **Emotion** | CSS-in-JS styling | 11.14.0 |

### **AI/ML Services**
- **Mistral AI**: Embeddings (`mistral-embed` model)
- **Groq**: LLM reranking & summarization
- **MongoDB Atlas**: Vector search indexes

### **Development Tools**
- **Concurrently**: Parallel dev server execution
- **React Scripts**: Build & development
- **ESLint**: Code quality

---

## 🏗️ Architecture & Data Flow

### **Overall System Architecture**

```
┌─────────────────────────────────────────────────────────────┐
│                    CLIENT (React App)                        │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │Data Mgmt │  │ Search   │  │Processing│  │ Settings │   │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘   │
└───────┼─────────────┼─────────────┼─────────────┼──────────┘
        │             │             │             │
        │    HTTP/REST APIs (Port 3001)          │
        │             │             │             │
┌───────┼─────────────┼─────────────┼─────────────┼──────────┐
│       ▼             ▼             ▼             ▼           │
│              EXPRESS.JS SERVER (Backend)                    │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Upload → Convert → Embed → Store → Search → Process │  │
│  └──────────────────────────────────────────────────────┘  │
└───┬─────────────┬─────────────┬─────────────┬──────────────┘
    │             │             │             │
    ▼             ▼             ▼             ▼
┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐
│ MongoDB │  │Mistral  │  │  Groq   │  │  File   │
│  Atlas  │  │   AI    │  │   AI    │  │ System  │
└─────────┘  └─────────┘  └─────────┘  └─────────┘
```

### **Backend → Frontend Integration Flow**

#### **1️⃣ Data Ingestion Flow**
```
Excel File Upload
     │
     ▼
┌────────────────────────┐
│ POST /api/upload-excel │ ← Frontend: ConvertToJson.js
└────────┬───────────────┘
         │
         ▼
  Parse with XLSX lib
         │
         ▼
  Transform to JSON format
         │
         ▼
  Save to /src/data/*.json
         │
         ▼
  Return filename & stats
```

#### **2️⃣ Embedding Creation Flow**
```
Select JSON Files
     │
     ▼
┌────────────────────────────────┐
│ POST /api/create-embeddings-   │ ← Frontend: EmbeddingsStore.js
│      batch                      │
└────────┬───────────────────────┘
         │
         ▼
  Create Job ID
         │
         ▼
  Spawn batch script
  (create-embeddings-batch-mistral.js)
         │
         ▼
  For each document:
   ├── Generate embedding (Mistral AI)
   ├── Upsert to MongoDB
   └── Update job progress
         │
         ▼
  Return completion status
```

#### **3️⃣ Search Flow (Vector Search)**
```
User Query + Filters
     │
     ▼
┌────────────────────┐
│ POST /api/search   │ ← Frontend: QuerySearch.js
└────────┬───────────┘
         │
         ▼
  Generate query embedding
  (Mistral AI)
         │
         ▼
  MongoDB Vector Search
  ($vectorSearch pipeline)
         │
         ▼
  Apply metadata filters
         │
         ▼
  Return ranked results
  with similarity scores
```

#### **4️⃣ BM25 Keyword Search Flow**
```
Keyword Query
     │
     ▼
┌────────────────────────┐
│ POST /api/search/bm25  │ ← Frontend: BM25Search.js
└────────┬───────────────┘
         │
         ▼
  MongoDB Atlas Search
  ($search with text operator)
         │
         ▼
  Apply filters & sorting
         │
         ▼
  Return keyword-matched results
```

#### **5️⃣ Hybrid Search Flow**
```
Query + Search Strategy
     │
     ▼
┌─────────────────────────────┐
│ POST /api/search/hybrid     │ ← Frontend: HybridSearch.js
└────────┬────────────────────┘
         │
         ▼
  Execute in parallel:
   ├── Vector Search (semantic)
   └── BM25 Search (keyword)
         │
         ▼
  Merge results with weights:
   - RRF (Reciprocal Rank Fusion)
   - Weighted Average
   - Simple Average
         │
         ▼
  Return combined results
```

#### **6️⃣ Reranking Flow**
```
Initial Results + Query
     │
     ▼
┌─────────────────────────────┐
│ POST /api/search/rerank     │ ← Frontend: RerankingSearch.js
└────────┬────────────────────┘
         │
         ▼
  Send to Groq LLM
  (Relevance scoring)
         │
         ▼
  Parse JSON rankings
         │
         ▼
  Reorder by AI scores
         │
         ▼
  Return reranked results
```

#### **7️⃣ Query Preprocessing Flow**
```
Raw Query
     │
     ▼
┌──────────────────────────────┐
│ POST /api/search/preprocess  │ ← Frontend: QueryPreprocessing.js
└────────┬─────────────────────┘
         │
         ▼
  Apply transformations:
   ├── Normalize (lowercase, trim)
   ├── Expand abbreviations (EMR→Electronic Medical Records)
   ├── Add synonyms (doctor→physician)
   └── Extract entities
         │
         ▼
  Return enhanced query
```

#### **8️⃣ Summarization & Deduplication Flow**
```
Search Results
     │
     ├────────────────────────────┐
     │                            │
     ▼                            ▼
┌─────────────────────┐  ┌─────────────────────────┐
│ POST /api/search/   │  │ POST /api/search/       │
│   deduplicate       │  │   summarize             │
└────────┬────────────┘  └────────┬────────────────┘
         │                        │
         ▼                        ▼
  Group similar        Send to Groq LLM
  results by:          (Generate summary)
   - Title similarity           │
   - ID matching                ▼
   - Score proximity      Return concise
         │                summary text
         ▼
  Return unique results
```

---

## 📡 API Reference

### **Health & System APIs**

#### `GET /api/health`
**Purpose**: Server health check  
**Response**:
```json
{
  "status": "OK",
  "message": "Server is running"
}
```

#### `GET /api/env`
**Purpose**: Get current environment configuration  
**Response**:
```json
{
  "DB_NAME": "db_testcases",
  "COLLECTION_NAME": "test_cases",
  "VECTOR_INDEX_NAME": "vector_index",
  "BM25_INDEX_NAME": "testcases-bm25-index"
}
```

#### `POST /api/env`
**Purpose**: Update environment configuration  
**Request Body**:
```json
{
  "DB_NAME": "db_testcases",
  "COLLECTION_NAME": "test_cases",
  "VECTOR_INDEX_NAME": "vector_index"
}
```

---

### **Job Management APIs**

#### `GET /api/jobs/active`
**Purpose**: Get all active background jobs  
**Response**:
```json
{
  "jobs": [
    {
      "id": "job-1234567890-abc123",
      "status": "in-progress",
      "progress": 45,
      "total": 100,
      "currentFile": "testcases-batch-1.json"
    }
  ]
}
```

#### `GET /api/jobs/:jobId`
**Purpose**: Get specific job status  
**Response**:
```json
{
  "id": "job-1234567890-abc123",
  "status": "completed",
  "progress": 100,
  "total": 100,
  "results": [...],
  "startTime": "2026-07-25T10:00:00.000Z",
  "endTime": "2026-07-25T10:15:00.000Z"
}
```

---

### **Data Management APIs**

#### `GET /api/files`
**Purpose**: List all JSON files in data directory  
**Response**:
```json
[
  {
    "name": "converted-1784961452370.json",
    "path": "D:/GenAI_Tools/Day 07/rag-mongo-demo-v9/src/data/converted-1784961452370.json",
    "size": 245678,
    "modified": "2026-07-25T10:00:00.000Z",
    "type": "json"
  }
]
```

#### `POST /api/upload-excel`
**Purpose**: Upload and convert Excel file to JSON  
**Content-Type**: `multipart/form-data`  
**Request Body**:
```
file: <Excel file>
sheetName: "Testcases" (optional)
dataType: "testcases" | "userstories"
```
**Response**:
```json
{
  "success": true,
  "message": "Test cases file converted successfully",
  "outputFile": "converted-1784961452370.json",
  "output": "✅ Converted 150 rows..."
}
```

#### `GET /api/metadata/distinct`
**Purpose**: Get distinct values for filter dropdowns  
**Response**:
```json
{
  "success": true,
  "metadata": {
    "modules": ["Authentication", "Billing", "Reports"],
    "priorities": ["High", "Medium", "Low"],
    "risks": ["High", "Medium", "Low"],
    "types": ["Automated", "Manual"]
  }
}
```

---

### **Embedding Management APIs**

#### `POST /api/create-embeddings`
**Purpose**: Create embeddings (individual processing)  
**Request Body**:
```json
{
  "files": ["converted-1784961452370.json"]
}
```
**Response**:
```json
{
  "success": true,
  "jobId": "job-1234567890-abc123",
  "message": "Embedding creation started",
  "filesCount": 1
}
```

#### `POST /api/create-embeddings-batch`
**Purpose**: Create embeddings using batch script (faster)  
**Request Body**:
```json
{
  "files": ["converted-1784961452370.json"],
  "scriptName": "create-embeddings-batch-mistral.js",
  "jobType": "testcases"
}
```
**Response**:
```json
{
  "success": true,
  "jobId": "job-1234567890-abc123",
  "message": "Batch embedding creation started using create-embeddings-batch-mistral.js",
  "filesCount": 1,
  "jobType": "testcases"
}
```

---

### **Search APIs**

#### `POST /api/search`
**Purpose**: Vector similarity search  
**Frontend Component**: `QuerySearch.js`  
**Request Body**:
```json
{
  "query": "login functionality test cases",
  "limit": 10,
  "filters": {
    "module": "Authentication",
    "priority": "High",
    "risk": "Medium",
    "automationManual": "Automated"
  }
}
```
**Response**:
```json
{
  "results": [
    {
      "id": "TC-001",
      "title": "Verify user login with valid credentials",
      "description": "Test case to validate login...",
      "score": 0.8945,
      "module": "Authentication",
      "priority": "High"
    }
  ],
  "query": "login functionality test cases",
  "cost": 0.00001,
  "tokens": 12,
  "filters": {...}
}
```

#### `POST /api/search/bm25`
**Purpose**: Keyword-based search (BM25)  
**Frontend Component**: `BM25Search.js`  
**Request Body**:
```json
{
  "query": "login test",
  "limit": 10,
  "filters": {...}
}
```
**Response**: Similar to vector search

#### `POST /api/search/hybrid`
**Purpose**: Combined vector + keyword search  
**Frontend Component**: `HybridSearch.js`  
**Request Body**:
```json
{
  "query": "patient appointment booking",
  "limit": 10,
  "searchStrategy": "rrf", // "rrf" | "weighted" | "average"
  "vectorWeight": 0.7,
  "bm25Weight": 0.3,
  "filters": {...}
}
```
**Response**:
```json
{
  "results": [...],
  "metadata": {
    "strategy": "rrf",
    "vectorResults": 10,
    "bm25Results": 8,
    "mergedResults": 12
  }
}
```

#### `POST /api/search/rerank`
**Purpose**: AI-powered result reranking  
**Frontend Component**: `RerankingSearch.js`  
**Request Body**:
```json
{
  "query": "prescription management",
  "results": [...], // Initial search results
  "topK": 5
}
```
**Response**:
```json
{
  "rerankedResults": [...],
  "originalCount": 10,
  "rerankedCount": 5,
  "model": "openai/gpt-oss-120b"
}
```

#### `POST /api/search/user-stories`
**Purpose**: Search user stories collection  
**Request Body**:
```json
{
  "query": "user authentication",
  "limit": 10
}
```

#### `GET /api/testcases/latest-id`
**Purpose**: Get latest test case ID for ID generation  
**Response**:
```json
{
  "latestId": "TC-150",
  "nextId": "TC-151"
}
```

---

### **Query Processing APIs**

#### `POST /api/search/preprocess`
**Purpose**: Preprocess and enhance query  
**Frontend Component**: `QueryPreprocessing.js`  
**Request Body**:
```json
{
  "query": "EMR system for dr appointment"
}
```
**Response**:
```json
{
  "original": "EMR system for dr appointment",
  "processed": "electronic medical records system for doctor physician appointment scheduling",
  "transformations": {
    "normalized": "emr system for dr appointment",
    "abbreviationsExpanded": ["EMR→Electronic Medical Records"],
    "synonymsAdded": ["dr→doctor, physician"],
    "entities": ["EMR", "doctor", "appointment"]
  }
}
```

#### `POST /api/search/analyze`
**Purpose**: Analyze query for search suggestions  
**Request Body**:
```json
{
  "query": "login test"
}
```
**Response**:
```json
{
  "analysis": {
    "tokens": ["login", "test"],
    "suggestedFields": ["title", "description"],
    "estimatedResultCount": "50-100"
  }
}
```

---

### **Results Processing APIs**

#### `POST /api/search/deduplicate`
**Purpose**: Remove duplicate search results  
**Frontend Component**: `SummarizationDedup.js`  
**Request Body**:
```json
{
  "results": [...], // Array of search results
  "threshold": 0.9 // Similarity threshold
}
```
**Response**:
```json
{
  "deduplicatedResults": [...],
  "originalCount": 15,
  "uniqueCount": 12,
  "removedCount": 3
}
```

#### `POST /api/search/summarize`
**Purpose**: Generate AI summary of results  
**Frontend Component**: `SummarizationDedup.js`  
**Request Body**:
```json
{
  "results": [...],
  "query": "login functionality"
}
```
**Response**:
```json
{
  "summary": "Found 12 test cases related to login functionality covering authentication, password validation, session management, and security features...",
  "resultCount": 12
}
```

#### `POST /api/test-prompt`
**Purpose**: Test prompt schema formatting  
**Frontend Component**: `PromptSchemaManager.js`  
**Request Body**:
```json
{
  "schemaType": "testcase",
  "results": [...],
  "query": "login test"
}
```

---

## 📁 Project Structure & Impacted Files

### **File Organization**

```
rag-mongo-demo-v9/
│
├── 📦 Root Configuration
│   ├── package.json              # Backend dependencies & scripts
│   ├── .env                      # Environment variables (MongoDB, API keys)
│   └── README.md                 # This file
│
├── 🖥️ Backend (server/)
│   └── index.js                  # Express server & all API routes
│
├── 🎨 Frontend (client/)
│   ├── package.json              # Frontend dependencies
│   ├── public/                   # Static assets
│   ├── build/                    # Production build
│   └── src/
│       ├── App.js                # Main application component
│       ├── index.js              # React entry point
│       └── components/
│           ├── data/
│           │   ├── ConvertToJson.js         # Excel upload & conversion
│           │   └── EmbeddingsStore.js       # Embedding creation UI
│           ├── search/
│           │   ├── QuerySearch.js           # Vector search UI
│           │   ├── BM25Search.js            # Keyword search UI
│           │   ├── HybridSearch.js          # Combined search UI
│           │   └── RerankingSearch.js       # Reranking UI
│           ├── processing/
│           │   ├── QueryPreprocessing.js    # Query enhancement UI
│           │   ├── SummarizationDedup.js    # Result processing UI
│           │   └── PromptSchemaManager.js   # Schema management UI
│           └── settings/
│               └── Settings.js              # Configuration UI
│
├── 📊 Data & Scripts (src/)
│   ├── config/                   # Index configurations
│   │   ├── testcases-bm25-index.json
│   │   ├── testcases-vector-index.json
│   │   ├── user-stories-bm25-index.json
│   │   └── user-stories-vector-index.json
│   │
│   ├── data/                     # Generated JSON files
│   │   ├── converted-*.json      # Converted test cases
│   │   └── stories-*.json        # Converted user stories
│   │
│   └── scripts/
│       ├── data-conversion/
│       │   ├── excel-to-json.js              # Test case converter
│       │   ├── excel-to-userstories.js       # User story converter
│       │   └── fetch-jira-stories.js         # Jira integration
│       │
│       ├── embeddings/
│       │   ├── create-embeddings-batch-mistral.js        # Batch embeddings
│       │   └── create-userstories-embeddings-batch-mistral.js
│       │
│       ├── query-preprocessing/
│       │   ├── queryPreprocessor.js          # Main preprocessor
│       │   ├── abbreviationMapper.js         # Abbreviation expansion
│       │   ├── synonymExpander.js            # Synonym mapping
│       │   ├── normalizer.js                 # Text normalization
│       │   ├── dictionaries.js               # Medical/domain dictionaries
│       │   └── test-preprocessing.js         # Testing script
│       │
│       ├── search/
│       │   ├── search-vector-db.js           # Vector search script
│       │   ├── bm25-search.js                # BM25 search script
│       │   ├── score-fusion-search.js        # Hybrid search script
│       │   ├── rerank-search.js              # Reranking script
│       │   ├── search-combined-stores.js     # Multi-collection search
│       │   ├── search-jira-stories.js        # User story search
│       │   └── compare-field-weights.js      # Weight optimization
│       │
│       └── utilities/
│           ├── mistralEmbedding.js           # Mistral AI client
│           ├── groqClient.js                 # Groq AI client
│           └── delete-all-documents.js       # Cleanup utility
│
├── 📤 Uploads (uploads/)         # Temporary file uploads
│
└── 📋 Release Data (releases/)   # Sample user stories
    └── 1.21.2/stories/
```

---

## 🔑 Key Impacted Files Summary

### **Critical Backend Files**
| File | Lines | Purpose | Dependencies |
|------|-------|---------|--------------|
| `server/index.js` | ~2500 | Main API server, all routes | Express, MongoDB, Mistral, Groq |
| `src/scripts/utilities/mistralEmbedding.js` | ~150 | Embedding generation | Mistral AI API |
| `src/scripts/utilities/groqClient.js` | ~200 | Reranking & summarization | Groq SDK |

### **Critical Frontend Files**
| File | Lines | Purpose | API Endpoints Used |
|------|-------|---------|-------------------|
| `client/src/App.js` | ~400 | Main app layout & routing | None (container) |
| `client/src/components/data/ConvertToJson.js` | ~300 | File upload & conversion | `/api/upload-excel` |
| `client/src/components/data/EmbeddingsStore.js` | ~400 | Embedding management | `/api/create-embeddings-batch`, `/api/jobs/:jobId` |
| `client/src/components/search/QuerySearch.js` | ~500 | Vector search interface | `/api/search`, `/api/metadata/distinct` |
| `client/src/components/search/BM25Search.js` | ~450 | Keyword search interface | `/api/search/bm25` |
| `client/src/components/search/HybridSearch.js` | ~550 | Hybrid search interface | `/api/search/hybrid` |
| `client/src/components/search/RerankingSearch.js` | ~450 | Reranking interface | `/api/search/rerank` |
| `client/src/components/processing/QueryPreprocessing.js` | ~400 | Query enhancement | `/api/search/preprocess` |
| `client/src/components/processing/SummarizationDedup.js` | ~500 | Result processing | `/api/search/deduplicate`, `/api/search/summarize` |
| `client/src/components/settings/Settings.js` | ~350 | Configuration management | `/api/env` |

### **Batch Processing Scripts**
| Script | Purpose | Trigger |
|--------|---------|---------|
| `create-embeddings-batch-mistral.js` | Batch embedding creation | API: `/api/create-embeddings-batch` |
| `create-userstories-embeddings-batch-mistral.js` | User story embeddings | API: `/api/create-embeddings-batch` |

---

## 🚀 Setup & Installation

### **Prerequisites**
- Node.js 18+ and npm
- MongoDB Atlas account with Vector Search enabled
- Mistral AI API key
- Groq API key

### **Installation Steps**

1. **Clone and Install**
```bash
cd rag-mongo-demo-v9
npm install
```

2. **Configure Environment**
Create `.env` file in root:
```env
# MongoDB Configuration
MONGODB_URI=mongodb+srv://<username>:<password>@cluster0.xxx.mongodb.net/
DB_NAME=db_testcases
COLLECTION_NAME=test_cases
VECTOR_INDEX_NAME=vector_index
BM25_INDEX_NAME=testcases-bm25-index

# User Stories Collection
USER_STORIES_COLLECTION_NAME=user_stories
USER_STORIES_VECTOR_INDEX_NAME=user-stories-vector-index

# Mistral AI (Embeddings)
MISTRAL_API_KEY=your_mistral_api_key_here
MISTRAL_EMBEDDING_MODEL=mistral-embed

# Groq AI (Reranking & Summarization)
GROQ_API_KEY=your_groq_api_key_here
GROQ_RERANK_MODEL=openai/gpt-oss-120b
GROQ_SUMMARIZATION_MODEL=openai/gpt-oss-120b

# Server Configuration
PORT=3001
```

3. **MongoDB Atlas Setup**

Create Vector Search Index (testcases):
```json
{
  "fields": [
    {
      "type": "vector",
      "path": "embedding",
      "numDimensions": 1024,
      "similarity": "cosine"
    },
    {
      "type": "filter",
      "path": "module"
    },
    {
      "type": "filter",
      "path": "priority"
    }
  ]
}
```

Create Atlas Search Index (BM25):
```json
{
  "mappings": {
    "dynamic": false,
    "fields": {
      "title": {
        "type": "string",
        "analyzer": "lucene.standard"
      },
      "description": {
        "type": "string",
        "analyzer": "lucene.standard"
      }
    }
  }
}
```

4. **Run Application**

Development mode (both servers):
```bash
npm run dev
```

Separate terminals:
```bash
# Terminal 1 - Backend
npm run server

# Terminal 2 - Frontend
npm run client
```

Production build:
```bash
npm run build
```

5. **Access Application**
- Frontend: http://localhost:3000
- Backend API: http://localhost:3001
- API Health: http://localhost:3001/api/health

---

## 📖 Usage Examples

### **Example 1: Upload and Process Test Cases**

**Step 1**: Upload Excel File
```javascript
// Frontend: ConvertToJson.js
const formData = new FormData();
formData.append('file', excelFile);
formData.append('sheetName', 'Testcases');
formData.append('dataType', 'testcases');

const response = await axios.post(
  'http://localhost:3001/api/upload-excel',
  formData
);
// Response: { outputFile: "converted-1784961452370.json" }
```

**Step 2**: Create Embeddings
```javascript
// Frontend: EmbeddingsStore.js
const response = await axios.post(
  'http://localhost:3001/api/create-embeddings-batch',
  {
    files: ['converted-1784961452370.json'],
    scriptName: 'create-embeddings-batch-mistral.js',
    jobType: 'testcases'
  }
);
// Response: { jobId: "job-1234567890-abc123" }
```

**Step 3**: Monitor Progress
```javascript
const jobResponse = await axios.get(
  `http://localhost:3001/api/jobs/${jobId}`
);
// Response: { status: "in-progress", progress: 45, total: 100 }
```

### **Example 2: Vector Search**

```javascript
// Frontend: QuerySearch.js
const response = await axios.post('http://localhost:3001/api/search', {
  query: 'login functionality with 2FA',
  limit: 10,
  filters: {
    module: 'Authentication',
    priority: 'High'
  }
});

// Response:
// {
//   results: [
//     {
//       id: "TC-001",
//       title: "Verify 2FA login",
//       score: 0.8945,
//       module: "Authentication"
//     }
//   ],
//   cost: 0.00001,
//   tokens: 12
// }
```

### **Example 3: Hybrid Search with Reranking**

```javascript
// Step 1: Hybrid Search
const hybridResponse = await axios.post('http://localhost:3001/api/search/hybrid', {
  query: 'patient appointment scheduling',
  limit: 20,
  searchStrategy: 'rrf',
  vectorWeight: 0.7,
  bm25Weight: 0.3
});

// Step 2: Rerank Results
const rerankResponse = await axios.post('http://localhost:3001/api/search/rerank', {
  query: 'patient appointment scheduling',
  results: hybridResponse.data.results,
  topK: 10
});

// Final results optimized by AI
```

### **Example 4: Query Preprocessing**

```javascript
const response = await axios.post('http://localhost:3001/api/search/preprocess', {
  query: 'EMR system for dr appt'
});

// Response:
// {
//   original: "EMR system for dr appt",
//   processed: "electronic medical records system for doctor physician appointment scheduling",
//   transformations: {
//     abbreviationsExpanded: ["EMR→Electronic Medical Records", "dr→doctor", "appt→appointment"],
//     synonymsAdded: ["doctor→physician"]
//   }
// }
```

---

## 🔄 Complete Workflow Example

### **End-to-End: From Excel to Search Results**

```bash
# 1. Start the application
npm run dev

# 2. Navigate to "Data Management" → "Convert to JSON"
#    - Upload testcases.xlsx
#    - Select sheet: "Testcases"
#    - Click "Upload & Convert"
#    - Result: converted-1784961452370.json

# 3. Navigate to "Data Management" → "Embeddings Store"
#    - Select: converted-1784961452370.json
#    - Click "Create Embeddings (Batch)"
#    - Monitor progress in job tracker

# 4. Navigate to "Search" → "Vector Search"
#    - Enter query: "user authentication with SSO"
#    - Set limit: 10
#    - Apply filters: Module = "Authentication"
#    - Click "Search"
#    - View results with similarity scores

# 5. Navigate to "Processing" → "Query Preprocessing"
#    - Test query enhancement
#    - See abbreviation expansions
#    - Get synonym suggestions

# 6. Navigate to "Search" → "Hybrid Search"
#    - Try combined vector + keyword search
#    - Experiment with different strategies (RRF, weighted)

# 7. Navigate to "Search" → "Reranking"
#    - Select initial results
#    - Apply AI reranking
#    - Get optimized result order
```

---

## 🧪 Testing the APIs

### **Using cURL**

```bash
# Health check
curl http://localhost:3001/api/health

# Get metadata filters
curl http://localhost:3001/api/metadata/distinct

# Vector search
curl -X POST http://localhost:3001/api/search \
  -H "Content-Type: application/json" \
  -d '{
    "query": "login test",
    "limit": 5
  }'

# BM25 search
curl -X POST http://localhost:3001/api/search/bm25 \
  -H "Content-Type: application/json" \
  -d '{
    "query": "authentication",
    "limit": 5
  }'

# Hybrid search
curl -X POST http://localhost:3001/api/search/hybrid \
  -H "Content-Type: application/json" \
  -d '{
    "query": "user login",
    "limit": 10,
    "searchStrategy": "rrf"
  }'
```

---

## 🎯 Feature Highlights

### **1. Intelligent Search**
- **Vector Search**: Semantic similarity using Mistral embeddings
- **BM25 Search**: Traditional keyword matching with BM25 scoring
- **Hybrid Search**: Combines both approaches with configurable weights
- **AI Reranking**: Uses Groq LLM to reorder results by relevance

### **2. Query Enhancement**
- Abbreviation expansion (EMR → Electronic Medical Records)
- Synonym enrichment (doctor → physician, dr, medical professional)
- Entity extraction and normalization
- Domain-specific dictionaries (medical, technical)

### **3. Result Processing**
- Deduplication based on similarity threshold
- AI-powered summarization
- Schema-based formatting for different output types
- Metadata filtering (module, priority, risk, type)

### **4. Data Management**
- Excel to JSON conversion with field mapping
- Batch embedding generation with progress tracking
- Support for test cases and user stories
- Automatic field validation and transformation

### **5. Performance Optimization**
- Batch processing for embeddings (5-10x faster)
- Job tracking for long-running operations
- Rate limiting and retry logic for API calls
- Concurrent search execution for hybrid mode

---

## 📊 MongoDB Collection Schemas

### **Test Cases Collection**
```javascript
{
  _id: ObjectId,
  id: "TC-001",
  module: "Authentication",
  title: "Verify login with valid credentials",
  description: "Test case to validate user login...",
  steps: "1. Open login page\n2. Enter credentials\n...",
  expectedResults: "User successfully logged in",
  preRequisites: "Valid user account exists",
  automationManual: "Automated",
  priority: "High",
  risk: "Medium",
  createdBy: "QA Team",
  createdDate: "2026-01-15",
  lastModifiedDate: "2026-07-20",
  linkedStories: ["US-123", "US-124"],
  version: "1.0",
  type: "Functional",
  embedding: [0.123, 0.456, ...], // 1024-dimensional vector
  embeddingMetadata: {
    model: "mistral-embed",
    createdAt: ISODate("2026-07-25T10:00:00Z")
  }
}
```

### **User Stories Collection**
```javascript
{
  _id: ObjectId,
  key: "US-123",
  summary: "As a patient, I want to book appointments online",
  description: "Detailed user story description...",
  status: {
    name: "In Progress",
    category: "In Progress"
  },
  priority: {
    name: "High",
    id: "2"
  },
  assignee: {
    displayName: "John Doe",
    emailAddress: "john@example.com"
  },
  created: "2026-07-01T10:00:00Z",
  updated: "2026-07-25T10:00:00Z",
  components: ["Frontend", "Backend"],
  labels: ["patient-portal", "appointments"],
  storyPoints: 8,
  project: "Healthcare Platform",
  epic: "Patient Self-Service",
  acceptanceCriteria: "1. User can select date\n2. Confirmation email sent",
  embedding: [0.789, 0.234, ...],
  embeddingMetadata: {
    model: "mistral-embed",
    createdAt: ISODate("2026-07-25T10:00:00Z")
  }
}
```

---

## 🔒 Environment Variables Reference

| Variable | Required | Description | Example |
|----------|----------|-------------|---------|
| `MONGODB_URI` | ✅ | MongoDB Atlas connection string | `mongodb+srv://user:pass@cluster0.xxx.mongodb.net/` |
| `DB_NAME` | ✅ | Database name | `db_testcases` |
| `COLLECTION_NAME` | ✅ | Test cases collection | `test_cases` |
| `VECTOR_INDEX_NAME` | ✅ | Vector search index name | `vector_index` |
| `BM25_INDEX_NAME` | ✅ | BM25 search index name | `testcases-bm25-index` |
| `USER_STORIES_COLLECTION_NAME` | ✅ | User stories collection | `user_stories` |
| `USER_STORIES_VECTOR_INDEX_NAME` | ✅ | User stories vector index | `user-stories-vector-index` |
| `MISTRAL_API_KEY` | ✅ | Mistral AI API key | `sk_abc123...` |
| `MISTRAL_EMBEDDING_MODEL` | ❌ | Embedding model | `mistral-embed` (default) |
| `GROQ_API_KEY` | ✅ | Groq API key | `gsk_abc123...` |
| `GROQ_RERANK_MODEL` | ❌ | Reranking model | `openai/gpt-oss-120b` (default) |
| `GROQ_SUMMARIZATION_MODEL` | ❌ | Summarization model | `openai/gpt-oss-120b` (default) |
| `PORT` | ❌ | Backend server port | `3001` (default) |

---

## 🐛 Troubleshooting

### **Common Issues**

**1. MongoDB Connection Failed**
```
Error: MongoServerSelectionError
```
**Solution**: 
- Verify `MONGODB_URI` in `.env`
- Check MongoDB Atlas network access (add your IP)
- Ensure database user has correct permissions

**2. No Search Results**
```
Error: Collection is empty. Please create embeddings first.
```
**Solution**:
- Upload test cases via "Convert to JSON"
- Create embeddings via "Embeddings Store"
- Wait for embedding job to complete

**3. Vector Index Not Found**
```
Error: Search index 'vector_index' not found
```
**Solution**:
- Create Vector Search index in MongoDB Atlas
- Ensure index name matches `VECTOR_INDEX_NAME` in `.env`
- Wait 1-2 minutes for index to be ready

**4. API Key Errors**
```
Error: Mistral API error: 401 - Unauthorized
```
**Solution**:
- Verify `MISTRAL_API_KEY` in `.env`
- Check API key validity in Mistral dashboard
- Ensure sufficient API credits

**5. CORS Errors**
```
Access to fetch at 'http://localhost:3001' blocked by CORS
```
**Solution**:
- Backend CORS is configured for all origins in development
- If still occurring, restart backend server

---

## 📈 Performance Metrics

| Operation | Time (avg) | Cost (avg) |
|-----------|------------|------------|
| Excel Upload & Convert | 2-5 sec | Free |
| Single Embedding Generation | 0.5-1 sec | $0.00001 |
| Batch Embeddings (100 docs) | 30-60 sec | $0.001 |
| Vector Search | 100-300 ms | $0.00001 |
| BM25 Search | 50-150 ms | Free |
| Hybrid Search | 200-500 ms | $0.00001 |
| AI Reranking (10 docs) | 1-2 sec | $0.0001 |
| Summarization | 2-4 sec | $0.0002 |

---

## 🔮 Future Enhancements

- [ ] Implement user authentication (JWT)
- [ ] Add Redis caching for frequently searched queries
- [ ] Support for PDF/Word document ingestion
- [ ] Real-time collaboration features
- [ ] Advanced analytics dashboard
- [ ] Multi-language support
- [ ] Export search results to Excel/PDF
- [ ] Scheduled batch processing
- [ ] Query history and saved searches
- [ ] Custom schema templates
- [ ] Webhook integrations (Jira, Slack)

---

## 📝 License

This project is for demonstration purposes.

---

## 👥 Contributing

This is a demo project. For production use, consider:
- Adding comprehensive error handling
- Implementing authentication & authorization
- Setting up monitoring & logging (e.g., Winston, PM2)
- Adding unit & integration tests
- Implementing rate limiting
- Setting up CI/CD pipelines

---

## 📞 Support

For issues or questions:
1. Check the troubleshooting section above
2. Review API documentation
3. Check MongoDB Atlas and API provider dashboards
4. Review server logs in terminal

---

## 🎓 Learning Resources

### **MongoDB Atlas Vector Search**
- [MongoDB Vector Search Documentation](https://www.mongodb.com/docs/atlas/atlas-vector-search/)
- [Vector Search Tutorial](https://www.mongodb.com/docs/atlas/atlas-vector-search/vector-search-tutorial/)

### **Mistral AI**
- [Mistral Embeddings Documentation](https://docs.mistral.ai/capabilities/embeddings/)
- [API Reference](https://docs.mistral.ai/api/)

### **Groq**
- [Groq SDK Documentation](https://console.groq.com/docs/quickstart)

### **React & Material-UI**
- [React Documentation](https://react.dev/)
- [Material-UI Components](https://mui.com/material-ui/getting-started/)

---

**Built with ❤️ using RAG, MongoDB Atlas Vector Search, Mistral AI, and Groq**
