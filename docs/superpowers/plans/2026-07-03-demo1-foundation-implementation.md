# Demo 1 Foundation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the first runnable agentic-SOAR demo slice: event ingestion, normalized storage, alert/case creation, deterministic analysis, local LLM analysis contract, comparison, recommend-only responses, audit trail, and minimal SOC dashboard.

**Architecture:** Use a small monorepo with a Python FastAPI backend, PostgreSQL persistence, React/Vite dashboard, Docker Compose lab services, and scenario fixtures. The SOAR core stays independent from a specific LLM model by routing all LLM calls through a local runtime adapter, with Ollama as the first supported runtime.

**Tech Stack:** Python 3.12, FastAPI, Pydantic v2, SQLAlchemy 2, Alembic, PostgreSQL 16, pytest, Ruff, Docker Compose, Node 22, React, TypeScript, Vite, Vitest, Ollama-compatible HTTP API.

## Global Constraints

- Demo 1 is recommend-only; no infrastructure-changing response action may execute.
- Demo 1 must support the flow: `event -> alert -> case -> investigation -> deterministic/LLM comparison -> recommendation -> audit -> SOC dashboard`.
- The canonical event schema is internal and minimal, inspired by ECS/OCSF, and must remain mappable to standards later.
- PostgreSQL stores events, alerts, cases, recommendations, and audit records.
- The LLM runs locally through an abstract provider interface; Ollama is the first runtime adapter.
- The local development profile targets an NVIDIA GeForce RTX 3050 with 4 GB VRAM, so the LLM receives compact case packets rather than raw log streams.
- Public cybersecurity datasets are optional import sources, not required to start Demo 1.
- Keep scenario fixtures small and repository-friendly.
- Preserve raw event references so normalized events remain auditable.
- Preserve deterministic and LLM outputs for comparison and reproducibility.
- The repo must remain buildable and testable without a specific coding-agent product.

---

## Planned File Structure

```text
README.md
pyproject.toml
package.json
docker-compose.yml
.env.example

backend/
  app/
    main.py
    config.py
    db.py
    models.py
    schemas/
      events.py
      alerts.py
      cases.py
      analysis.py
      recommendations.py
    repositories/
      events.py
      alerts.py
      cases.py
      audit.py
    services/
      ingestion.py
      triggers.py
      case_manager.py
      deterministic_engine.py
      case_packet.py
      llm_provider.py
      comparison.py
      recommendations.py
      audit.py
    api/
      health.py
      events.py
      alerts.py
      cases.py
      demo.py
  alembic/
  tests/

frontend/
  package.json
  index.html
  src/
    main.tsx
    api.ts
    App.tsx
    components/
      AlertsView.tsx
      CasesView.tsx
      CaseDetail.tsx
      AnalysisComparison.tsx
      RecommendationsPanel.tsx
    styles.css
  tests/

scenarios/
  demo1/
    ssh_bruteforce.json
    port_scan.json
    web_attack.json
    beacon_c2.json

labs/
  demo1/
    README.md
    nginx/
      default.conf
      index.html

docs/
  adr/
  phases/
    2026-07-03-demo1-foundation/
      SPEC.md
      PLAN.md
      UAT.md
      EVAL.md
      REVIEW.md
```

## Task 1: Repository Skeleton And Tooling

**Files:**
- Create: `README.md`
- Create: `.env.example`
- Create: `pyproject.toml`
- Create: `package.json`
- Create: `backend/app/main.py`
- Create: `backend/app/config.py`
- Create: `backend/tests/test_health.py`
- Create: `frontend/package.json`
- Create: `frontend/src/main.tsx`
- Create: `frontend/src/App.tsx`
- Create: `frontend/tests/App.test.tsx`

**Interfaces:**
- Produces: runnable backend health endpoint `GET /health`.
- Produces: runnable frontend app shell with title `Agentic-SOAR Demo 1`.
- Produces: base commands `pytest`, `ruff check`, `npm --prefix frontend test`.

- [ ] **Step 1: Create backend package and health test**

Create `backend/tests/test_health.py`:

```python
from fastapi.testclient import TestClient

from backend.app.main import app


def test_health_endpoint_returns_ok() -> None:
    client = TestClient(app)

    response = client.get("/health")

    assert response.status_code == 200
    assert response.json() == {"status": "ok"}
```

