# AI Project - Hackathon Tasks

**Duration**: 4 Hours  
**Goal**: Upload CV PDF → get competence matches with scores

## Pre-Event Checklist

- [ ] 5-10 dummy CVs in PDF format in `research/test_data/`
- [ ] Dependencies added: `chromadb`, `sentence-transformers`, `pypdf`, `python-multipart`
- [ ] Sentence-transformers model pre-downloaded
- [ ] Competence list JSON created

---

## Task 1: PDF Parser Service (30 min)

**Goal**: Extract text from CV PDFs.

**Why**: Need text content for embeddings.

**Implementation**:

Create `src/app/services/pdf_parser.py`:

```python
from pypdf import PdfReader
from typing import Dict

class PDFParser:
    def extract_text(self, pdf_path: str) -> Dict[str, str]:
        reader = PdfReader(pdf_path)
        text = ""
        for page in reader.pages:
            text += page.extract_text()
        
        # Basic cleaning
        text = text.strip()
        text = " ".join(text.split())  # normalize whitespace
        
        return {
            "raw_text": text,
            "page_count": len(reader.pages),
            "word_count": len(text.split())
        }
```

**Acceptance Criteria**:
- [ ] `pdf_parser.py` exists in `src/app/services/`
- [ ] Extracts text from PDF files
- [ ] Cleans whitespace and formatting
- [ ] Returns dict with text, page count, word count
- [ ] Handles multi-page PDFs
- [ ] Basic error handling for corrupted PDFs

---

## Task 2: Vector Database Setup (45 min)

**Goal**: Add ChromaDB to docker-compose.

**Why**: Need vector storage for semantic search.

**Implementation**:

Add to `docker-compose.yml`:

```yaml
services:
  app:
    # existing config...
    depends_on:
      - chromadb
    environment:
      - CHROMA_HOST=chromadb
      - CHROMA_PORT=8000

  chromadb:
    image: chromadb/chroma:latest
    ports:
      - "8001:8000"
    volumes:
      - chroma_data:/chroma/chroma
    environment:
      - IS_PERSISTENT=TRUE

volumes:
  chroma_data:
```

Create `src/app/services/vector_store.py`:

```python
import chromadb
from typing import List, Dict

class VectorStore:
    def __init__(self):
        self.client = chromadb.HttpClient(host="chromadb", port=8000)
        self.collection = self.client.get_or_create_collection("cvs")
    
    def add_cv(self, cv_id: str, embedding: List[float], metadata: Dict):
        self.collection.add(
            ids=[cv_id],
            embeddings=[embedding],
            metadatas=[metadata]
        )
    
    def search(self, query_embedding: List[float], n_results: int = 5):
        return self.collection.query(
            query_embeddings=[query_embedding],
            n_results=n_results
        )
```

**Acceptance Criteria**:
- [ ] ChromaDB service in docker-compose.yml
- [ ] Persistent volume configured
- [ ] `vector_store.py` client wrapper created
- [ ] Can add embeddings to collection
- [ ] Can query similar vectors
- [ ] Data persists after container restart

---

## Task 3: Embedding Service (45 min)

**Goal**: Generate embeddings using Italian sentence-transformers model.

**Why**: Need vector representations of text for semantic search.

**Implementation**:

Create `src/app/services/embeddings.py`:

```python
from sentence_transformers import SentenceTransformer
from typing import List

class EmbeddingService:
    def __init__(self):
        # Italian multilingual model
        self.model = SentenceTransformer(
            'paraphrase-multilingual-MiniLM-L12-v2'
        )
    
    def encode(self, text: str) -> List[float]:
        return self.model.encode(text).tolist()
    
    def encode_batch(self, texts: List[str]) -> List[List[float]]:
        return self.model.encode(texts).tolist()
```

Update `Dockerfile` to cache model:

```dockerfile
# After installing dependencies
RUN python -c "from sentence_transformers import SentenceTransformer; \
    SentenceTransformer('paraphrase-multilingual-MiniLM-L12-v2')"
```

**Acceptance Criteria**:
- [ ] `embeddings.py` service created
- [ ] Uses Italian multilingual model
- [ ] Generates 384-dim vectors
- [ ] Supports batch encoding
- [ ] Model cached in Docker image
- [ ] Fast inference (<100ms per CV)

---

## Task 4: API Endpoints (50 min)

**Goal**: Create endpoints for CV upload and search.

**Why**: Need API to interact with the system.

**Implementation**:

Create `src/app/api/cv.py`:

```python
from fastapi import APIRouter, UploadFile, File
from ..services.pdf_parser import PDFParser
from ..services.embeddings import EmbeddingService
from ..services.vector_store import VectorStore

router = APIRouter(prefix="/api/cv", tags=["cv"])
parser = PDFParser()
embedder = EmbeddingService()
vector_store = VectorStore()

@router.post("/upload")
async def upload_cv(file: UploadFile = File(...)):
    # Save temp file
    content = await file.read()
    temp_path = f"/tmp/{file.filename}"
    with open(temp_path, "wb") as f:
        f.write(content)
    
    # Extract text
    data = parser.extract_text(temp_path)
    
    # Generate embedding
    embedding = embedder.encode(data["raw_text"])
    
    # Store in vector DB
    cv_id = file.filename.replace(".pdf", "")
    vector_store.add_cv(cv_id, embedding, {
        "filename": file.filename,
        "word_count": data["word_count"]
    })
    
    return {"cv_id": cv_id, "status": "processed", **data}

@router.post("/search")
async def search_cvs(query: str, limit: int = 5):
    query_embedding = embedder.encode(query)
    results = vector_store.search(query_embedding, n_results=limit)
    return {"query": query, "results": results}

@router.post("/competences/match")
async def match_competences(cv_id: str, competences: List[str]):
    # Get CV embedding from vector store
    # Compare with competence embeddings
    # Return top matches with scores
    pass  # Implement similarity scoring
```

**Acceptance Criteria**:
- [ ] `POST /api/cv/upload` accepts PDF files
- [ ] Upload returns extracted text and cv_id
- [ ] `POST /api/cv/search` searches by query string
- [ ] Returns top 5 similar CVs with scores
- [ ] `POST /api/competences/match` returns scored matches
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

## Task 6: Simple Upload UI (25 min)

**Goal**: Basic HTML form for CV upload.

**Why**: Need UI for testing.

**Implementation**:

Create `src/app/templates/index.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>CV Competence Mapping</title>
    <style>
        body { font-family: Arial; max-width: 800px; margin: 50px auto; }
        form { border: 2px dashed #ccc; padding: 30px; text-align: center; }
        .results { margin-top: 20px; padding: 20px; background: #f5f5f5; }
    </style>
</head>
<body>
    <h1>CV Competence Mapping</h1>
    
    <form id="uploadForm" enctype="multipart/form-data">
        <input type="file" id="fileInput" accept=".pdf" required>
        <button type="submit">Upload CV</button>
    </form>
    
    <div id="results" class="results" style="display:none;">
        <h2>Results</h2>
        <pre id="resultText"></pre>
    </div>
    
    <script>
        document.getElementById('uploadForm').onsubmit = async (e) => {
            e.preventDefault();
            const formData = new FormData();
            formData.append('file', document.getElementById('fileInput').files[0]);
            
            const response = await fetch('/api/cv/upload', {
                method: 'POST',
                body: formData
            });
            
            const result = await response.json();
            document.getElementById('results').style.display = 'block';
            document.getElementById('resultText').textContent = JSON.stringify(result, null, 2);
        };
    </script>
</body>
</html>
```

Update `main.py` to serve template:

```python
from fastapi.templating import Jinja2Templates
from fastapi.staticfiles import StaticFiles

templates = Jinja2Templates(directory="src/app/templates")

@app.get("/")
async def root(request: Request):
    return templates.TemplateResponse("index.html", {"request": request})
```

**Acceptance Criteria**:
- [ ] HTML form at `/` root path
- [ ] File upload input (PDF only)
- [ ] Submit button triggers upload
- [ ] Results displayed as JSON
- [ ] Basic styling applied
- [ ] Works in modern browsers

---

## Overall Success Criteria

**Must Have**:
- [ ] Upload PDF → returns extracted text
- [ ] ChromaDB service running
- [ ] Search returns results with similarity scores
- [ ] `docker compose up` works
- [ ] Basic UI form functional

**Stretch Goals** (if time permits):
- [ ] Batch upload
- [ ] Competence matching endpoint
- [ ] Better error handling

---

## Quick Start

```bash
# 1. Install dependencies
cd competence-mapping-ai
uv sync

# 2. Start services
docker compose up -d

# 3. Test upload
curl -X POST -F "file=@research/test_data/cv-mario-rossi.pdf" \
    http://localhost:8000/api/cv/upload

# 4. Test search
curl -X POST -H "Content-Type: application/json" \
    -d '{"query":"Python developer","limit":3}' \
    http://localhost:8000/api/cv/search
```

## Testing

1. Prepare test data (5-10 dummy CVs)
2. Start services: `docker compose up`
3. Upload CVs via UI or API
4. Verify embeddings stored in ChromaDB
5. Test search with various queries
6. Check competence matching accuracy
