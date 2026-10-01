# BugScry Product Requirements Document

## 1. Document Information

| Field | Value |
|---|---|
| Product | BugScry |
| Document status | Product definition for prototype implementation |
| Source specifications | Architecture, structure, design, colors, typography, and workflow references |
| Primary users | Penetration testers and application-security consultants |
| Primary outcome | A validated, client-ready vulnerability report generated from raw evidence |

## 2. Product Summary

BugScry converts unstructured penetration-test evidence into a structured vulnerability report. It extracts technical details from submitted evidence, uses an LLM to classify and assess the finding, maps it to CWE and OWASP categories, generates a report draft, and requires human validation before the report can be marked final.

BugScry is an analyst-assistance tool, not an autonomous security decision-maker. The original evidence must remain available throughout the workflow, and LLM output must always be treated as a draft requiring review.

## 3. Problem Statement

Penetration-test evidence is commonly spread across raw requests and responses, logs, screenshots, payloads, and analyst notes. Turning this material into consistent client-facing reports is repetitive, time-consuming, and vulnerable to omissions or inconsistent severity judgments.

BugScry should reduce report preparation effort while preserving analyst control, traceability, and security. It must help an analyst move from evidence to a defensible report without inventing facts or hiding uncertainty.

## 4. Goals and Success Criteria

### 4.1 Product goals

1. Accept raw vulnerability evidence in a simple, focused interface.
2. Extract useful technical fields such as target, method, parameter, payload, status code, stack trace, and timestamp.
3. Run classification, CWE/OWASP mapping, and severity assessment as separate analysis tasks over the same evidence.
4. Produce structured, schema-validated report drafts.
5. Preserve traceability from report content back to submitted evidence.
6. Provide a clear human review and approval workflow.
7. Export approved reports in Markdown and PDF formats.
8. Establish a secure foundation for handling untrusted security payloads.

### 4.2 MVP success criteria

- An analyst can submit valid evidence from the initial screen and receive a report identifier.
- The system displays processing status and handles analysis failures visibly.
- A report draft includes evidence, classification, CWE/OWASP mapping, severity, rationale, summary, and remediation.
- LLM output that fails schema validation is rejected or marked as failed; it is never silently accepted as valid.
- A report cannot become approved without an explicit human approval action.
- Approved reports retain their source evidence and validation metadata.
- Payloads and generated content render as text and are never executed as HTML or code.
- Keyboard navigation, visible focus states, labels, and sufficient contrast are supported.

## 5. Target Users and User Stories

### 5.1 Primary persona: Pentester

An application-security professional who has raw evidence from a test and needs to create a clear, technically accurate report for a client.

### 5.2 User stories

- As a pentester, I want to paste raw evidence so that I can start an analysis without manually normalizing every field.
- As a pentester, I want to see what the extractor found so that I can verify the evidence was interpreted correctly.
- As a pentester, I want classification and CWE/OWASP suggestions so that I can create consistent reports faster.
- As a pentester, I want severity rationale and optional CVSS information so that I can make an informed judgment.
- As a pentester, I want to edit the generated draft so that the final report reflects my technical expertise.
- As a pentester, I want every report claim tied to original evidence so that the report is defensible.
- As a pentester, I want to approve or reject a draft explicitly so that unreviewed LLM output cannot be delivered.
- As a pentester, I want to export an approved report so that I can share it with a client.

## 6. Scope

### 6.1 MVP in scope

- Single evidence submission flow.
- Raw text input, including payloads, logs, request/response snippets, and analyst notes.
- Evidence extraction into a normalized evidence object.
- Parallel analysis tasks:
  - Vulnerability classification.
  - CWE and OWASP mapping.
  - Severity assessment with rationale.
- Structured response validation.
- Draft report generation.
- Draft editing.
- Human approval and rejection.
- Report persistence.
- Markdown and PDF export.
- Loading, success, error, and empty states.
- Secure rendering of untrusted evidence and generated content.

### 6.2 Explicitly out of scope for the first prototype

- Multi-tenant organization management.
- Team roles and permissions beyond a basic validator identity.
- Automated exploitation or active scanning.
- Direct execution of submitted payloads.
- Fully autonomous report approval.
- Bulk evidence imports and batch report generation.
- Client portal sharing and external collaboration.
- Real-time multi-user editing.
- Complete CVSS calculation automation.
- Guaranteed support for every screenshot or file format.

