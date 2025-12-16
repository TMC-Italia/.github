# Competence Mapping AI - Hackathon Tasks

**Duration**: 4 Hours  
**Goal**: RAG-based CV competence matching system with zero data leakage (fully local)

## Architecture Overview

**Services (Docker Compose)**:
1. **Frontend Service** - Simple UI (React/shadcn OR plain HTML/JS if time-limited)
2. **Backend API** (`fastapi_backend`) - FastAPI orchestrator with RAG logic
3. **LLM Runtime** (`ollama`) - Llama 3.1 8B model server (port 11434)
4. **Vector Database** (`chromadb`) - Persistent vector store for CV embeddings
5. **Docker Volumes** - Persist `ollama_models`, `chroma_data`, and `/curricula` mapping

**Data Flow**: PDF Upload → Text Extraction → Chunking → Embedding → ChromaDB Storage → Query → RAG Retrieval → LLM Analysis → Results

---

## Pre-Event Checklist

- [ ] Ollama installed with Llama 3.1 8B (`ollama pull llama3.1:8b`)
- [ ] 5-10 dummy CV PDFs in `research/test_data/`
- [ ] Dependencies: `chromadb`, `pymupdf`, `python-multipart`, `ollama`, `sentence-transformers`
- [ ] Docker & Docker Compose installed

---

## Task 1: Docker Compose Services Setup (30 min)

**Goal**: Configure all services with proper networking and persistence.

**Implementation**:

Create/update `docker-compose.yml`:

```yaml
services:
  # Backend API Service
  fastapi_backend:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8000:8000"
    depends_on:
      - chromadb
      - ollama
    environment:
      - CHROMA_HOST=chromadb
      - CHROMA_PORT=8000
      - OLLAMA_HOST=http://ollama:11434
    volumes:
      - ./research/test_data:/app/curricula:ro  # Map CV folder

  # ChromaDB Vector Store
  chromadb:
    image: chromadb/chroma:latest
    ports:
      - "8001:8000"
    volumes:
      - chroma_data:/chroma/chroma
    environment:
      - IS_PERSISTENT=TRUE
      - ANONYMIZED_TELEMETRY=FALSE

  # Ollama LLM Runtime
  ollama:
    image: ollama/ollama:latest
    ports:
      - "11434:11434"
    volumes:
      - ollama_models:/root/.ollama
    # Uncomment for GPU support
    # deploy:
    #   resources:
    #     reservations:
    #       devices:
    #         - driver: nvidia
    #           count: all
    #           capabilities: [gpu]

volumes:
  chroma_data:      # Persistent vector embeddings
  ollama_models:    # Persistent LLM models
```

**Acceptance Criteria**:

- [ ] All 3 services defined in docker-compose.yml
- [ ] Persistent volumes for ChromaDB and Ollama
- [ ] `/curricula` volume mounted for CV access
- [ ] Environment variables configured
- [ ] Services start with `docker compose up -d`
- [ ] Health check: Ollama at `localhost:11434`, ChromaDB at `localhost:8001`

---

## Task 2: PDF Processing & Embedding Pipeline (45 min)

**Goal**: Extract text from PDFs, chunk, and generate embeddings using local models.

**Why**: Need to convert CVs into searchable vectors. Using `sentence-transformers` integrated in backend.

**Implementation**:

Create `src/app/services/ingestion.py`:

