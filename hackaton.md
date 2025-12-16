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

- [ ] 3x Fresh Ubuntu 22.04 LTS PCs (each with 1 SSD)
- [ ] Tailscale pre-auth keys generated
- [ ] Static IPs planned (192.168.1.100-102)
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

### ☁️ Cloud: "3-Node Cluster with Distributed Storage"

**Goal**: Automated setup for 3-node cluster with networking, security, distributed storage, and visible management

**Tasks** (4.5h total):

1. **Node Setup Script** (90min)
   - Create `scripts/setup-node.sh` for automated PC setup
   - Configure static IP and hostname
   - Install Docker, basic packages
   - Auto-connect to Tailscale with pre-auth key
   - System hardening basics

2. **Security & Firewall** (45min)
   - Create `scripts/configure-security.sh`
   - UFW firewall with required ports (SSH, Tailscale, storage)
   - SSH hardening (key-only auth, no root)
   - Install fail2ban

3. **Distributed Storage Research & Setup** (105min)
   - **Phase 1: Research (30min)** - Evaluate options:
     - MicroCeph (simplified Ceph)
     - GlusterFS (simple replication)
     - Longhorn (Kubernetes-native)
     - Ceph (full-featured)
   - **Phase 2: Implementation (75min)** - Deploy chosen solution
   - Configure 3-node cluster using each PC's SSD
   - Setup replication and fault tolerance
   - Test storage accessibility from all nodes

4. **Visible Management & Monitoring** (60min)
   - Deploy Portainer (container management UI)
   - Deploy Grafana + Loki (monitoring & logs)
   - Configure dashboards showing cluster health
   - Create demo script for management

**Success Criteria**:

- Single command to setup fresh PC
- All 3 nodes with static IPs and Tailscale connected
- SSH accessible with keys only (hardened)
- Firewall enabled and configured
- Storage solution chosen and documented
- Distributed storage operational across 3 nodes
- Data accessible from all nodes
- Storage survives single node failure
- **Portainer UI accessible showing all 3 nodes**
- **Grafana showing live logs from entire cluster**
- **Can demonstrate working cluster to management**

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

### 📍 Places: "Backend Testing, Code Quality & Deployment"

**Goal**: Backend testing, code quality improvements, CI/CD setup, and deployment to tmc-cloud

**Tasks** (4h total):

1. **Backend Testing** (90min)
   - Setup pytest with test structure
   - Write tests for main API endpoints
   - Basic test coverage for critical paths
   - Tests pass with coverage report

2. **Code Quality Tools** (45min)
   - Add ruff for linting and formatting
   - Configure pyproject.toml with quality tools
   - Create Makefile with lint/format/test commands
   - Fix all linting errors

3. **CI/CD Pipeline** (45min)
   - Create GitHub Actions workflow
   - Automate testing on PRs
   - Enforce linting checks
   - Test workflow runs successfully

4. **Deployment Preparation** (60min)
   - Create Kubernetes manifests (namespace, deployment, service)
   - Prepare deployment documentation
   - Test Docker build process
   - (Optional) Deploy to tmc-cloud cluster if time permits

**Success Criteria**:

- Backend: 5+ tests passing with coverage report
- Linting configured and passing (ruff)
- Makefile with lint/format/test commands
- CI workflow running on GitHub Actions
- Kubernetes deployment manifests created
- Deployment documentation written
- (Stretch) Application deployed to tmc-cloud