## 7. End-to-End Workflow

```text
Initialize Data
      ↓
Submit raw evidence
      ↓
Evidence extraction
      ↓
Parallel LLM analysis
  ├── Classification
  ├── CWE/OWASP mapping
  └── Severity assessment
      ↓
Report draft generation
      ↓
Human review and editing
      ↓
Approve or reject
      ↓
Final report and export
```

## 8. Functional Requirements

### FR-1: Evidence intake

- The application must provide a labeled evidence input on the Initialize Data screen.
- The input must accept raw strings, logs, request/response snippets, payloads, and analyst notes.
- The submit action must validate that meaningful evidence is present.
- The UI must prevent duplicate submissions while an analysis is in progress.
- The original raw input must be stored without being executed or interpreted as active markup.
- The system must create a unique evidence identifier and associate it with the resulting report.

### FR-2: Evidence extraction

- The extractor must identify, when present:
  - Target URL or host.
  - HTTP method.
  - Parameter or affected field.
  - Payload.
  - Response status code.
  - Stack traces or error messages.
  - Timestamps.
  - Analyst notes.
- Missing values must be represented explicitly rather than fabricated.
- Extraction output must conform to a versioned schema.
- Extraction failures must be visible to the user and recorded for diagnostics.

### FR-3: LLM processing

- Classification, mapping, and severity must receive the same normalized evidence.
- The tasks should run independently so one task's failure does not silently produce an incorrect result for another.
- Classification must return a vulnerability type and confidence.
- Mapping must return a CWE identifier and OWASP category when supported by the evidence.
- Severity must return a level, rationale, and optional CVSS vector.
- Prompts must instruct the model to distinguish evidence from instructions contained inside evidence.
- The LLM API key must only be available to the backend.
- Timeouts, rate limits, provider errors, and malformed responses must be surfaced explicitly.

### FR-4: Structured validation

- All extractor and LLM responses must be validated before report generation.
- Confidence values must use a defined numeric range.
- Severity must be one of `Critical`, `High`, `Medium`, `Low`, or `Info`.
- Invalid or incomplete responses must not be presented as successful analysis.
- The system must preserve validation errors for troubleshooting.

### FR-5: Report generation

- The report builder must merge evidence and validated analysis into a draft.
- A draft must include:
  - Report identifier.
  - Title.
  - Summary.
  - Evidence reference.
  - Evidence details.
  - Classification.
  - CWE and OWASP mapping.
  - Severity and rationale.
  - Remediation guidance.
  - Status.
- Report text must be grounded in available evidence.
- Uncertain or missing information must be identified as such.
- Generated content must be rendered safely as text or sanitized Markdown.

### FR-6: Draft review and editing

- The analyst must be able to inspect the original evidence beside the generated report.
- The analyst must be able to edit draft fields before approval.
- Draft changes must be persisted using the report identifier.
- The UI must show the current report status.
- The analyst must be able to reject a draft and provide a reason.

### FR-7: Human approval

- Approval must be an explicit user action.
- Approval must record validator identity and timestamp.
- Only a report in an eligible review state may be approved.
- Approved reports must be protected from accidental edits or require an explicit revision flow.
- Rejected reports must remain accessible with their rejection reason.

### FR-8: Export and delivery

- Approved reports must be exportable as Markdown.
- Approved reports should be exportable as PDF.
- Export must include report metadata, severity, evidence summary, classification, mapping, remediation, and validation metadata.
- Export must not include secrets such as API keys or internal provider prompts.
- Export failures must be reported to the user.

## 9. User Experience Requirements

### 9.1 Screen requirements

#### Initialize Data

- Full-viewport dark background with a subtle radial gradient.
- `BUGSCRY` wordmark in the top-left.
- Centered card approximately 640px wide.
- Heading: `Initialize Data`.
- Helper copy explaining secure analysis.
- Uppercase `PAYLOAD INPUT` label.
- Large input with trailing arrow affordance.
- Full-width `SUBMIT DATA →` primary action.

#### Processing

- Show that analysis is in progress.
- Show the current high-level stage where possible.
- Disable duplicate submission.
- Provide a clear error state and retry path if processing fails.