- [ ] **Step 2: Run backend test to verify it fails**

Run: `pytest backend/tests/test_health.py -v`

Expected: FAIL because `backend.app.main` does not exist yet.

- [ ] **Step 3: Create minimal backend implementation**

Create `backend/app/main.py`:

```python
from fastapi import FastAPI

app = FastAPI(title="Agentic-SOAR Demo 1")


@app.get("/health")
def health() -> dict[str, str]:
    return {"status": "ok"}
```

Create `backend/app/config.py`:

```python
from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    database_url: str = "postgresql+psycopg://agentic_soar:agentic_soar@localhost:5432/agentic_soar"
    ollama_base_url: str = "http://localhost:11434"
    ollama_model: str = "llama3.2:3b-instruct-q4_K_M"

    model_config = SettingsConfigDict(env_file=".env", env_prefix="SOAR_")


settings = Settings()
```

- [ ] **Step 4: Create project tool configs**

Create `pyproject.toml`:

```toml
[project]
name = "agentic-soar"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = [
  "alembic>=1.13",
  "fastapi>=0.115",
  "httpx>=0.27",
  "psycopg[binary]>=3.2",
  "pydantic>=2.8",
  "pydantic-settings>=2.4",
  "sqlalchemy>=2.0",
  "uvicorn[standard]>=0.30"
]

[project.optional-dependencies]
dev = [
  "pytest>=8.3",
  "ruff>=0.6"
]

[tool.pytest.ini_options]
pythonpath = ["."]

[tool.ruff]
line-length = 100
target-version = "py312"
```

Create `.env.example`:

```text
SOAR_DATABASE_URL=postgresql+psycopg://agentic_soar:agentic_soar@localhost:5432/agentic_soar
SOAR_OLLAMA_BASE_URL=http://localhost:11434
SOAR_OLLAMA_MODEL=llama3.2:3b-instruct-q4_K_M
```

Create root `package.json`:

```json
{
  "scripts": {
    "test": "pytest backend/tests -v && npm --prefix frontend test",
    "lint": "ruff check backend"
  }
}
```

- [ ] **Step 5: Create frontend shell and test**

Create `frontend/package.json`:

```json
{
  "scripts": {
    "dev": "vite --host 0.0.0.0",
    "build": "vite build",
    "test": "vitest run"
  },
  "dependencies": {
    "@vitejs/plugin-react": "^4.3.0",
    "vite": "^5.4.0",
    "typescript": "^5.5.0",
    "react": "^18.3.0",
    "react-dom": "^18.3.0"
  },
  "devDependencies": {
    "@testing-library/react": "^16.0.0",
    "@testing-library/jest-dom": "^6.4.0",
    "jsdom": "^25.0.0",
    "vitest": "^2.0.0"
  }
}
```

Create `frontend/src/main.tsx`:

```tsx
import React from "react";
import ReactDOM from "react-dom/client";
import { App } from "./App";

ReactDOM.createRoot(document.getElementById("root") as HTMLElement).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

Create `frontend/src/App.tsx`:

```tsx
export function App() {
  return (
    <main>
      <h1>Agentic-SOAR Demo 1</h1>
      <p>Minimal SOC dashboard shell</p>
    </main>
  );
}
```

Create `frontend/tests/App.test.tsx`:

```tsx
import { render, screen } from "@testing-library/react";
import { describe, expect, it } from "vitest";
import { App } from "../src/App";

describe("App", () => {
  it("renders the dashboard shell", () => {
    render(<App />);
    expect(screen.getByText("Agentic-SOAR Demo 1")).toBeInTheDocument();
  });
});
```

- [ ] **Step 6: Run tests**

Run: `pytest backend/tests/test_health.py -v`

Expected: PASS.

Run: `npm --prefix frontend install`

Expected: dependencies install successfully.

Run: `npm --prefix frontend test`

Expected: PASS.

- [ ] **Step 7: Commit**

```bash
git add README.md .env.example pyproject.toml package.json backend frontend
git commit -m "chore: add demo1 project skeleton"
```

## Task 2: PostgreSQL Persistence And Canonical Models

**Files:**
- Create: `backend/app/db.py`
- Create: `backend/app/models.py`
- Create: `backend/app/schemas/events.py`
- Create: `backend/app/schemas/alerts.py`
- Create: `backend/app/schemas/cases.py`
- Create: `backend/app/schemas/analysis.py`
- Create: `backend/app/schemas/recommendations.py`
- Create: `backend/tests/test_models.py`
- Create: `docker-compose.yml`

**Interfaces:**
- Produces: SQLAlchemy models `EventRecord`, `AlertRecord`, `CaseRecord`, `AnalysisRecord`, `RecommendationRecord`, `AuditRecord`.
- Produces: Pydantic schema `CanonicalEvent`.
- Consumes: `settings.database_url` from Task 1.

- [ ] **Step 1: Write schema test**

Create `backend/tests/test_models.py`:

```python
from datetime import UTC, datetime

