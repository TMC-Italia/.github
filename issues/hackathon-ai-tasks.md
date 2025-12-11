# AI Project - Hackathon Tasks

**Duration**: 4 Hours  
**Goal**: Upload CV PDF → get competence matches with scores

## Pre-Event Checklist

- [ ] 5-10 dummy CVs in PDF format in `research/test_data/`
- [ ] Dependencies added: `chromadb`, `langchain`, `langchain-community`, `pymupdf`, `python-multipart`, `ollama`
- [ ] Ollama installed locally with Llama 3.1 8B model pulled (`ollama pull llama3.1:8b`)
- [ ] BAAI/bge-m3 embedding model pre-downloaded via HuggingFace
- [ ] Competence list JSON created

---

## Task 1: PDF Parser Service with Chunking (45 min)

**Goal**: Extract text from CV PDFs and split into chunks with metadata.

**Why**: Semi-structured CVs require chunking to find specific skills. Metadata ensures we don't lose candidate identity.

**Implementation**:

Create `src/app/services/pdf_parser.py`:

```python
import fitz  # PyMuPDF
from typing import Dict, List
from langchain.text_splitter import RecursiveCharacterTextSplitter

class PDFParser:
    def __init__(self):
        self.text_splitter = RecursiveCharacterTextSplitter(
            chunk_size=1000,
            chunk_overlap=200,
            length_function=len,
        )
    
    def extract_text(self, pdf_path: str) -> Dict[str, any]:
        doc = fitz.open(pdf_path)
        full_text = ""
        
        for page in doc:
            full_text += page.get_text()
        
        doc.close()
        
        # Basic cleaning
        full_text = full_text.strip()
        full_text = " ".join(full_text.split())
        
        # Split into chunks
        chunks = self.text_splitter.split_text(full_text)
        
        return {
            "raw_text": full_text,
            "chunks": chunks,
            "page_count": len(doc),
            "word_count": len(full_text.split()),
            "chunk_count": len(chunks)
        }
```

**Acceptance Criteria**:

- [ ] `pdf_parser.py` exists in `src/app/services/`
- [ ] Uses PyMuPDF (fitz) for extraction
- [ ] Implements LangChain text splitter (1000 chars, 200 overlap)
- [ ] Returns chunks with metadata
- [ ] Handles multi-page PDFs
- [ ] Basic error handling for corrupted PDFs

---

## Task 2: Vector Database Setup with Metadata (45 min)

**Goal**: Add ChromaDB to docker-compose with chunk-level metadata storage.

**Why**: Need vector storage for semantic search. Metadata prevents losing candidate identity.

**Implementation**:

Add to `docker-compose.yml`:

```yaml
services:
  app:
    # existing config...
    depends_on:
      - chromadb
      - ollama
    environment:
      - CHROMA_HOST=chromadb
      - CHROMA_PORT=8000
      - OLLAMA_HOST=http://ollama:11434

  chromadb:
    image: chromadb/chroma:latest
    ports:
      - "8001:8000"
    volumes:
      - chroma_data:/chroma/chroma
    environment:
      - IS_PERSISTENT=TRUE

  ollama:
    image: ollama/ollama:latest
    ports:
      - "11434:11434"
    volumes:
      - ollama_models:/root/.ollama
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]

volumes:
  chroma_data:
  ollama_models:
```

Create `src/app/services/vector_store.py`:

```python
import chromadb
from typing import List, Dict

class VectorStore:
    def __init__(self):
        self.client = chromadb.HttpClient(host="chromadb", port=8000)
        self.collection = self.client.get_or_create_collection(
            name="cv_chunks",
            metadata={"hnsw:space": "cosine"}
        )
    
    def add_chunks(self, cv_id: str, chunks: List[str], embeddings: List[List[float]], filename: str):
        """Add CV chunks with metadata to preserve candidate identity"""
        ids = [f"{cv_id}_chunk_{i}" for i in range(len(chunks))]
        metadatas = [
            {
                "cv_id": cv_id,
                "filename": filename,
                "chunk_index": i,
                "text": chunk[:500]  # Store preview
            }
            for i, chunk in enumerate(chunks)
        ]
        
        self.collection.add(
            ids=ids,
            embeddings=embeddings,
            metadatas=metadatas,
            documents=chunks
        )
    
    def search(self, query_embedding: List[float], n_results: int = 5):
        return self.collection.query(
            query_embeddings=[query_embedding],
            n_results=n_results,
            include=["metadatas", "documents", "distances"]
        )
```

