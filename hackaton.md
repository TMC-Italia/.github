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

### 🧠 AI (3 people - Reviewer: Alessandro)

- Matilde
- Francesco
- Beatrice
- Walter
- Thomas
- Roberto
- Mattia

### ☁️ Cloud (2 people - Reviewer: Flavio)

- Marco - Bootstrap script + Security hardening
- Flavio - Network + Storage + Validation
- Zaccaria - Docker + Portainer + Monitoring
- Mattia -

### 📍 Places (2 people - Reviewer: Alberto)

- Alberto - Backend tests + Code quality + CI/CD
- XXX - Frontend + UX improvements

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

### 🧠 AI: "CV Upload → Competence Matching"

**Goal**: Upload PDF → extract text → show similar competences

**Tasks** (3.5h total):

1. **Core Services** (120min)
   - PDF parser (`pdf_parser.py`)
   - ChromaDB in docker-compose
   - Vector store wrapper (`vector_store.py`)

2. **API Layer** (90min)
   - Embedding service with sentence-transformers
   - `/api/cv/upload` endpoint
   - `/api/cv/search` endpoint

3. **UI + Testing** (60min)
   - Create 5 dummy CVs (no real data)
   - Simple HTML upload form
   - Test all endpoints manually

**Success Criteria**:

- Upload PDF → returns extracted text + word count
- Search "Python" → returns relevant CVs
- `docker compose up` works
- UI form uploads and shows results
- At least 3 test CVs processed successfully

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
## 