from backend.app.schemas.events import CanonicalEvent


def test_canonical_event_accepts_minimal_demo1_event() -> None:
    event = CanonicalEvent(
        event_id="evt-1",
        timestamp=datetime(2026, 7, 3, 1, 0, tzinfo=UTC),
        source_type="auth_logs",
        event_type="auth_failure",
        severity="medium",
        asset="internal-linux-host",
        user="alice",
        src_ip="10.10.99.10",
        dst_ip="10.10.20.10",
        labels=["ssh", "bruteforce"],
        raw_ref="scenario:ssh_bruteforce:1",
    )

    assert event.attack is None
    assert event.execution_allowed is False
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest backend/tests/test_models.py -v`

Expected: FAIL because `CanonicalEvent` does not exist.

- [ ] **Step 3: Implement canonical event schema**

Create `backend/app/schemas/events.py`:

```python
from datetime import datetime
from typing import Literal

from pydantic import BaseModel, Field

Severity = Literal["low", "medium", "high", "critical"]


class AttackMapping(BaseModel):
    technique_id: str
    tactic: str


class CanonicalEvent(BaseModel):
    event_id: str
    timestamp: datetime
    source_type: str
    event_type: str
    severity: Severity
    asset: str | None = None
    user: str | None = None
    src_ip: str | None = None
    dst_ip: str | None = None
    src_port: int | None = None
    dst_port: int | None = None
    protocol: str | None = None
    process: str | None = None
    url: str | None = None
    domain: str | None = None
    labels: list[str] = Field(default_factory=list)
    attack: AttackMapping | None = None
    raw_ref: str
    execution_allowed: bool = False
```

- [ ] **Step 4: Add database models**

Create `backend/app/db.py`:

```python
from collections.abc import Generator

from sqlalchemy import create_engine
from sqlalchemy.orm import DeclarativeBase, Session, sessionmaker

from backend.app.config import settings


class Base(DeclarativeBase):
    pass


engine = create_engine(settings.database_url)
SessionLocal = sessionmaker(bind=engine, expire_on_commit=False)


def get_session() -> Generator[Session, None, None]:
    with SessionLocal() as session:
        yield session
```

Create `backend/app/models.py`:

```python
from datetime import datetime
from typing import Any
from uuid import uuid4

from sqlalchemy import JSON, DateTime, String, Text
from sqlalchemy.orm import Mapped, mapped_column

from backend.app.db import Base


def new_id(prefix: str) -> str:
    return f"{prefix}-{uuid4().hex}"


class EventRecord(Base):
    __tablename__ = "events"

    event_id: Mapped[str] = mapped_column(String, primary_key=True)
    timestamp: Mapped[datetime] = mapped_column(DateTime(timezone=True), index=True)
    source_type: Mapped[str] = mapped_column(String, index=True)
    event_type: Mapped[str] = mapped_column(String, index=True)
    severity: Mapped[str] = mapped_column(String, index=True)
    asset: Mapped[str | None] = mapped_column(String, nullable=True, index=True)
    user: Mapped[str | None] = mapped_column(String, nullable=True, index=True)
    src_ip: Mapped[str | None] = mapped_column(String, nullable=True, index=True)
    dst_ip: Mapped[str | None] = mapped_column(String, nullable=True, index=True)
    raw_ref: Mapped[str] = mapped_column(Text)
    payload: Mapped[dict[str, Any]] = mapped_column(JSON)


