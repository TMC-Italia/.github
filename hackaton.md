# TMC Hackathon Plan

**Duration**: 4 Hours (3.5h coding + 0.5h demos)
**Participants**: 7 People  
**Goal**: One shippable feature per project

**Reality Check**:

- Actual coding time: ~3.5 hours
- Time lost to: setup issues, debugging, integration
- Keep scope minimal, cut ruthlessly

## What NOT to Do

- ❌ Don't aim for 80% test coverage (aim for 5 key tests)
- ❌ Don't try to implement all features (pick ONE working demo)
- ❌ Don't spend >30min on any single bug (timebox and move on)
- ❌ Don't write comprehensive docs (README basics only)
- ❌ Don't perfect the code (working > perfect)

## Pre-Event Checklist

### General

- [ ] Pizza/snacks ordered
- [ ] WiFi access for all laptops
- [ ] GitHub access to TMC-Italia org confirmed

### ☁️ Cloud

- [ ] Fresh Ubuntu 22.04 LTS installed on test PC (with VM snapshot)
- [ ] Tailscale pre-auth keys generated
- [ ] Static IPs documented (Master: 192.168.1.100, Worker: 192.168.1.101, etc.)
- [ ] tmc-cloud repo cloned with executable scripts

### 🧠 AI

- [ ] 5-10 dummy CVs created in `research/test_data/` (no real data!)

### 📍 Places

## Team Assignments (11 people)

### 🧠 AI (Reviewer: Alessandro)

- Matilde
- Francesco
- Beatrice
- Walter
- Thomas
- Roberto
- Mattia

### ☁️ Cloud (Reviewer: Flavio)

- Marco
- Flavio
- Zaccaria
- Mattia

### 📍 Places (Reviewer: Alberto)

- Alberto

---

## Project Goals & Tasks

### ☁️ Cloud: "Automated Node Setup"

**Goal**: Single script to bootstrap Ubuntu → production node

**Tasks** (3.5h total):

1. **Bootstrap Script** (90min)
   - Create `scripts/bootstrap-node.sh` with role selection
   - Call existing scripts in sequence
   - Add basic error handling

2. **Security Hardening** (60min)
   - UFW firewall with role-based rules
   - SSH key-only auth (no passwords)
   - Basic fail2ban setup

3. **Network + Storage** (60min)
   - Static IP configuration
   - NFS server on storage node
   - NFS client mounts on others

4. **Validation** (30min)
   - Create `validate-node.sh` script
   - Check: Docker, network, SSH, storage

**Success Criteria**:

- Bootstrap script runs without manual intervention
- PC accessible via SSH with keys
- Docker installed and running
- Static IP configured
- Storage mounted (if worker/master)
- Validation script passes all checks

### 🧠 AI: "RAG-based CV Competence Matching"

**Goal**: Upload PDF → chunk with metadata → semantic search → LLM matching

**Tasks** (3.5h total):

1. **Core Services** (135min)
   - PDF parser with LangChain chunking (`pdf_parser.py`)
   - ChromaDB + Ollama in docker-compose
   - Vector store with chunk metadata (`vector_store.py`)
   - BAAI/bge-m3 embedding service

2. **RAG API Layer** (90min)
   - `/api/cv/upload` endpoint (chunks + embeddings)
   - `/api/cv/search` semantic search endpoint
   - `/api/cv/competences/match` RAG with Llama 3.1

3. **UI + Testing** (45min)
   - Create 5 dummy CVs (no real data)
   - HTML form with upload + competence matching
   - Test RAG workflow end-to-end

**Success Criteria**:

- Upload PDF → chunks stored with candidate metadata
- Search "Python" → returns relevant chunks with CV names
- Competence matching uses Llama 3.1 for intelligent scoring
- `docker compose up` starts ChromaDB + Ollama + App
- UI shows RAG results with LLM responses
- Zero data leakage (all local processing)

### 📍 Places: "Testing + Polish"

**Goal**: Add tests, linting, better UX

**Tasks** (3.5h total):

1. **Backend Quality** (120min)
   - Setup pytest with fixtures
   - Write tests for main API endpoints
   - Add ruff + mypy to pyproject.toml
   - Create basic CI workflow

2. **Frontend Polish** (120min)
   - Setup Vitest + React Testing Library
   - Test 2-3 key components
   - Add loading spinners
   - Error boundaries
   - Toast notifications

**Success Criteria**:

- Backend: 5+ tests passing
- Frontend: 3+ component tests passing
- Linting configured (ruff, eslint)
- UI shows loading states
- UI handles errors gracefully
- CI workflow file exists (doesn't need to run)
