# Skill Card — Test Case Generator

---

## Skill Name
**testcase-generator** (`v1.1.0`)

Generate high-quality manual test cases from a spec/PRD/BRS/User Story with strict validation, risk scanning, and traceability.

---

## What this Skill Automates

Given `spec_content`, the skill will:

1. Mask sensitive data (optional, default ON).
2. Validate input/spec quality before generation.
3. Parse feature, flows, business rules, validations, and boundaries.
4. Run an **Intelligence Scan Report** (what it understood + risk areas + gaps).
5. Generate test cases by coverage level (`basic` / `standard` / `full`).
6. Include security scenarios (default ON).
7. Produce output in `markdown` / `json` / `csv`.
8. Build a traceability matrix (except when skipped by low coverage mode).

---

## Inputs

| Parameter | Required | Default | Notes |
|---|---|---|---|
| `spec_content` | Yes | — | Full spec text, minimum quality required |
| `feature_name` | No | Auto-extract | Use when spec has multiple names |
| `output_format` | No | `markdown` | `markdown` / `json` / `csv` |
| `coverage_level` | No | `standard` | `basic` / `standard` / `full` |
| `enable_security` | No | `true` | Include OWASP/auth/permission abuse cases |
| `mask_pii` | No | `true` | Redact sensitive data before processing |

---

## Quality Gates & Safety Rules

- Hard-stop on invalid input:
  - `ERR-001` empty spec
  - `ERR-002` invalid encoding/characters
  - `ERR-003` spec too short (< 50 chars)
  - `ERR-004` invalid output format
  - `ERR-005` invalid coverage level
- Hard-stop on weak spec with `WARN-001` if missing:
  - at least one user flow
  - at least one business rule/validation
- Non-negotiable output behavior:
  - No vague steps (must be explicit actions)
  - No unverifiable expected results
  - Must provide concrete test data
  - Must self-check anti-patterns before final output

---

## BEFORE vs AFTER

| Aspect | Manual (Before) | With Skill (After) |
|---|---|---|
| Spec analysis | QC reads and interprets manually | Auto-parsed with rule extraction + risk flags |
| Coverage design | Depends on individual experience | Structured by checklist + coverage level |
| Security cases | Often skipped or delayed | Included by default |
| Traceability | Manual linking | Auto-generated matrix |
| Typical effort | 2–4h (or more for complex features) | ~15–30m including review |

---

## Outputs

- Test case suite (Happy path, Negative, Edge, Security)
- Intelligence Scan Report
- Spec gap warnings and confidence assessment
- Traceability matrix (rule ↔ test case mapping)
- PII masking report (when enabled)

---

## Tooling

| Tool | Role |
|---|---|
| Claude / Claude Code | Execute skill workflow and generation |
| Markdown / JSON / CSV | Output formats |
| Git / GitHub | Versioning and PR review |

---

## Current Limitations

- If spec quality is too low, skill stops (does not guess).
- Focuses on **test case generation**, not test execution.
- Does not directly publish to Jira/TestRail by default.
- Multi-feature specs may need splitting for cleaner outputs.

---

## Roadmap

| Priority | Planned Improvement |
|---|---|
| High | Better ambiguity detection + actionable spec fixes |
| High | Native export/integration to Jira/TestRail APIs |
| Medium | Generate automation skeletons from TCs (Playwright/Pytest) |
| Medium | Batch generation for multiple features |
| Low | Multi-language tuning and localized templates |