class AlertRecord(Base):
    __tablename__ = "alerts"

    alert_id: Mapped[str] = mapped_column(String, primary_key=True, default=lambda: new_id("alert"))
    title: Mapped[str] = mapped_column(String)
    scenario: Mapped[str] = mapped_column(String, index=True)
    severity: Mapped[str] = mapped_column(String, index=True)
    status: Mapped[str] = mapped_column(String, default="new", index=True)
    event_ids: Mapped[list[str]] = mapped_column(JSON)
    payload: Mapped[dict[str, Any]] = mapped_column(JSON)


class CaseRecord(Base):
    __tablename__ = "cases"

    case_id: Mapped[str] = mapped_column(String, primary_key=True, default=lambda: new_id("case"))
    title: Mapped[str] = mapped_column(String)
    status: Mapped[str] = mapped_column(String, default="new", index=True)
    severity: Mapped[str] = mapped_column(String, index=True)
    payload: Mapped[dict[str, Any]] = mapped_column(JSON)


class AnalysisRecord(Base):
    __tablename__ = "analyses"

    analysis_id: Mapped[str] = mapped_column(String, primary_key=True, default=lambda: new_id("analysis"))
    case_id: Mapped[str] = mapped_column(String, index=True)
    engine: Mapped[str] = mapped_column(String, index=True)
    payload: Mapped[dict[str, Any]] = mapped_column(JSON)


class RecommendationRecord(Base):
    __tablename__ = "recommendations"

    recommendation_id: Mapped[str] = mapped_column(String, primary_key=True, default=lambda: new_id("rec"))
    case_id: Mapped[str] = mapped_column(String, index=True)
    action_type: Mapped[str] = mapped_column(String, index=True)
    status: Mapped[str] = mapped_column(String, default="proposed")
    execution_allowed: Mapped[bool] = mapped_column(default=False)
    payload: Mapped[dict[str, Any]] = mapped_column(JSON)


class AuditRecord(Base):
    __tablename__ = "audit_records"

    audit_id: Mapped[str] = mapped_column(String, primary_key=True, default=lambda: new_id("audit"))
    case_id: Mapped[str | None] = mapped_column(String, nullable=True, index=True)
    event_type: Mapped[str] = mapped_column(String, index=True)
    payload: Mapped[dict[str, Any]] = mapped_column(JSON)
```

- [ ] **Step 5: Create focused schemas required by later services**

Create `backend/app/schemas/alerts.py`:

```python
from pydantic import BaseModel, Field


class Alert(BaseModel):
    alert_id: str
    title: str
    scenario: str
    severity: str
    status: str = "new"
    event_ids: list[str] = Field(default_factory=list)
    payload: dict = Field(default_factory=dict)
```

Create `backend/app/schemas/cases.py`:

```python
from pydantic import BaseModel, Field


class EvidenceItem(BaseModel):
    event_id: str
    reason: str


class Case(BaseModel):
    case_id: str
    title: str
    status: str = "new"
    severity: str
    source_alerts: list[str] = Field(default_factory=list)
    affected_entities: dict = Field(default_factory=dict)
    timeline: list[dict] = Field(default_factory=list)
    evidence: list[EvidenceItem] = Field(default_factory=list)
    ground_truth: dict = Field(default_factory=dict)
```

Create `backend/app/schemas/analysis.py`:

```python
from pydantic import BaseModel, Field


class AnalysisResult(BaseModel):
    engine: str
    severity: str
    confidence: float
    summary: str
    evidence_refs: list[str] = Field(default_factory=list)
    recommended_actions: list[dict] = Field(default_factory=list)
    limitations: list[str] = Field(default_factory=list)
    metadata: dict = Field(default_factory=dict)


class ComparisonResult(BaseModel):
    case_id: str
    agreements: list[str] = Field(default_factory=list)
    disagreements: list[str] = Field(default_factory=list)
    risk_notes: list[str] = Field(default_factory=list)
```

Create `backend/app/schemas/recommendations.py`:

```python
from pydantic import BaseModel


class Recommendation(BaseModel):
    recommendation_id: str
    case_id: str
    action_type: str
    target: str
    risk: str
    required_approval: bool
    rationale: str
    expected_effect: str
    rollback_hint: str
    source_engine: str
    confidence: float
    status: str = "proposed"
    execution_allowed: bool = False
