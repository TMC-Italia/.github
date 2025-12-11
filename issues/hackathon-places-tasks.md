# Places Project - Hackathon Tasks

**Duration**: 4 Hours  
**Goal**: Add testing, improve code quality, enhance UI/UX

## Pre-Event Checklist

- [ ] mock_data.json created for Graph API testing
- [ ] Development environment running (`npm install` + `uv sync`)

---

## Task 1: Backend Unit Tests (60 min)

**Goal**: Setup pytest with basic tests.

**Why**: Need automated testing for reliability.

**Implementation**:

Create `backend/tests/` structure:

```
tests/
├── __init__.py
├── conftest.py
├── test_api.py
├── test_services.py
└── test_graph_client.py
```

`conftest.py` with fixtures:

```python
import pytest
from fastapi.testclient import TestClient
from src.main import app

@pytest.fixture
def client():
    return TestClient(app)

@pytest.fixture
def mock_graph_response():
    return {
        "value": [
            {
                "id": "room-001",
                "displayName": "Meeting Room 1",
                "capacity": 10
            }
        ]
    }
```

`test_api.py`:

```python
def test_get_places(client, mock_graph_response):
    response = client.get("/api/places")
    assert response.status_code == 200
    assert "places" in response.json()

def test_get_bookings_by_date(client):
    response = client.get("/api/bookings?date=2025-12-11")
    assert response.status_code == 200
```

**Acceptance Criteria**:
- [ ] `tests/` directory with pytest structure
- [ ] Fixtures for mocking Graph API responses
- [ ] Tests for all API endpoints
- [ ] Tests for services layer
- [ ] Coverage >80% (`pytest --cov`)
- [ ] All tests pass (`pytest`)

---

## Task 2: Frontend Unit Tests (45 min)

**Goal**: Setup Vitest for component tests.

**Why**: Catch UI bugs early.

**Implementation**:

Install dependencies:

```bash
cd frontend
npm install -D vitest @testing-library/react @testing-library/jest-dom @testing-library/user-event jsdom
```

Add to `package.json`:

```json
{
  "scripts": {
    "test": "vitest",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest --coverage"
  }
}
```

Create `vite.config.js` test config:

```js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: './src/test/setup.ts',
  },
})
```

Example test `src/components/__tests__/MapView.test.tsx`:

```tsx
import { render, screen } from '@testing-library/react'
import { MapView } from '../MapView'

describe('MapView', () => {
  it('renders office map', () => {
    render(<MapView />)
    expect(screen.getByRole('img')).toBeInTheDocument()
  })

  it('highlights available rooms in green', () => {
    const rooms = [{ id: '1', status: 'available' }]
    render(<MapView rooms={rooms} />)
    expect(screen.getByTestId('room-1')).toHaveClass('available')
  })
})
```

**Acceptance Criteria**:
- [ ] Vitest configured and running
- [ ] Tests for MapView component
- [ ] Tests for RoomStatus component
- [ ] Tests for DatePicker component
- [ ] Tests for API hooks
- [ ] `npm test` passes all tests

---

## Task 3: Code Quality Tools (30 min)

**Goal**: Add ruff and mypy to backend.

**Why**: Consistent code style and type checking.

**Implementation**:

Update `pyproject.toml`:

```toml
[dependency-groups]
dev = [
    "pytest>=8.0.0",
    "pytest-asyncio>=0.23.0",
    "pytest-cov>=4.1.0",
    "httpx>=0.27.0",
    "ruff>=0.3.0",
    "black>=24.0.0",
    "mypy>=1.9.0",
]

[tool.ruff]
line-length = 100
target-version = "py311"

[tool.ruff.lint]
select = ["E", "F", "I", "N", "W"]

[tool.black]
line-length = 100
target-version = ['py311']

[tool.mypy]
python_version = "3.11"
strict = true
```

Add to `Makefile`:

```makefile
lint:
	cd backend && uv run ruff check src/
	cd backend && uv run mypy src/

format:
	cd backend && uv run black src/
	cd backend && uv run ruff check --fix src/

test:
	cd backend && uv run pytest --cov=src --cov-report=html
```

**Acceptance Criteria**:
- [ ] Ruff configured and passes
- [ ] Black formats code consistently
- [ ] Mypy type checking passes
- [ ] All imports sorted (ruff I)
- [ ] No linting errors
- [ ] `make lint` passes

---

## Task 4: UI/UX Improvements (60 min)

**Goal**: Add loading states and error handling.

**Why**: Better user experience.

**Implementation**:

Create `src/components/LoadingSpinner.tsx`:

```tsx
export const LoadingSpinner = () => (
  <div className="flex items-center justify-center p-8">
    <div className="animate-spin rounded-full h-12 w-12 border-b-2 border-blue-500" />
  </div>
)
```

Create `src/components/ErrorBoundary.tsx`:

```tsx
import { Component, ReactNode } from 'react'

export class ErrorBoundary extends Component<
  { children: ReactNode },
  { hasError: boolean }
> {
  state = { hasError: false }

  static getDerivedStateFromError() {
    return { hasError: true }
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="p-4 bg-red-50 border border-red-200 rounded">
          <h2 className="text-red-800 font-bold">Something went wrong</h2>
          <button onClick={() => this.setState({ hasError: false })}>
            Try again
          </button>
        </div>
      )
    }
    return this.props.children
  }
}
```

Add toast notifications using `react-hot-toast`:

```bash
npm install react-hot-toast
```

Update components to use loading/error states:

```tsx
const { data, isLoading, error } = useQuery({
  queryKey: ['places'],
  queryFn: fetchPlaces
})

if (isLoading) return <LoadingSpinner />
if (error) return <ErrorMessage error={error} />
```

