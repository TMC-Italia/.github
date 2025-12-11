# TMC Hackathon Plan

**Duration**: 4 Hours  
**Participants**: 7-8 People  
**Goal**: Ship one working feature per project

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
- [ ] Dependencies added to pyproject.toml: `chromadb`, `sentence-transformers`, `pypdf`, `python-multipart`
- [ ] Sentence-transformers model pre-downloaded
- [ ] Competence list defined in JSON

### 📍 Places

- [ ] mock_data.json created for Graph API testing


## Team Assignments

### ☁️ Cloud (Reviewer: Silvio)

- Marco - Hardware/OS setup
- Flavio - Network/Storage

### 🧠 AI (Reviewer: Alessandro)

- Matilde - Data/Competence definitions
- Francesco - PDF parsing & embeddings
- Beatrice - API endpoints
- Walter - Testing & integration

### 📍 Places (Reviewer: Marco)

- Alberto - Backend testing & quality
- Zaccaria - Frontend testing & UI/UX

---

## Project Goals & Tasks

### ☁️ Cloud: "PC Online and Ready"

**Goal**: One script transforms fresh Ubuntu PC → production-ready node

**Tasks**:

1. Create `scripts/bootstrap-node.sh` - idempotent setup script (Docker, essentials, system config)
2. Configure security: UFW rules, fail2ban, SSH hardening
3. Automate network: static IPs, DNS, hostname
4. Setup storage: NFS mounts with proper permissions
5. Deploy Portainer: accessible at `http://<node-ip>:9000`
6. Integrate Tailscale: auto-connect with pre-auth keys

**Success Criteria**:

- `./scripts/bootstrap-node.sh` completes without errors
- PC reachable via ping and SSH
- `docker ps` shows healthy containers
- Portainer UI accessible in browser
- `tailscale status` shows connected
- Storage mounted and writable

### 🧠 AI: "Working RAG Pipeline"

**Goal**: Upload CV PDF → get competence matches with scores

**Tasks**:

1. Create `src/app/services/pdf_parser.py` - extract text from PDFs (PyMuPDF/pypdf)
2. Add ChromaDB to docker-compose + `src/app/services/vector_store.py` wrapper
3. Create `src/app/services/embeddings.py` - Italian sentence-transformers model
4. Build API endpoints: `/api/cv/upload`, `/api/cv/search`, `/api/competences/match`
5. Prepare 5-10 dummy CVs in `research/test_data/`
6. Add basic HTML form at `/` for CV upload

**Success Criteria**:

- `POST /api/cv/upload` accepts PDF and returns extracted text
- ChromaDB stores embeddings (verify with query)
- `POST /api/cv/search` with "Python developer" returns relevant results
- Competence matching returns top 3 matches with scores
- `docker compose up` starts both services
- UI at `http://localhost:8000` allows CV upload

### 📍 Places: "Production-Ready Quality"

**Goal**: Add testing, improve code quality, enhance UI/UX

**Tasks**:

1. Backend tests: pytest with fixtures for Graph API mocking
2. Frontend tests: Vitest + React Testing Library for key components
3. Code quality: add ruff/black/mypy, improve type hints and error handling
4. UI/UX: loading states, error boundaries, accessibility, toast notifications
5. CI/CD: pipeline for tests, linting, type checking on PRs
6. E2E tests: Playwright for critical user flows
7. Documentation: testing guide and deployment checklist

**Success Criteria**:

- Backend tests pass with >80% coverage (`pytest`)
- Frontend tests pass (`npm test`)
- CI pipeline passes on test PR
- UI shows loading states and error handling
- E2E test completes: map loads → date selected → rooms show status
- Code quality tools configured (ruff, black, mypy, eslint)