```python
import fitz  # PyMuPDF
from sentence_transformers import SentenceTransformer
from typing import List, Dict
import chromadb

class IngestionService:
    def __init__(self, chroma_host="chromadb", chroma_port=8000):
        # Initialize embedding model (runs in fastapi_backend container)
        self.embedder = SentenceTransformer('sentence-transformers/all-MiniLM-L6-v2')
        
        # Connect to ChromaDB
        self.chroma_client = chromadb.HttpClient(host=chroma_host, port=chroma_port)
        self.collection = self.chroma_client.get_or_create_collection(
            name="cv_embeddings",
            metadata={"hnsw:space": "cosine"}
        )
    
    def extract_text_from_pdf(self, pdf_path: str) -> str:
        """Extract full text from PDF"""
        doc = fitz.open(pdf_path)
        text = ""
        for page in doc:
            text += page.get_text()
        doc.close()
        return text.strip()
    
    def chunk_text(self, text: str, chunk_size: int = 500) -> List[str]:
        """Simple chunking by character count with overlap"""
        chunks = []
        words = text.split()
        current_chunk = []
        current_length = 0
        
        for word in words:
            current_chunk.append(word)
            current_length += len(word) + 1
            
            if current_length >= chunk_size:
                chunks.append(" ".join(current_chunk))
                # Keep last 50 words for overlap
                current_chunk = current_chunk[-50:]
                current_length = sum(len(w) + 1 for w in current_chunk)
        
        if current_chunk:
            chunks.append(" ".join(current_chunk))
        
        return chunks
    
    def ingest_cv(self, cv_path: str, cv_id: str, filename: str):
        """Full ingestion pipeline: PDF → text → chunks → embeddings → ChromaDB"""
        # Extract text
        text = self.extract_text_from_pdf(cv_path)
        
        # Chunk text
        chunks = self.chunk_text(text)
        
        # Generate embeddings
        embeddings = self.embedder.encode(chunks).tolist()
        
        # Store in ChromaDB with metadata
        ids = [f"{cv_id}_chunk_{i}" for i in range(len(chunks))]
        metadatas = [
            {
                "cv_id": cv_id,
                "filename": filename,
                "chunk_index": i,
                "text_preview": chunk[:200]
            }
            for i, chunk in enumerate(chunks)
        ]
        
        self.collection.add(
            ids=ids,
            embeddings=embeddings,
            metadatas=metadatas,
            documents=chunks
        )
        
        return {
            "cv_id": cv_id,
            "filename": filename,
            "chunks_created": len(chunks),
            "status": "ingested"
        }
```

**Acceptance Criteria**:

- [ ] PyMuPDF extracts text from PDFs
- [ ] Text chunked into ~500 char segments with overlap
- [ ] Embeddings generated using sentence-transformers (local, no API calls)
- [ ] Chunks stored in ChromaDB with cv_id and filename metadata
- [ ] Can process multiple CVs sequentially
- [ ] Embeddings persist in ChromaDB volume

---

## Task 3: RAG Query & LLM Integration (60 min)

**Goal**: Implement RAG retrieval and Llama 3.1 integration for competence matching.

**Why**: Core feature - retrieve relevant CV chunks and use LLM to analyze matches with scores.

**Implementation**:

Create `src/app/services/rag.py`:

```python
import ollama
from sentence_transformers import SentenceTransformer
import chromadb
from typing import List, Dict

class RAGService:
    def __init__(self, chroma_host="chromadb", chroma_port=8000, ollama_host="http://ollama:11434"):
        self.embedder = SentenceTransformer('sentence-transformers/all-MiniLM-L6-v2')
        self.chroma_client = chromadb.HttpClient(host=chroma_host, port=chroma_port)
        self.collection = self.chroma_client.get_collection(name="cv_embeddings")
        self.ollama_host = ollama_host
    
    def retrieve_relevant_chunks(self, query: str, top_k: int = 5) -> Dict:
        """Semantic search in ChromaDB"""
        query_embedding = self.embedder.encode([query])[0].tolist()
        
        results = self.collection.query(
            query_embeddings=[query_embedding],
            n_results=top_k,
            include=["metadatas", "documents", "distances"]
        )
        
        return results
    
    def match_competence(self, competence: str, top_k: int = 3) -> Dict:
        """RAG pipeline: Retrieve + LLM analysis"""
        # Step 1: Retrieve relevant CV chunks
        results = self.retrieve_relevant_chunks(competence, top_k)
        
        if not results['documents'][0]:
            return {
                "competence": competence,
                "matches": [],
                "message": "No relevant CVs found"
            }
        
        # Step 2: Build context from retrieved chunks
        context_parts = []
        for i, (doc, metadata) in enumerate(zip(results['documents'][0], results['metadatas'][0])):
            context_parts.append(
                f"CV: {metadata['filename']}\n"
                f"Content: {doc}\n"
                f"---"
            )
        context = "\n".join(context_parts)
        
        # Step 3: Create LLM prompt
        prompt = f"""You are a CV analyst. Analyze these CV excerpts to find candidates with the competence: "{competence}"

CV Excerpts:
{context}

Task: For each CV, determine if the candidate has this competence and provide:
1. Candidate name (from CV filename)
2. Confidence score (0-100)
3. Brief evidence (1 sentence)

Format your response as:
- Candidate Name | Score: XX | Evidence: <brief quote>

Only include candidates with score > 40."""

        # Step 4: Call Ollama (Llama 3.1 8B)
        try:
            response = ollama.chat(
                model='llama3.1:8b',
                messages=[{'role': 'user', 'content': prompt}],
                options={'temperature': 0.3}  # Lower temperature for factual analysis
            )
            
            return {
                "competence": competence,
                "llm_analysis": response['message']['content'],
                "chunks_retrieved": len(results['documents'][0]),
                "status": "success"
            }
        except Exception as e:
            return {
                "competence": competence,
                "error": str(e),
                "status": "failed"
            }
```