```

- [ ] **Step 6: Add Docker Compose database**

Create `docker-compose.yml`:

```yaml
services:
  postgres:
    image: postgres:16
    environment:
      POSTGRES_USER: agentic_soar
      POSTGRES_PASSWORD: agentic_soar
      POSTGRES_DB: agentic_soar
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data

volumes:
  postgres-data:
```

- [ ] **Step 7: Run tests**

Run: `pytest backend/tests/test_models.py -v`

Expected: PASS.

- [ ] **Step 8: Commit**

```bash
git add backend docker-compose.yml
git commit -m "feat: add canonical schema and persistence models"
```

## Task 3: Scenario Fixtures And Event Generator

**Files:**
- Create: `scenarios/demo1/ssh_bruteforce.json`
- Create: `scenarios/demo1/port_scan.json`
- Create: `scenarios/demo1/web_attack.json`
- Create: `scenarios/demo1/beacon_c2.json`
- Create: `backend/app/services/ingestion.py`
- Create: `backend/tests/test_ingestion.py`

**Interfaces:**
- Consumes: `CanonicalEvent`.
- Produces: `load_scenario_events(path: Path) -> list[CanonicalEvent]`.

- [ ] **Step 1: Write ingestion test for SSH brute force fixture**

Create `backend/tests/test_ingestion.py`:

```python
from pathlib import Path

from backend.app.services.ingestion import load_scenario_events


def test_loads_ssh_bruteforce_fixture() -> None:
    events = load_scenario_events(Path("scenarios/demo1/ssh_bruteforce.json"))

    assert len(events) >= 3
    assert {event.event_type for event in events} >= {"auth_failure"}
    assert all(event.raw_ref.startswith("scenario:ssh_bruteforce") for event in events)
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest backend/tests/test_ingestion.py -v`

Expected: FAIL because fixture and service do not exist.

- [ ] **Step 3: Create SSH fixture**

Create `scenarios/demo1/ssh_bruteforce.json` with at least five failed SSH login events from `10.10.99.10` to `internal-linux-host`, mapped to MITRE ATT&CK `T1110`.

- [ ] **Step 4: Create remaining scenario fixtures**

Create:

- `port_scan.json` with many destination ports from one source.
- `web_attack.json` with suspicious URL paths against `dmz-web`.
- `beacon_c2.json` with periodic DNS or outbound events from `internal-linux-host`.

- [ ] **Step 5: Implement ingestion loader**

Create `backend/app/services/ingestion.py`:

```python
import json
from pathlib import Path

from backend.app.schemas.events import CanonicalEvent


def load_scenario_events(path: Path) -> list[CanonicalEvent]:
    payload = json.loads(path.read_text())
    return [CanonicalEvent.model_validate(item) for item in payload["events"]]
