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

- [x] Pizza/snacks ordered
- [ ] WiFi access for all laptops
- [x] GitHub access to TMC-Italia org confirmed

### ☁️ Cloud

- [ ] 3x Fresh Ubuntu 22.04 LTS PCs (each with 1 SSD)
- [ ] Tailscale pre-auth keys generated
- [ ] Static IPs planned (192.168.1.100-102)
- [ ] tmc-cloud repo cloned with executable scripts

### 🧠 AI

- [ ] 5-10 dummy CVs created in `research/test_data/` (no real data!)

### 📍 Places

## Team Assignments

1. Matilde Campo
2. Beatrice Pazzucconi
3. Francesco Vattiato
4. Walter Maltese
5. Thomas Verardo
6. Roberto Valendino
7. Mattia Campana
8. Marco Selva
9. Flavio Renzio
10. Zaccaria Carrettoni
11. Alberto Macaluso (online)
12. Alessandro Lisi


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
- Beatrice
- Marco

---

## Project Goals & Tasks

### ☁️ Cloud: "3-Node Cluster with Distributed Storage"

**Goal**: Automated setup for 3-node cluster with networking, security, distributed storage, and visible management

**Success Criteria**:

- [ ] Single command to setup fresh PC
- [ ] All 3 nodes with static IPs and Tailscale connected
- [ ] SSH accessible with keys only (hardened)
- [ ] Firewall enabled and configured
- [ ] Distributed storage operational across 3 nodes
- [ ] Storage survives single node failure
- [ ] Portainer UI accessible showing all 3 nodes
- [ ] Grafana showing live logs from entire cluster
- [ ] Can demonstrate working cluster to management

📋 **Detailed Tasks**: See [Cloud task](./issues/hackathon-cloud-tasks.md)

### 🧠 AI: "RAG-based CV Competence Matching with Zero Data Leakage"

**Goal**: RAG-based CV competence matching system with zero data leakage (fully local)

**Architecture**: FastAPI + ChromaDB + Ollama (Llama 3.1 8B)

**Data Flow**: PDF Upload → Text Extraction → Chunking → Embedding → ChromaDB Storage → Query → RAG Retrieval → LLM Analysis → Results

**Success Criteria**:

- [ ] Docker Compose with all 3 services running (fastapi_backend, chromadb, ollama)
- [ ] CV upload ingestion pipeline working (PDF → ChromaDB)
- [ ] RAG competence matching with Llama 3.1 working
- [ ] Results preserve candidate identity (filename/cv_id)
- [ ] Zero data leakage (fully local processing)

📋 **Detailed Tasks**: See [AI Tasks](./issues/hackathon-ai-tasks.md)

### 📍 Places: "Backend Testing, Code Quality & Deployment"

**Goal**: Backend testing, code quality improvements, CI/CD setup, and deployment to tmc-cloud

**Success Criteria**:

- [ ] Backend: 5+ tests passing with coverage report
- [ ] Linting configured and passing (ruff)
- [ ] Makefile with lint/format/test commands
- [ ] CI workflow running on GitHub Actions
- [ ] Kubernetes deployment manifests created
- [ ] Deployment documentation written
- [ ] (Stretch) Application deployed to tmc-cloud

📋 **Detailed Tasks**: See [Places Tasks](./issues/hackathon-places-tasks.md)