**Acceptance Criteria**:

- [ ] ChromaDB service in docker-compose.yml
- [ ] Ollama service in docker-compose.yml
- [ ] Persistent volumes configured (chroma_data, ollama_models)
- [ ] `vector_store.py` stores chunks with candidate metadata
- [ ] Can add multiple chunks per CV
- [ ] Search returns metadata including cv_id and filename
- [ ] Data persists after container restart

---

## Task 3: Embedding Service with BAAI/bge-m3 (45 min)

**Goal**: Generate embeddings using BAAI/bge-m3 multilingual model.

**Why**: Need vector representations for semantic search. BGE-M3 supports Italian and provides high-quality embeddings.

**Implementation**:

Create `src/app/services/embeddings.py`:

```python
from langchain_community.embeddings import HuggingFaceEmbeddings
from typing import List

class EmbeddingService:
    def __init__(self):
        # BAAI/bge-m3 multilingual model
        model_name = "BAAI/bge-m3"
        model_kwargs = {'device': 'cpu'}  # Use 'cuda' if GPU available
        encode_kwargs = {'normalize_embeddings': True}
        
        self.model = HuggingFaceEmbeddings(
            model_name=model_name,
            model_kwargs=model_kwargs,
            encode_kwargs=encode_kwargs
        )
    
    def embed_text(self, text: str) -> List[float]:
        """Embed single text"""
        return self.model.embed_query(text)
    
    def embed_documents(self, texts: List[str]) -> List[List[float]]:
        """Embed multiple texts (chunks) in batch"""
        return self.model.embed_documents(texts)
```

Update `Dockerfile` to cache model:

```dockerfile
# After installing dependencies
RUN python -c "from langchain_community.embeddings import HuggingFaceEmbeddings; \
    HuggingFaceEmbeddings(model_name='BAAI/bge-m3')"
```

**Acceptance Criteria**:

- [ ] `embeddings.py` service created
- [ ] Uses BAAI/bge-m3 via LangChain
- [ ] Generates 1024-dim vectors (BGE-M3 default)
- [ ] Supports batch encoding for chunks
- [ ] Normalized embeddings enabled
- [ ] Model cached in Docker image
- [ ] Fast inference (<200ms per CV with chunks)

---

## Task 4: API Endpoints with RAG (60 min)

**Goal**: Create endpoints for CV upload, search, and competence matching using RAG.

**Why**: Need API to interact with the system and leverage LLM for intelligent matching.

**Implementation**:

Create `src/app/api/cv.py`:

```python
from fastapi import APIRouter, UploadFile, File, HTTPException
from typing import List
from pydantic import BaseModel
import uuid
import ollama

from ..services.pdf_parser import PDFParser
from ..services.embeddings import EmbeddingService
from ..services.vector_store import VectorStore

router = APIRouter(prefix="/api/cv", tags=["cv"])
parser = PDFParser()
embedder = EmbeddingService()
vector_store = VectorStore()

class CompetenceQuery(BaseModel):
    competences: List[str]
    top_k: int = 5

@router.post("/upload")
async def upload_cv(file: UploadFile = File(...)):
    """Upload CV, extract text, chunk, embed, and store"""
    if not file.filename.endswith('.pdf'):
        raise HTTPException(400, "Only PDF files accepted")
    
    # Save temp file
    content = await file.read()
    cv_id = str(uuid.uuid4())
    temp_path = f"/tmp/{cv_id}.pdf"
    
    with open(temp_path, "wb") as f:
        f.write(content)
    
    # Extract text and chunks
    data = parser.extract_text(temp_path)
    chunks = data["chunks"]
    
    # Generate embeddings for all chunks
    embeddings = embedder.embed_documents(chunks)
    
    # Store chunks with metadata in vector DB
    vector_store.add_chunks(
        cv_id=cv_id,
        chunks=chunks,
        embeddings=embeddings,
        filename=file.filename
    )
    
    return {
        "cv_id": cv_id,
        "filename": file.filename,
        "status": "processed",
        "word_count": data["word_count"],
        "chunk_count": data["chunk_count"]
    }

@router.post("/search")
async def search_cvs(query: str, limit: int = 5):
    """Semantic search across CV chunks"""
    query_embedding = embedder.embed_text(query)
    results = vector_store.search(query_embedding, n_results=limit)
    
    return {
        "query": query,
        "results": results
    }

@router.post("/competences/match")
async def match_competences(query: CompetenceQuery):
    """RAG-based competence matching using Ollama + Llama 3.1"""
    all_matches = []
    
    for competence in query.competences:
        # Retrieve relevant CV chunks
        comp_embedding = embedder.embed_text(competence)
        results = vector_store.search(comp_embedding, n_results=query.top_k)
        
        # Build context from chunks
        context = "\n\n".join([
            f"Candidate: {r['metadatas'][0]['filename']}\n{r['documents'][0]}"
            for r in results['results']
        ])
        
        # LLM prompt for competence matching
        prompt = f"""You are analyzing CVs for competence matching.

Competence to find: {competence}

Relevant CV excerpts:
{context}

Task: List candidates who have this competence with confidence scores (0-100).
Format: "Candidate Name - Score: XX - Evidence: brief quote"
"""
        
        # Call Ollama (Llama 3.1)
        response = ollama.chat(
            model='llama3.1:8b',
            messages=[{'role': 'user', 'content': prompt}]
        )
        
        all_matches.append({
            "competence": competence,
            "llm_response": response['message']['content'],
            "retrieved_chunks": len(results['results'])
        })
    
    return {"matches": all_matches}
```