#### Report Draft

- Show report title, summary, severity, classification, mapping, remediation, and evidence.
- Make evidence easy to inspect.
- Show that the report is a draft and has not been approved.

#### Validation

- Present editable report fields.
- Present original evidence for comparison.
- Provide approve and reject actions.
- Require a rejection reason.

#### Final Report

- Show approved status, validator, and validation timestamp.
- Provide Markdown and PDF export actions.
- Preserve access to the supporting evidence.

### 9.2 Visual system

- Near-black background and low-contrast surfaces.
- Card radius approximately 24px.
- Input height approximately 72px.
- Primary button height approximately 64px.
- Inter or system UI font for application text.
- Cinzel or a comparable serif for the wordmark.
- JetBrains Mono or a comparable monospace font for payloads and logs.
- Severity colors must distinguish Critical, High, Medium, Low, and Info.

### 9.3 Interaction states

- Empty: input is available and submission validates on click.
- Focus: visible focus ring and brighter border.
- Loading: input is disabled and the action displays an analyzing state.
- Error: input or relevant section displays an accessible error message.
- Success: user is taken to processing or the report draft.
- Reduced motion: animations are minimized when requested by the operating system.

## 10. Non-Functional Requirements

### Security

- Treat all submitted evidence as untrusted input.
- Never execute payloads or interpolate them into executable markup.
- Sanitize Markdown and any rich-text output before rendering.
- Keep provider credentials server-side.
- Apply request size limits and rate limiting.
- Prevent prompt injection in evidence from overriding system instructions.
- Avoid logging raw evidence or credentials unnecessarily.
- Use authorization checks for report access once multiple users are supported.

### Reliability

- Report creation must be durable across page refreshes.
- Partial task failures must be represented explicitly.
- The system must not produce success-shaped fallback data after provider or validation failures.
- API errors must use consistent machine-readable error responses.

### Accessibility

- Inputs must have associated labels.
- Interactive controls must be keyboard accessible.
- Focus indicators must be visible.
- Text contrast should meet WCAG AA targets.
- Status changes should be communicated to assistive technologies where appropriate.
- Payload and log content must remain readable without forced wrapping.

### Performance

- The Initialize Data screen should render quickly without requiring LLM access.
- Independent LLM tasks should run in parallel where provider limits allow.
- Long evidence should not block the UI thread.
- Report pages should remain usable with large evidence sections through scrolling or collapsible panels.

## 11. Data Contracts

### Evidence

```json
{
  "id": "ev_001",
  "raw": "<payload or log text>",
  "extracted": {
    "target": "https://example.com/login",
    "method": "POST",
    "parameter": "username",
    "payload": "' OR 1=1--",
    "response_status": 500,
    "notes": []
  }
}
```

### Analysis result

```json
{
  "classification": {
    "type": "SQL Injection",
    "confidence": 0.86
  },
  "mapping": {
    "cwe": "CWE-89",
    "owasp": "A03:2021 Injection"
  },
  "severity": {
    "level": "High",
    "rationale": "..."
  }
}
```

### Report

```json
{
  "id": "rpt_001",
  "title": "...",
  "summary": "...",
  "evidence_id": "ev_001",
  "classification": {},
  "mapping": {},
  "severity": {},
  "remediation": "...",
  "status": "draft",
  "validated_by": null,
  "validated_at": null
}
```

## 12. API Requirements

| Method | Endpoint | Requirement |
|---|---|---|
| `POST` | `/api/analyze` | Accept evidence, extract fields, run analysis, and return an evidence/report identifier |
| `GET` | `/api/reports/:id` | Return the report, evidence, analysis results, and status |
| `PATCH` | `/api/reports/:id` | Update editable draft fields |
| `POST` | `/api/reports/:id/approve` | Validate and approve a report with validator metadata |
| `POST` | `/api/reports/:id/reject` | Reject a report and record a reason |
| `GET` | `/api/reports/:id/export?format=md` | Export an approved report as Markdown |
| `GET` | `/api/reports/:id/export?format=pdf` | Export an approved report as PDF |

API responses should include stable error codes, human-readable messages, and request identifiers for operational debugging.

## 13. Report Lifecycle