```

- [ ] **Step 6: Run tests**

Run: `pytest backend/tests/test_ingestion.py -v`

Expected: PASS.

- [ ] **Step 7: Commit**

```bash
git add scenarios backend/app/services/ingestion.py backend/tests/test_ingestion.py
git commit -m "feat: add demo1 scenario fixtures"
```

## Task 4: Triggers, Alerts, And Cases

**Files:**
- Create: `backend/app/services/triggers.py`
- Create: `backend/app/services/case_manager.py`
- Create: `backend/tests/test_triggers_cases.py`

**Interfaces:**
- Consumes: `list[CanonicalEvent]`.
- Produces: `detect_alerts(events: list[CanonicalEvent]) -> list[Alert]`.
- Produces: `create_case_from_alert(alert: Alert, events: list[CanonicalEvent]) -> Case`.

- [ ] **Step 1: Write trigger and case tests**

Create tests for:

- SSH brute force creates `possible_ssh_bruteforce`.
- Port scan creates `possible_reconnaissance`.
- Web attack creates `possible_web_attack`.
- Beacon creates `suspicious_outbound_beacon`.
- Each alert creates a case with affected entities, timeline, evidence, and ground truth.

- [ ] **Step 2: Run tests to verify they fail**

Run: `pytest backend/tests/test_triggers_cases.py -v`

Expected: FAIL because services do not exist.

- [ ] **Step 3: Implement trigger rules**

Create deterministic alert rules with explicit thresholds:

- SSH brute force: at least five `auth_failure` events from the same `src_ip` to the same `asset`.
- Port scan: at least eight distinct destination ports from the same `src_ip`.
- Web attack: suspicious URL contains one of `../`, `union select`, `<script`, `/etc/passwd`.
- Beacon: at least four DNS or outbound network events to the same domain with regular intervals.

- [ ] **Step 4: Implement case manager**

Create cases with:

- `case_id`
- `title`
- `status = "new"`
- `severity`
- `source_alerts`
- `affected_entities`
- `timeline`
- `evidence`
- `ground_truth`

- [ ] **Step 5: Run tests**

Run: `pytest backend/tests/test_triggers_cases.py -v`

Expected: PASS.

- [ ] **Step 6: Commit**

```bash
git add backend/app/services/triggers.py backend/app/services/case_manager.py backend/tests/test_triggers_cases.py
git commit -m "feat: create alerts and cases from demo events"
```

## Task 5: Deterministic Analysis Engine

**Files:**
- Create: `backend/app/services/deterministic_engine.py`
- Create: `backend/tests/test_deterministic_engine.py`

**Interfaces:**
- Consumes: `Case`.
- Produces: `run_deterministic_analysis(case: Case) -> AnalysisResult`.

- [ ] **Step 1: Write deterministic analysis tests**

Create one test per initial scenario verifying:

- `engine == "deterministic"`
- severity is expected.
- cited evidence references are present.
- recommendations are recommend-only proposals.

- [ ] **Step 2: Run tests to verify they fail**

Run: `pytest backend/tests/test_deterministic_engine.py -v`

Expected: FAIL.

- [ ] **Step 3: Implement deterministic analysis**

Implement fixed logic:

- SSH brute force: recommend reviewing affected account and blocking source IP.
- Port scan: recommend monitoring source IP and adding a temporary detection.
- Web attack: recommend reviewing web logs and adding WAF-style detection.
- Beacon/C2: recommend blocking domain and inspecting host.

- [ ] **Step 4: Run tests**

Run: `pytest backend/tests/test_deterministic_engine.py -v`

Expected: PASS.

- [ ] **Step 5: Commit**

```bash
git add backend/app/services/deterministic_engine.py backend/tests/test_deterministic_engine.py
git commit -m "feat: add deterministic analysis baseline"
```

## Task 6: Local LLM Adapter And Case Packet Builder

**Files:**
- Create: `backend/app/services/case_packet.py`
- Create: `backend/app/services/llm_provider.py`
- Create: `backend/tests/test_case_packet.py`
- Create: `backend/tests/test_llm_provider.py`

**Interfaces:**
- Consumes: `Case`, deterministic `AnalysisResult`.
- Produces: `build_case_packet(case: Case, baseline: AnalysisResult) -> CasePacket`.
- Produces: `LocalLLMProvider.analyze(packet: CasePacket) -> AnalysisResult`.

- [ ] **Step 1: Write case packet test**

Verify the packet includes:

- case summary
- alert
- entities
- reduced timeline
- top evidence
- deterministic baseline result
- policy constraints
- allowed recommendation types

- [ ] **Step 2: Write LLM provider test with mocked HTTP**

Mock the Ollama-compatible API and verify:

- model name is sent.
- prompt contains case packet evidence refs.
- structured JSON output is parsed.
- timeout returns a captured error result rather than crashing.

- [ ] **Step 3: Run tests to verify they fail**

Run: `pytest backend/tests/test_case_packet.py backend/tests/test_llm_provider.py -v`

Expected: FAIL.

- [ ] **Step 4: Implement case packet builder**

Ensure raw logs are not passed wholesale. Include only reduced timeline and top evidence.

- [ ] **Step 5: Implement Ollama-compatible adapter**

Use `httpx` against `POST /api/generate` or equivalent Ollama endpoint. Capture:

- runtime
- model
- prompt template version
- response
- latency
- errors

- [ ] **Step 6: Run tests**

Run: `pytest backend/tests/test_case_packet.py backend/tests/test_llm_provider.py -v`

Expected: PASS.

- [ ] **Step 7: Commit**

```bash
git add backend/app/services/case_packet.py backend/app/services/llm_provider.py backend/tests/test_case_packet.py backend/tests/test_llm_provider.py
git commit -m "feat: add local llm analysis adapter"
```

## Task 7: Comparison, Recommendation Policy, And Audit

**Files:**
- Create: `backend/app/services/comparison.py`
- Create: `backend/app/services/recommendations.py`
- Create: `backend/app/services/audit.py`
- Create: `backend/tests/test_comparison_recommendations.py`

**Interfaces:**
- Consumes: deterministic `AnalysisResult`, LLM `AnalysisResult`, `Case`.
- Produces: `compare_analysis(...) -> ComparisonResult`.
- Produces: `build_recommendations(...) -> list[Recommendation]`.
- Produces: `record_audit(event_type: str, payload: dict) -> AuditEvent`.

- [ ] **Step 1: Write tests**

Verify:

- agreement and disagreement are captured.
- recommendations always have `status = "proposed"`.
- recommendations always have `execution_allowed = false`.
- unsafe LLM action requests are marked unsupported.
- audit records include case id and source engine.

- [ ] **Step 2: Run tests to verify they fail**

Run: `pytest backend/tests/test_comparison_recommendations.py -v`

Expected: FAIL.

- [ ] **Step 3: Implement comparison**

Compare:

- severity
- affected entities
- cited evidence
- recommended actions
- confidence

- [ ] **Step 4: Implement recommend-only policy**

Force all Demo 1 recommendations to:

```text
status = proposed
execution_allowed = false
```

- [ ] **Step 5: Implement audit events**

Audit event types:

- `event_ingested`
- `alert_created`
- `case_created`
- `deterministic_analysis_completed`
- `llm_analysis_completed`
- `comparison_completed`
- `recommendation_created`

- [ ] **Step 6: Run tests**

Run: `pytest backend/tests/test_comparison_recommendations.py -v`

Expected: PASS.

- [ ] **Step 7: Commit**

```bash
git add backend/app/services/comparison.py backend/app/services/recommendations.py backend/app/services/audit.py backend/tests/test_comparison_recommendations.py
git commit -m "feat: compare analyses and enforce recommend-only policy"
```

## Task 8: API Endpoints And Demo Runner

**Files:**
- Create: `backend/app/api/health.py`
- Create: `backend/app/api/events.py`
- Create: `backend/app/api/alerts.py`
- Create: `backend/app/api/cases.py`
- Create: `backend/app/api/demo.py`
- Modify: `backend/app/main.py`
- Create: `backend/tests/test_api_demo.py`

**Interfaces:**
- Produces: `POST /demo/run/{scenario}`.
- Produces: `GET /alerts`.
- Produces: `GET /cases`.
- Produces: `GET /cases/{case_id}`.

- [ ] **Step 1: Write API integration test**

Verify:

- `POST /demo/run/ssh_bruteforce` returns created alert and case.
- `GET /cases` includes the created case.
- `GET /cases/{case_id}` returns evidence, deterministic analysis, LLM analysis or LLM error, comparison, recommendations, and audit refs.

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest backend/tests/test_api_demo.py -v`