**Acceptance Criteria**:

- [ ] RAG service queries ChromaDB with embedded competence
- [ ] Returns top-k most relevant CV chunks
- [ ] Constructs context with candidate metadata
- [ ] Calls Ollama Llama 3.1 8B model
- [ ] LLM returns structured analysis with scores
- [ ] Handles cases with no matches gracefully
- [ ] Preserves candidate identity in results

---

## Task 4: FastAPI Endpoints (45 min)

**Goal**: Create API endpoints for CV ingestion and competence matching.

**Why**: Need HTTP API to interact with the RAG system from frontend.

**Implementation**:

Create `src/app/api/cv_endpoints.py`:

```python
from fastapi import APIRouter, UploadFile, File, HTTPException
from pydantic import BaseModel
from typing import List
import uuid
import os

from ..services.ingestion import IngestionService
from ..services.rag import RAGService

router = APIRouter(prefix="/api", tags=["cv"])

# Initialize services
ingestion_service = IngestionService()
rag_service = RAGService()

class CompetenceMatchRequest(BaseModel):
    competences: List[str]
    top_k: int = 3

@router.post("/cv/upload")
async def upload_cv(file: UploadFile = File(...)):
    """Upload CV PDF and ingest into vector database"""
    if not file.filename.endswith('.pdf'):
        raise HTTPException(400, "Only PDF files are accepted")
    
    # Generate unique ID
    cv_id = str(uuid.uuid4())
    temp_path = f"/tmp/{cv_id}.pdf"
    
    # Save uploaded file
    content = await file.read()
    with open(temp_path, "wb") as f:
        f.write(content)
    
    try:
        # Ingest CV
        result = ingestion_service.ingest_cv(temp_path, cv_id, file.filename)
        os.remove(temp_path)  # Cleanup
        return result
    except Exception as e:
        if os.path.exists(temp_path):
            os.remove(temp_path)
        raise HTTPException(500, f"Ingestion failed: {str(e)}")

@router.post("/competences/match")
async def match_competences(request: CompetenceMatchRequest):
    """Match multiple competences against all CVs using RAG"""
    results = []
    
    for competence in request.competences:
        match_result = rag_service.match_competence(competence, request.top_k)
        results.append(match_result)
    
    return {
        "matches": results,
        "total_competences": len(request.competences)
    }

@router.get("/health")
async def health_check():
    """Health check endpoint"""
    return {"status": "healthy", "service": "competence-mapping-ai"}
```

Register in `src/app/main.py`:

```python
from fastapi import FastAPI
from .api.cv_endpoints import router as cv_router

app = FastAPI(title="Competence Mapping AI")

app.include_router(cv_router)

@app.get("/")
async def root():
    return {"message": "Competence Mapping AI - RAG System"}
```

**Acceptance Criteria**:

- [ ] `POST /api/cv/upload` accepts PDF files
- [ ] Upload triggers full ingestion pipeline
- [ ] Returns cv_id and chunk count
- [ ] `POST /api/competences/match` accepts list of competences
- [ ] Returns LLM analysis for each competence
- [ ] `GET /health` returns service status
- [ ] Proper error handling and HTTP status codes
- [ ] File cleanup after processing

---

## Task 5: Simple Frontend (40 min - OPTIONAL)

**Goal**: Basic HTML interface for testing the RAG system.

**Note**: If time is limited, skip this and use cURL/Postman for testing.

**Implementation**:

Create `src/app/static/index.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Competence Mapping AI</title>
    <style>
        body { font-family: Arial; max-width: 800px; margin: 40px auto; padding: 20px; }
        .section { border: 1px solid #ddd; padding: 20px; margin: 20px 0; border-radius: 8px; }
        button { background: #007bff; color: white; padding: 10px 20px; border: none; cursor: pointer; }
        button:hover { background: #0056b3; }
        textarea { width: 100%; padding: 10px; margin: 10px 0; }
        pre { background: #f5f5f5; padding: 15px; overflow: auto; }
    </style>
</head>
<body>
    <h1>🧠 Competence Mapping AI</h1>
    <p><strong>Zero Data Leakage:</strong> All processing is local (Ollama + ChromaDB)</p>
    
    <div class="section">
        <h2>1. Upload CV</h2>
        <input type="file" id="fileInput" accept=".pdf">
        <button onclick="uploadCV()">Upload</button>
        <pre id="uploadResult"></pre>
    </div>
    
    <div class="section">
        <h2>2. Match Competences</h2>
        <textarea id="competences" rows="5" placeholder="Python&#10;Docker&#10;Project Management"></textarea>
        <button onclick="matchCompetences()">Match</button>
        <pre id="matchResult"></pre>
    </div>
    
    <script>
        async function uploadCV() {
            const file = document.getElementById('fileInput').files[0];
            if (!file) { alert('Select a PDF'); return; }
            
            const formData = new FormData();
            formData.append('file', file);
            
            document.getElementById('uploadResult').textContent = 'Processing...';
            const res = await fetch('/api/cv/upload', { method: 'POST', body: formData });
            const data = await res.json();
            document.getElementById('uploadResult').textContent = JSON.stringify(data, null, 2);
        }
        
        async function matchCompetences() {
            const competences = document.getElementById('competences').value.split('\n').filter(c => c.trim());
            if (competences.length === 0) { alert('Enter competences'); return; }
            
            document.getElementById('matchResult').textContent = 'Querying...';
            const res = await fetch('/api/competences/match', {
                method: 'POST',
                headers: {'Content-Type': 'application/json'},
                body: JSON.stringify({competences, top_k: 3})
            });
            const data = await res.json();
            document.getElementById('matchResult').textContent = JSON.stringify(data, null, 2);
        }
    </script>
</body>
</html>
```

Update `src/app/main.py` to serve static files:

```python
from fastapi.staticfiles import StaticFiles

app.mount("/static", StaticFiles(directory="src/app/static"), name="static")
```

**Acceptance Criteria**:

- [ ] Simple HTML UI at `/static/index.html`
- [ ] CV upload form working
- [ ] Competence matching form working
- [ ] Results displayed in readable format
- [ ] OR skip this task and use API testing tools

---

## Overall Success Criteria

**Must Have**:

- [ ] Docker Compose with all 3 services running
- [ ] CV upload ingestion pipeline working (PDF → ChromaDB)
- [ ] RAG competence matching with Llama 3.1 working
- [ ] Results preserve candidate identity (filename/cv_id)
- [ ] Data persists in Docker volumes
- [ ] Zero data leakage (fully local processing)

**Stretch Goals** (if time permits):

- [ ] Frontend UI (React/shadcn or HTML/JS)
- [ ] Batch CV upload endpoint
- [ ] Better prompt engineering for LLM
- [ ] Caching for repeated queries

---

## Quick Start Commands

```bash
# 1. Pull Ollama model (do this first, takes time!)
ollama pull llama3.1:8b

# 2. Start all services
cd competence-mapping-ai
docker compose up -d

# 3. Check service health
curl http://localhost:11434/api/tags        # Ollama models
curl http://localhost:8001/api/v1/heartbeat # ChromaDB
curl http://localhost:8000/health           # Backend API

# 4. Upload a CV
curl -X POST -F "file=@research/test_data/cv-example.pdf" \
    http://localhost:8000/api/cv/upload

# 5. Match competences
curl -X POST -H "Content-Type: application/json" \
    -d '{"competences":["Python","Docker","Leadership"],"top_k":3}' \
    http://localhost:8000/api/competences/match
```

---

## Testing Checklist

1. [ ] All services start with `docker compose up -d`
2. [ ] Ollama accessible at `localhost:11434`
3. [ ] ChromaDB accessible at `localhost:8001`
4. [ ] Backend API accessible at `localhost:8000`
5. [ ] Upload 3-5 test CVs successfully
6. [ ] Query a competence and get LLM response with scores
7. [ ] Restart containers and verify data persists
8. [ ] Check that candidate names/filenames are preserved in results

---

## Time Allocation

- **Task 1 - Docker Services**: 30 minutes
- **Task 2 - PDF & Embedding**: 45 minutes  
- **Task 3 - RAG & LLM**: 60 minutes
- **Task 4 - API Endpoints**: 45 minutes
- **Task 5 - Frontend** (optional): 40 minutes
- **Total**: 3-4 hours

---

## Architecture Summary

```
┌─────────────┐      ┌──────────────────┐      ┌─────────────┐
│  Frontend   │─────▶│ fastapi_backend  │─────▶│   ollama    │
│ (Optional)  │      │  - Orchestrator  │      │ (Llama 3.1) │
└─────────────┘      │  - RAG Logic     │      └─────────────┘
                     │  - Embeddings    │
                     └────────┬─────────┘
                              │
                              ▼
                     ┌─────────────────┐
                     │    chromadb     │
                     │ (Vector Store)  │
                     └─────────────────┘

Volumes: ollama_models, chroma_data, /curricula
```