Add ARIA labels for accessibility:

```tsx
<button
  aria-label="Select date"
  aria-pressed={isSelected}
  onClick={handleClick}
>
  {date}
</button>
```

**Acceptance Criteria**:
- [ ] Loading spinner during API calls
- [ ] Error boundary catches component errors
- [ ] Toast notifications for success/errors
- [ ] All interactive elements have ARIA labels
- [ ] Keyboard navigation works
- [ ] Responsive design on mobile

---

## Task 5: CI/CD Pipeline (30 min)

**Goal**: Create basic CI workflow.

**Why**: Automate testing on PRs.

**Implementation**:

Create `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  backend-tests:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ./backend
    steps:
      - uses: actions/checkout@v4
      
      - name: Install uv
        uses: astral-sh/setup-uv@v1
      
      - name: Install dependencies
        run: uv sync
      
      - name: Run linting
        run: |
          uv run ruff check src/
          uv run mypy src/
      
      - name: Run tests
        run: uv run pytest --cov=src --cov-report=xml
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          file: ./backend/coverage.xml

  frontend-tests:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ./frontend
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run linting
        run: npm run lint
      
      - name: Run tests
        run: npm test
```

**Acceptance Criteria**:
- [ ] CI workflow file created
- [ ] Backend tests run on PR
- [ ] Frontend tests run on PR
- [ ] Linting checks enforced
- [ ] Type checking enforced
- [ ] Coverage reports uploaded
- [ ] PR blocked if checks fail

---

## Task 6: E2E Testing with Playwright (SKIP - Out of scope)

**Goal**: ~~Setup Playwright~~ **Move to post-hackathon backlog**.

**Why**: 4 hours isn't enough for E2E setup + other priorities.

**Implementation**:

Install Playwright:

```bash
cd frontend
npm install -D @playwright/test
npx playwright install
```

Create `e2e/` directory:

```
e2e/
├── fixtures/
│   └── mock-data.json
└── tests/
    └── booking-flow.spec.ts
```

`e2e/tests/booking-flow.spec.ts`:

```ts
import { test, expect } from '@playwright/test'

test.describe('Booking visualization flow', () => {
  test('should load map and show room availability', async ({ page }) => {
    await page.goto('http://localhost:3000')
    
    // Wait for map to load
    await expect(page.locator('[data-testid="office-map"]')).toBeVisible()
    
    // Select date
    await page.click('[data-testid="date-picker"]')
    await page.click('[data-testid="date-2025-12-11"]')
    
    // Verify rooms are colored
    const availableRooms = await page.locator('.room-available').count()
    expect(availableRooms).toBeGreaterThan(0)
    
    // Click on a room
    await page.click('[data-testid="room-101"]')
    
    // Verify details modal opens
    await expect(page.locator('[data-testid="room-details"]')).toBeVisible()
  })
})
```

Add npm script:

```json
{
  "scripts": {
    "e2e": "playwright test",
    "e2e:ui": "playwright test --ui"
  }
}
```

**Acceptance Criteria**:
- [ ] Playwright installed and configured
- [ ] E2E test for main booking flow
- [ ] Test: load map → select date → view room status
- [ ] Test runs in headless mode
- [ ] `npm run e2e` passes
- [ ] Screenshots on failure

---

## Task 7: Documentation Updates (SKIP - Out of scope)

**Goal**: ~~Update README~~ **Move to post-hackathon backlog**.

**Why**: Focus on working code first, docs later.

**Implementation**:

Update `README.md` sections:

```markdown
## Development

### Backend
```bash
cd backend
uv sync
uv run uvicorn src.main:app --reload
```

### Frontend
```bash
cd frontend
npm install
npm run dev
```

## Testing

### Backend Tests
```bash
cd backend
uv run pytest --cov  # Run tests with coverage
uv run ruff check src/  # Linting
uv run mypy src/  # Type checking
```

### Frontend Tests
```bash
cd frontend
npm test  # Unit tests
npm run e2e  # E2E tests
npm run lint  # ESLint
```

## CI/CD

Pull requests automatically run:
- Backend: pytest, ruff, mypy
- Frontend: vitest, eslint, playwright

Coverage reports uploaded to Codecov.

## Deployment Checklist

- [ ] All tests passing
- [ ] Linting clean
- [ ] Type checking passing
- [ ] E2E tests passing
- [ ] Environment variables set
- [ ] Database migrations run
- [ ] Monitoring configured
```

**Acceptance Criteria**:
- [ ] README has development section
- [ ] Testing instructions documented
- [ ] CI/CD process explained
- [ ] Deployment checklist added
- [ ] Troubleshooting section
- [ ] API documentation linked

---

## Overall Success Criteria

**Must Have**:
- [ ] Backend: 5+ tests passing
- [ ] Frontend: 3+ component tests passing
- [ ] Linting configured (ruff, eslint)
- [ ] Loading spinners in UI
- [ ] Error boundaries in UI
- [ ] CI workflow file created

**Stretch Goals** (if time permits):
- [ ] Toast notifications
- [ ] Type checking with mypy
- [ ] Coverage >50%

---

## Quick Start

```bash
# Backend tests
cd backend
uv run pytest --cov

# Frontend tests
cd frontend
npm test

# E2E tests (requires both backend and frontend running)
cd frontend
npm run e2e

# Run all checks
make lint
make test
```

## Testing Strategy

1. **Unit Tests**: Test individual functions and components in isolation
2. **Integration Tests**: Test API endpoints with mocked dependencies
3. **E2E Tests**: Test complete user workflows in browser
4. **Manual Testing**: Visual QA on staging environment