Expected: FAIL.

- [ ] **Step 3: Implement routers**

Create focused routers and include them in `backend/app/main.py`.

- [ ] **Step 4: Implement demo runner**

The demo runner executes the full pipeline for one scenario:

```text
load fixture -> trigger alert -> create case -> deterministic analysis -> case packet -> LLM analysis -> compare -> recommend -> audit
```

- [ ] **Step 5: Run tests**

Run: `pytest backend/tests/test_api_demo.py -v`

Expected: PASS.

- [ ] **Step 6: Commit**

```bash
git add backend/app/api backend/app/main.py backend/tests/test_api_demo.py
git commit -m "feat: expose demo1 soar workflow api"
```

## Task 9: Minimal SOC Dashboard

**Files:**
- Create: `frontend/src/api.ts`
- Create: `frontend/src/components/AlertsView.tsx`
- Create: `frontend/src/components/CasesView.tsx`
- Create: `frontend/src/components/CaseDetail.tsx`
- Create: `frontend/src/components/AnalysisComparison.tsx`
- Create: `frontend/src/components/RecommendationsPanel.tsx`
- Modify: `frontend/src/App.tsx`
- Create: `frontend/src/styles.css`
- Create: `frontend/tests/dashboard.test.tsx`

**Interfaces:**
- Consumes: backend API from Task 8.
- Produces: dashboard views for alerts, cases, case detail, comparison, recommendations.