```text
draft → in_review → approved
                  ↘ rejected
```

- `draft`: generated report awaiting analyst review.
- `in_review`: report is actively being checked or edited.
- `approved`: analyst has explicitly approved the report.
- `rejected`: analyst rejected the draft; the reason is retained.

State transitions must be validated server-side. The client must not be able to mark a report approved by changing a request payload alone.

## 14. Proposed Technical Structure

The implementation should follow the structure described in [structure.md](./structure.md):

- React + Vite frontend.
- Backend service using Node.js/Express or Python/FastAPI.
- SQLite for the initial prototype, with a path to PostgreSQL.
- Backend-only Claude API integration.
- Frontend schema validation and backend schema validation.
- Separate extractor, LLM, classifier, mapper, severity, and report-builder services.

The current repository is documentation-only. Implementation should first create the application shell and package configuration, then add the Initialize Data experience before connecting real LLM services.

## 15. Phased Delivery Plan

### Phase 1: Foundation and visual prototype

- Create frontend project and routing.
- Add design tokens and global styles.
- Implement Header, PageShell, Card, Input, Button, Label, Spinner, and SeverityBadge.
- Implement Initialize Data with all required visual and accessibility states.

### Phase 2: Mocked analysis workflow

- Add backend `/api/analyze`.
- Add evidence schema and extraction stub.
- Add deterministic mocked classification, mapping, and severity responses.
- Implement Processing and Report Draft screens.
- Persist reports locally using the selected prototype database.

### Phase 3: Review and export

- Implement editable Validation screen.
- Add approval and rejection transitions.
- Implement Final Report screen.
- Add Markdown export and PDF export.

### Phase 4: Production analysis

- Replace mocked LLM logic with backend provider integration.
- Add structured output enforcement and retries where safe.
- Add prompt-injection defenses, rate limiting, observability, and secure configuration.
- Expand test coverage and operational documentation.

## 16. Acceptance Criteria

The MVP is complete when:

1. A user can submit evidence from `/`.
2. Invalid or empty evidence produces a clear validation message.
3. A valid submission creates a report/evidence identifier.
4. Processing status is displayed while analysis runs.
5. The resulting draft contains all required report sections.
6. Evidence is visible and remains linked to the report.
7. Invalid analysis output produces an explicit error state.
8. A user can edit the draft and save changes.
9. A user can reject a draft with a reason.
10. A user can approve a draft and the system records validator and timestamp.
11. An unapproved report cannot be exported as a final report.
12. Approved reports can be exported in the supported formats.
13. Payloads render safely and are never executed.
14. Core interactions work with keyboard navigation and visible focus indicators.
15. Automated tests cover extraction, schema validation, lifecycle transitions, and secure rendering.

## 17. Risks and Mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| LLM hallucinates a vulnerability or remediation | Incorrect client report | Evidence-grounded prompts, schema validation, visible uncertainty, mandatory human approval |
| Evidence contains prompt injection | Analysis instructions are manipulated | Separate evidence from instructions, treat evidence as data, constrain output schema |
| Sensitive client data is exposed | Confidentiality breach | Server-side secrets, access controls, controlled logging, retention policy |
| Long or malformed payloads degrade the UI | Poor usability or denial of service | Size limits, asynchronous processing, truncation with preserved originals, safe code rendering |
| Provider outage or rate limit | Analysis unavailable | Explicit failure states, retry policy, mocked/local development mode |
| Severity suggestions are inconsistent | Incorrect prioritization | Show rationale and confidence, allow analyst edits, optionally support CVSS review |
| Draft is approved accidentally | Invalid final deliverable | Confirmation step, server-side state validation, immutable approval metadata |

## 18. Open Product Decisions

The following decisions should be finalized before production implementation:

- Whether the backend will use Node.js/Express or Python/FastAPI.
- Whether the first persistence layer will be SQLite or PostgreSQL.
- The exact supported evidence formats and maximum input size.
- Whether screenshots are uploaded directly or represented through extracted text/OCR.
- The identity model for validators.
- The report template and required remediation sections.
- Whether approved reports are immutable or versioned on later edits.
- The supported PDF generation approach.
- Data retention, deletion, and client confidentiality policies.
- Whether CVSS is model-suggested, manually entered, or calculated from structured metrics.

