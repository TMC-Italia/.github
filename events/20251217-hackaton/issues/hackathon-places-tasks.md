# Places Project - Hackathon Tasks

**Duration**: 4 Hours  
**Goal**: Backend testing, code quality improvements, CI/CD setup, and deployment to tmc-cloud.

## Pre-Event Checklist

- [x] Development environment running (`cd tmc-places/backend && uv sync`)
- [x] Verify project structure

---

## Task 1: Backend Testing (90 min)

**Goal**: Setup pytest with basic test coverage for critical endpoints.

**Implementation**:

1. Create `backend/tests/` structure:
```
tests/
├── __init__.py
├── conftest.py
├── test_api.py
└── test_services.py
```

2. Add test dependencies to `backend/pyproject.toml`:
```toml
[dependency-groups]
dev = [
    "pytest>=8.0.0",
    "pytest-asyncio>=0.23.0",
    "pytest-cov>=4.1.0",
    "httpx>=0.27.0",
]
```

3. Create basic fixtures in `conftest.py`:
```python
import pytest
from fastapi.testclient import TestClient
from src.main import app

@pytest.fixture
def client():
    return TestClient(app)
```

4. Write tests for main API endpoints in `test_api.py`:
```python
def test_health_check(client):
    response = client.get("/health")
    assert response.status_code == 200

def test_get_places(client):
    response = client.get("/api/places")
    assert response.status_code == 200
```

**Acceptance Criteria**:
- [ ] Tests directory created with basic structure
- [ ] At least 5 tests covering main endpoints
- [ ] All tests pass (`cd backend && uv run pytest`)
- [ ] Basic coverage report (`uv run pytest --cov=src`)

---

## Task 2: Code Quality Tools (45 min)

**Goal**: Add ruff for linting and code formatting.

**Implementation**:

1. Add ruff to `backend/pyproject.toml`:
```toml
[dependency-groups]
dev = [
    # ... existing deps
    "ruff>=0.3.0",
]

[tool.ruff]
line-length = 100
target-version = "py311"

[tool.ruff.lint]
select = ["E", "F", "I", "N", "W"]
ignore = []
```

2. Update or create `Makefile` in project root:
```makefile
.PHONY: lint test format

lint:
	cd tmc-places/backend && uv run ruff check src/

format:
	cd tmc-places/backend && uv run ruff check --fix src/
	cd tmc-places/backend && uv run ruff format src/

test:
	cd tmc-places/backend && uv run pytest --cov=src
```

3. Run and fix linting issues:
```bash
make format
make lint
```

**Acceptance Criteria**:
- [ ] Ruff configured in pyproject.toml
- [ ] Makefile with lint/format/test commands
- [ ] All linting errors fixed
- [ ] Code formatted consistently
- [ ] `make lint` passes with no errors

---

## Task 3: CI/CD Pipeline (45 min)

**Goal**: Create GitHub Actions workflow for automated testing.

**Implementation**:

1. Create `.github/workflows/tmc-places-ci.yml` in the repository:
```yaml
name: TMC Places CI

on:
  pull_request:
    paths:
      - 'tmc-places/**'
  push:
    branches: [main]
    paths:
      - 'tmc-places/**'

jobs:
  backend-tests:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ./tmc-places/backend
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Install uv
        uses: astral-sh/setup-uv@v3
      
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      
      - name: Install dependencies
        run: uv sync
      
      - name: Run linting
        run: uv run ruff check src/
      
      - name: Run tests
        run: uv run pytest --cov=src --cov-report=term
```

**Acceptance Criteria**:
- [ ] CI workflow file created
- [ ] Workflow triggers on PR and push to main
- [ ] Backend tests run automatically
- [ ] Linting checks enforced
- [ ] Push to test the workflow

---

## Task 4: Deployment Preparation (60 min)

**Goal**: Prepare deployment configuration for tmc-cloud Kubernetes cluster.

**Implementation**:

1. Create `tmc-places/k8s/` directory:
```
k8s/
├── namespace.yaml
├── backend-deployment.yaml
├── backend-service.yaml
└── ingress.yaml
```

2. Create `namespace.yaml`:
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: tmc-places
```

3. Create `backend-deployment.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: places-backend
  namespace: tmc-places
spec:
  replicas: 2
  selector:
    matchLabels:
      app: places-backend
  template:
    metadata:
      labels:
        app: places-backend
    spec:
      containers:
      - name: backend
        image: places-backend:latest
        ports:
        - containerPort: 8000
        env:
        - name: ENVIRONMENT
          value: "production"
```

4. Create `backend-service.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: places-backend
  namespace: tmc-places
spec:
  selector:
    app: places-backend
  ports:
  - port: 80
    targetPort: 8000
  type: ClusterIP
```

5. Update `tmc-places/backend/Dockerfile.prod` if needed for optimization

6. Document deployment steps in `tmc-places/docs/deployment.md`

**Acceptance Criteria**:
- [ ] Kubernetes manifests created
- [ ] Deployment configuration reviewed
- [ ] Docker image builds successfully
- [ ] Deployment documentation written
- [ ] (Optional) Test deployment on tmc-cloud if time permits

---

## Overall Success Criteria

**Must Have**:
- [ ] Backend: 5+ tests passing
- [ ] Linting configured and passing (ruff)
- [ ] CI workflow running on GitHub Actions
- [ ] Kubernetes deployment manifests created

**Stretch Goals** (if time permits):
- [ ] Actually deploy to tmc-cloud cluster
- [ ] Coverage report >60%

---

## Quick Commands

```bash
# Setup
cd tmc-places/backend
uv sync
cp .env.example .env 

# Testing
make test                    # Run all tests
uv run pytest --cov         # Tests with coverage

# Code Quality
make lint                    # Check code quality
make format                  # Auto-fix formatting

# Docker Build
docker build -f Dockerfile.prod -t places-backend:latest .

# Kubernetes Deploy (on tmc-cloud)
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/backend-deployment.yaml
kubectl apply -f k8s/backend-service.yaml
```

---

## Time Allocation

- **Task 1 - Testing**: 90 minutes
- **Task 2 - Code Quality**: 45 minutes
- **Task 3 - CI/CD**: 45 minutes
- **Task 4 - Deployment**: 60 minutes
- **Total**: 240 minutes (4 hours)