- [ ] **Step 1: Write UI tests**

Mock API responses and verify the UI renders:

- alerts list
- cases list
- case timeline
- deterministic analysis
- LLM analysis
- side-by-side comparison
- recommend-only actions

- [ ] **Step 2: Run test to verify it fails**

Run: `npm --prefix frontend test`

Expected: FAIL.

- [ ] **Step 3: Implement API client**

Create typed functions:

- `runScenario(scenario: string)`
- `listAlerts()`
- `listCases()`
- `getCase(caseId: string)`

- [ ] **Step 4: Implement dashboard components**

Use compact, operational UI:

- no marketing hero
- no decorative card nesting
- dense case tables
- clear severity/status labels
- readable comparison panels

- [ ] **Step 5: Run tests**

Run: `npm --prefix frontend test`

Expected: PASS.

- [ ] **Step 6: Commit**

```bash
git add frontend
git commit -m "feat: add minimal soc dashboard"
```

## Task 10: Docker Compose Lab And End-To-End Validation

**Files:**
- Modify: `docker-compose.yml`
- Create: `labs/demo1/README.md`
- Create: `labs/demo1/nginx/default.conf`
- Create: `labs/demo1/nginx/index.html`
- Create: `docs/phases/2026-07-03-demo1-foundation/SPEC.md`
- Create: `docs/phases/2026-07-03-demo1-foundation/PLAN.md`
- Create: `docs/phases/2026-07-03-demo1-foundation/UAT.md`
- Create: `docs/phases/2026-07-03-demo1-foundation/EVAL.md`
- Create: `docs/phases/2026-07-03-demo1-foundation/REVIEW.md`

**Interfaces:**
- Produces: local lab startup with Postgres, backend, frontend, DMZ web, and optional Ollama connection.
- Produces: documented UAT and evaluation evidence.

- [ ] **Step 1: Extend Docker Compose**

Add services:

- `backend`
- `frontend`
- `dmz-web`
- `postgres`

Keep Ollama external by default so users can run it on host or GPU server.

- [ ] **Step 2: Add lab docs**

Document:

- prerequisites
- startup
- running a scenario
- opening dashboard
- expected outputs
- known 4 GB VRAM constraints

- [ ] **Step 3: Add phase docs**

Create:

- phase `SPEC.md` summarizing Demo 1 scope
- phase `PLAN.md` linking this plan
- phase `UAT.md` with operator acceptance checks
- phase `EVAL.md` with scenario evaluation matrix
- phase `REVIEW.md` initialized with review checklist

- [ ] **Step 4: Run backend tests**

Run: `pytest backend/tests -v`

Expected: PASS.

- [ ] **Step 5: Run frontend tests**

Run: `npm --prefix frontend test`

Expected: PASS.

- [ ] **Step 6: Run lab smoke test**

Run: `docker compose up --build`

Expected:

- backend health endpoint returns `{"status":"ok"}`
- frontend dashboard loads
- Postgres accepts connections
- `POST /demo/run/ssh_bruteforce` creates a case

- [ ] **Step 7: Commit**

```bash
git add docker-compose.yml labs docs/phases
git commit -m "chore: add demo1 lab and validation docs"
```

## Self-Review

### Spec Coverage

- Demo 1 runnable lab: Task 10.
- Canonical event schema: Task 2.
- PostgreSQL storage: Task 2.
- Scenario fixtures: Task 3.
- Ingestion: Task 3.
- Triggers and alerts: Task 4.
- Case manager and evidence: Task 4.
- Deterministic analysis: Task 5.
- Local LLM/Ollama adapter: Task 6.
- Compact case packet: Task 6.
- Comparison engine: Task 7.
- Recommend-only policy: Task 7.
- Audit records: Task 7.
- API workflow: Task 8.
- SOC dashboard: Task 9.
- UAT and evaluation docs: Task 10.

### Known Execution Risks

- Git is not currently valid in this workspace, so commit steps will fail until the repository is initialized or repaired.
- Package installation may require network access.
- Ollama model availability depends on local hardware and installed models.
- The frontend test setup may need Vitest configuration if defaults are insufficient.

### Execution Guidance

Start with Task 1 only. Do not build all subsystems in one pass. Each task should finish with tests, review, and a commit where Git is available.
