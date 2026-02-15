# Design: Clinical Summary & Documentation Assistant (CSDA)

## 1. Overview

This document describes the architecture and design of the **Clinical Summary & Documentation Assistant**, an AI solution that supports healthcare workflows through summarization, key-information extraction, and draft documentation—using **synthetic and publicly available data only**, with clear limitations and human-in-the-loop usage.

---

## 2. Design Principles

| Principle | Application |
|-----------|-------------|
| **Responsibility** | No PHI; disclaimers on every output; no diagnostic/treatment decisions. |
| **Accuracy** | Structured prompts, optional citations, and confidence cues; human verification required. |
| **Meaningful support** | Focus on time-saving and clarity (summaries, drafts, education) without replacing judgment. |
| **Transparency** | Document model/prompt versions, data sources, and limitations in repo and UI. |

---

## 3. System Context

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         EXTERNAL ACTORS                                   │
│  Clinicians / Nurses / Researchers / Care Coordinators (optional)        │
└─────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                         CSDA APPLICATION LAYER                           │
│  • Web UI or API (REST)                                                  │
│  • Auth & access control                                                 │
│  • Request validation & rate limiting                                    │
└─────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                         ORCHESTRATION / SERVICE LAYER                    │
│  • Task routing (summarize / extract / draft / explain)                  │
│  • Prompt construction & template selection                              │
│  • Response formatting & disclaimer injection                            │
│  • Audit logging (model version, prompt version, timestamp)              │
└─────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                         AI / MODEL LAYER                                 │
│  • LLM API (e.g., OpenAI, Azure OpenAI, or open model)                   │
│  • Optional: embedding model for retrieval (public/synthetic corpus)     │
└─────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                         DATA & CONFIG                                     │
│  • Synthetic / public datasets (training or few-shot only)                │
│  • Prompts & templates (versioned)                                       │
│  • Config: feature flags, guardrails, disclaimer text                     │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Component Design

### 4.1 Application Layer

- **Web UI (optional):** Simple interface to paste text, choose task (summarize / extract / draft / explain), and view output with disclaimer always visible.
- **REST API:** Endpoints such as:
  - `POST /summarize` — body: `{ "text", "options": { "length", "sections" } }`
  - `POST /extract` — body: `{ "text" }` → returns structured key findings, medications, etc.
  - `POST /draft` — body: `{ "text", "template": "assessment_plan" | "follow_up" }`
  - `POST /explain` — body: `{ "concept_or_instruction", "audience": "patient" }` (synthetic/public source only)
- **Auth:** API key or OAuth; role-based access (e.g., disable patient-facing explain in strict deployments).
- **Validation:** Max input length, blocklist for obvious PHI patterns (e.g., SSN), and synthetic-only policy enforcement in demo mode.

### 4.2 Orchestration / Service Layer

- **Task router:** Maps request type to the appropriate prompt template and post-processing.
- **Prompt manager:** Versioned prompts per task; includes system instructions that enforce “assistive only,” “no diagnosis,” and “verify all content.”
- **Disclaimer injector:** Appends a standard disclaimer to every response (configurable text; see requirements).
- **Audit logger:** Writes minimal, non-PHI metadata: timestamp, task type, model id, prompt version, response length; optional export for audit.

### 4.3 AI / Model Layer

- **LLM:** Single or multiple providers behind a thin abstraction (e.g., `LLMGateway`) so that model and API can be swapped.
- **No PHI to model:** Only synthetic or de-identified/public content is sent; input validation and policy enforcement in the application/orchestration layer.
- **Optional retrieval:** If needed for “explain” or RAG, use only public/synthetic corpora (e.g., CDC, NIH, or synthetic Q&A pairs); no real patient data.

### 4.4 Data & Config

- **Prompts and templates:** Stored as versioned files or in config store; referenced in audit log.
- **Synthetic / public data:** Used for development, testing, and optional few-shot examples; documented in README and design.
- **Feature flags:** e.g., enable/disable patient-facing explain, max input length, disclaimer text.

---

## 5. Data Flow (Example: Summarization)

1. User submits text (and options) via UI or API.
2. Application layer validates input (length, no PHI patterns in demo), checks auth and rate limits.
3. Orchestration selects “summarize” prompt template, injects user text, calls LLM.
4. LLM returns raw summary.
5. Orchestration formats output (e.g., markdown), appends disclaimer, logs audit record.
6. Response returned to user; no persistent storage of content unless configured for synthetic-only retention.

---

## 6. Security & Privacy

- **Transport:** TLS for all client–server and server–LLM communication.
- **No PHI:** Design assumes no real PHI in any environment for this solution; any future use with real data would require a separate design and compliance review.
- **Logging:** Logs contain no identifiable patient data; only metadata and optionally hashed or synthetic content.
- **Access control:** Roles (e.g., admin, clinician) control which features are available per deployment.

---

## 7. Limitations & Risks (Design Mitigations)

| Limitation / Risk | Mitigation in design |
|-------------------|----------------------|
| Model errors or hallucinations | Strong prompts, disclaimer, and “human must verify” in UI and docs. |
| Bias from training data | Use of public/synthetic data only; document scope; periodic review. |
| Misuse (e.g., as diagnostic tool) | Explicit “assistive only” in prompts and UI; no treatment/diagnosis outputs. |
| Scope creep to PHI | Strict validation and policy; no PHI in default data flow or storage. |

---

## 8. Technology Choices (Recommendations)

| Area | Recommendation | Rationale |
|------|----------------|-----------|
| Backend | Python (FastAPI) or Node.js | Fast iteration, good LLM SDK support. |
| LLM | API-based (e.g., Azure OpenAI, OpenAI) or open model (e.g., Llama, Mistral) | Flexibility; no PHI sent; comply with provider ToS. |
| Frontend | Simple React or Vue SPA, or server-rendered | Readable output and clear disclaimer placement. |
| Storage | Optional SQLite/Postgres for audit metadata only | No document content storage by default. |
| Deployment | Container (Docker); optional Kubernetes | Reproducibility and portability. |

---

## 9. Consistency with requirements.md

- **FR-1–FR-6:** Implemented via task router, prompt manager, and audit logger.
- **FR-7–FR-9:** Handled by API schema and template options.
- **FR-10–FR-13:** Enforced by validation, disclaimer injector, and configurable guardrails.
- **NFR-1–NFR-5:** Addressed by async/streaming where needed, TLS, optional RBAC, and logging.
- **Data policy and limitations:** Reflected in this design (no PHI, synthetic/public only, disclaimers, and documented limitations).

---

## 10. Future Considerations (Out of Scope for Initial Deliverable)

- Integration with EHR (read-only, with appropriate data agreements).
- Multi-language and specialty-specific prompt sets.
- Formal evaluation framework on synthetic benchmarks (e.g., ROUGE, factual consistency).
- Regulatory path (e.g., if ever pursued as a medical device).

---

*Document version: 1.0 | Aligned with requirements.md | Synthetic and publicly available data only.*