Register router in `src/app/main.py`:

```python
from .api import cv

app.include_router(cv.router)
```

**Acceptance Criteria**:

- [ ] `POST /api/cv/upload` accepts PDF files
- [ ] Upload chunks CV and stores with metadata
- [ ] `POST /api/cv/search` performs semantic search
- [ ] Returns top matches with candidate names preserved
- [ ] `POST /api/competences/match` uses RAG + Llama 3.1
- [ ] LLM response includes confidence scores
- [ ] All endpoints have proper error handling

---

## Task 5: Test Data Preparation (20 min)

**Goal**: Create 5 dummy CVs for testing.

**Why**: Need test data without real personal info.

**Implementation**:
- Create `research/test_data/` directory
- Generate dummy CVs with realistic content:
  - Names: Mario Rossi, Laura Bianchi, etc.
  - Skills: Python, Docker, Project Management, etc.
  - Experience: 2-10 years in various roles
- Use different formats/layouts
- Include Italian and English content

**Acceptance Criteria**:
- [ ] 5-10 PDF files in `research/test_data/`
- [ ] No real personal data
- [ ] Realistic skills and experience
- [ ] Mix of technical and non-technical roles
- [ ] Different CV formats/layouts
- [ ] Both Italian and English CVs

---

## Task 6: Simple Upload UI with Competence Matching (30 min)

**Goal**: Basic HTML form for CV upload and competence matching demo.

**Why**: Need UI for testing RAG workflow end-to-end.

**Implementation**:

