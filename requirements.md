# Requirements: Clinical Summary & Documentation Assistant

## 1. Product Overview

**Working name:** Clinical Summary & Documentation Assistant (CSDA)

**Purpose:** An AI-powered assistant that improves efficiency and understanding in healthcare by summarizing clinical and research information, drafting structured documentation, and supporting care navigation—using **synthetic and publicly available data only**, with explicit limitations and human-in-the-loop workflows.

**Target users:** Healthcare professionals (clinicians, nurses, care coordinators), researchers, and—in limited, non-diagnostic roles—patient education and care navigation support.

---

## 2. Goals & Success Criteria

| Goal | Success Criterion |
|------|-------------------|
| Improve efficiency | Reduce time to produce structured summaries from long documents (measured on synthetic benchmarks). |
| Improve understanding | Summaries rated as accurate and useful in user studies on synthetic cases. |
| Meaningful support | Outputs align with stated use cases (summarization, drafting, education) and do not replace clinical judgment. |
| Responsibility | All data usage documented; no PHI; limitations and disclaimers clear and surfaced in UI and API. |

---

## 3. Functional Requirements

### 3.1 Core Capabilities

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-1 | **Summarization:** Generate concise, structured summaries from long clinical or research text (e.g., discharge summaries, progress notes, research abstracts). | Must |
| FR-2 | **Key information extraction:** Extract and list key findings, diagnoses, medications, allergies, and action items from input text. | Must |
| FR-3 | **Draft documentation:** Produce draft sections (e.g., assessment & plan, follow-up instructions) that a human must review and approve. | Must |
| FR-4 | **Patient-friendly explanations:** Generate simple-language explanations of medical concepts or care instructions from synthetic or public health content only. | Should |
| FR-5 | **Research abstract summarization:** Summarize publicly available research abstracts with key results and limitations. | Should |
| FR-6 | **Audit trail:** Log which model and prompt version produced each output; support optional export for audit. | Must |

### 3.2 Inputs & Outputs

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-7 | Accept plain text and optionally structured text (e.g., sections) as input. | Must |
| FR-8 | Output in both human-readable form (e.g., markdown/HTML) and optional structured format (e.g., JSON) for downstream systems. | Should |
| FR-9 | Support configurable output length and section templates (e.g., “brief” vs “detailed” summary). | Should |

### 3.3 Safety & Compliance

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-10 | **No PHI:** System MUST NOT require or store real patient-identifiable data; only synthetic or de-identified/public data. | Must |
| FR-11 | **Limitations notice:** Every response MUST be accompanied by a short, visible disclaimer (e.g., “For support only; not a substitute for professional judgment; verify all clinical content”). | Must |
| FR-12 | **No diagnostic or treatment decisions:** Outputs are explicitly “assistive” only; no recommendations that constitute diagnosis or treatment decisions. | Must |
| FR-13 | **Configurable guardrails:** Ability to disable certain features (e.g., patient-facing text) per deployment. | Should |

---

## 4. Non-Functional Requirements

| ID | Requirement | Priority |
|----|-------------|----------|
| NFR-1 | **Latency:** Summary generation within acceptable time (e.g., &lt; 30 s for typical document length) for interactive use. | Must |
| NFR-2 | **Availability:** Target 99% uptime for API/service during evaluation phase. | Should |
| NFR-3 | **Security:** All data in transit encrypted (TLS); no persistent storage of user input by default unless explicitly configured with synthetic-only policy. | Must |
| NFR-4 | **Explainability:** Where feasible, indicate source sections or confidence cues (e.g., “low confidence” for ambiguous input). | Should |
| NFR-5 | **Access control:** Support role-based access (e.g., admin vs. clinician) for deployment. | Should |

---

## 5. Data & Limitations (Mandatory)

### 5.1 Data Policy

- **Training / fine-tuning:** Only synthetic or publicly available, non-PHI datasets (e.g., MIMIC-III with approved access and de-identification, or fully synthetic clinical text).
- **Runtime:** Only synthetic or publicly available inputs for demos and benchmarks; no real PHI in any environment unless under a separate, compliant data agreement not in scope for this solution.
- **Storage:** No retention of identifiable patient data; logs may retain hashed or synthetic content only, with clear documentation.

### 5.2 Stated Limitations

The system MUST document and surface the following:

1. **Not a medical device:** Not intended for diagnosis, treatment, or replacement of clinical judgment.
2. **Possible errors:** Summaries and extractions may contain inaccuracies or omissions; all output must be verified by a qualified professional.
3. **Synthetic/public data focus:** Models and demos are built and validated on synthetic or public data; performance on real-world clinical text may differ.
4. **Language and domain:** Supported languages and clinical domains (e.g., English, general internal medicine) are limited and documented.
5. **Bias and fairness:** Models may reflect biases present in training data; regular review and mitigation are required.

---

## 6. Out of Scope (Explicit)

- Real PHI processing or storage.
- Direct diagnosis or treatment recommendation.
- Replacement of EHR or clinical documentation systems.
- Regulatory submission (e.g., FDA) as a medical device.
- Real-time patient monitoring or alerting.

---

## 7. Dependencies & Assumptions

- Access to LLM APIs or self-hosted models (e.g., for summarization) under acceptable use and data policies.
- Synthetic or public datasets available for development and testing (e.g., n2c2, MIMIC-III with approval, or generated synthetic notes).
- Deployment in a controlled environment (e.g., internal or research) with informed users who acknowledge limitations.
- Human reviewers in the loop for any use of outputs in real workflows.

---

## 8. Acceptance Criteria (Summary)

- [ ] Summarization and key-information extraction work on synthetic/public sample documents.
- [ ] Limitations and disclaimers are shown in UI and documented in design.
- [ ] requirements.md and design.md are present in the repository and consistent with this document.
- [ ] No PHI is required or stored; data policy is clearly stated in repo (e.g., README or this file).

---

*Document version: 1.0 | For use with synthetic and publicly available data only.*