Create `src/app/templates/index.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>CV Competence Mapping - RAG Demo</title>
    <style>
        body { font-family: Arial; max-width: 900px; margin: 50px auto; padding: 20px; }
        h1 { color: #333; }
        .section { border: 2px dashed #ccc; padding: 30px; margin: 20px 0; }
        button { 
            background: #007bff; 
            color: white; 
            padding: 10px 20px; 
            border: none; 
            cursor: pointer;
            margin: 10px 5px;
        }
        button:hover { background: #0056b3; }
        .results { margin-top: 20px; padding: 20px; background: #f5f5f5; }
        .loading { color: #666; font-style: italic; }
        textarea { width: 100%; padding: 10px; margin: 10px 0; }
    </style>
</head>
<body>
    <h1>🧠 CV Competence Mapping - RAG Demo</h1>
    <p><strong>Zero Data Leakage:</strong> All processing happens locally (Ollama + ChromaDB)</p>
    
    <!-- Upload Section -->
    <div class="section">
        <h2>1. Upload CV (PDF)</h2>
        <form id="uploadForm" enctype="multipart/form-data">
            <input type="file" id="fileInput" accept=".pdf" required>
            <button type="submit">Upload & Process</button>
        </form>
        <div id="uploadResults" class="results" style="display:none;">
            <h3>Upload Results:</h3>
            <pre id="uploadText"></pre>
        </div>
    </div>
    
    <!-- Competence Matching Section -->
    <div class="section">
        <h2>2. Match Competences (RAG + Llama 3.1)</h2>
        <textarea id="competences" rows="4" placeholder="Enter competences (one per line):
Python
Project Management
Docker"></textarea>
        <button onclick="matchCompetences()">🔍 Find Matches</button>
        <div id="matchResults" class="results" style="display:none;">
            <h3>RAG Results:</h3>
            <pre id="matchText"></pre>
        </div>
    </div>
    
    <script>
        // Upload CV
        document.getElementById('uploadForm').onsubmit = async (e) => {
            e.preventDefault();
            const formData = new FormData();
            formData.append('file', document.getElementById('fileInput').files[0]);
            
            document.getElementById('uploadResults').style.display = 'block';
            document.getElementById('uploadText').textContent = 'Processing...';
            
            try {
                const response = await fetch('/api/cv/upload', {
                    method: 'POST',
                    body: formData
                });
                const result = await response.json();
                document.getElementById('uploadText').textContent = JSON.stringify(result, null, 2);
            } catch (err) {
                document.getElementById('uploadText').textContent = 'Error: ' + err.message;
            }
        };
        
        // Match Competences
        async function matchCompetences() {
            const competences = document.getElementById('competences').value
                .split('\n')
                .filter(c => c.trim().length > 0);
            
            if (competences.length === 0) {
                alert('Please enter at least one competence');
                return;
            }
            
            document.getElementById('matchResults').style.display = 'block';
            document.getElementById('matchText').textContent = 'Querying ChromaDB and Llama 3.1...';
            
            try {
                const response = await fetch('/api/cv/competences/match', {
                    method: 'POST',
                    headers: {'Content-Type': 'application/json'},
                    body: JSON.stringify({competences: competences, top_k: 3})
                });
                const result = await response.json();
                document.getElementById('matchText').textContent = JSON.stringify(result, null, 2);
            } catch (err) {
                document.getElementById('matchText').textContent = 'Error: ' + err.message;
            }
        }
    </script>
</body>
</html>
```

Update `src/app/main.py`:

```python
from fastapi import Request
from fastapi.templating import Jinja2Templates
from fastapi.responses import HTMLResponse

templates = Jinja2Templates(directory="src/app/templates")

@app.get("/", response_class=HTMLResponse)
async def root(request: Request):
    return templates.TemplateResponse("index.html", {"request": request})
```

**Acceptance Criteria**:

- [ ] HTML form at `/` root path
- [ ] File upload section with PDF validation
- [ ] Competence matching section with textarea
- [ ] Both sections show loading states
- [ ] Results displayed as formatted JSON
- [ ] Works in modern browsers
- [ ] Shows "Zero Data Leakage" notice

---

## Overall Success Criteria

**Must Have**:

- [ ] Upload PDF → returns extracted text with chunks
- [ ] ChromaDB + Ollama services running
- [ ] BAAI/bge-m3 embeddings working
- [ ] Search returns results with candidate metadata preserved
- [ ] RAG competence matching with Llama 3.1 works
- [ ] `docker compose up` works
- [ ] Basic UI form functional with both upload and matching

**Stretch Goals** (if time permits):

- [ ] Batch CV upload
- [ ] Better chunk metadata (skills extraction)
- [ ] LLM response parsing and scoring
- [ ] Error handling and retry logic

---

## Quick Start

```bash
# 1. Install Ollama and pull Llama 3.1
ollama pull llama3.1:8b

# 2. Install dependencies
cd competence-mapping-ai
uv sync

# 3. Start services (ChromaDB + Ollama + App)
docker compose up -d

# 4. Open UI
open http://localhost:8000

# 5. Test upload via API
curl -X POST -F "file=@research/test_data/cv-mario-rossi.pdf" \
    http://localhost:8000/api/cv/upload

# 6. Test competence matching via API
curl -X POST -H "Content-Type: application/json" \
    -d '{"competences":["Python","Docker"],"top_k":3}' \
    http://localhost:8000/api/cv/competences/match
```

## Testing Checklist

1. **Prepare test data**: Create 5-10 dummy CVs with varied skills
2. **Start services**: `docker compose up -d`
3. **Verify Ollama**: `curl http://localhost:11434/api/tags`
4. **Verify ChromaDB**: `curl http://localhost:8001/api/v1/heartbeat`
5. **Upload CVs**: Via UI at http://localhost:8000
6. **Check embeddings**: Verify chunks stored in ChromaDB
7. **Test RAG**: Query competences and check LLM responses
8. **Validate metadata**: Ensure candidate names preserved in results
