# SPEC: Skill — Test Case Generator + Report Template

> **Version**: 1.1.0
> **Last updated**: 2026-02-24
> **Author**: QA Team
> **Status**: QC-Ready
> **Changelog**: v1.1.0 — Added Security, Prompt Logic System, Error Handling, Data Schema (JSON/CSV), Traceability, PII Masking per QC review feedback

---

## Table of Contents

1. [Overview](#1-overview)
2. [Scope](#2-scope)
3. [Rules](#3-rules)
4. [User Flow](#4-user-flow)
5. [Business Logic](#5-business-logic)
6. [Edge Cases](#6-edge-cases)
7. [Test Data](#7-test-data)
8. [Output Format](#8-output-format)
9. [Prompt Logic System](#9-prompt-logic-system)
10. [Error Handling](#10-error-handling)
11. [Data Schema — JSON/CSV Contract](#11-data-schema--jsoncsv-contract)
12. [Traceability — Requirement Mapping](#12-traceability--requirement-mapping)
13. [Security & PII Masking](#13-security--pii-masking)

---

## 1. Overview

### 1.1 Skill Name
**Test Case Generator + Report Template**

### 1.2 Actor
QC / QA Manager

### 1.3 Problem Statement
After generating test cases, QC teams still need to manually write test reports — a redundant, time-consuming step that introduces inconsistency across projects.

### 1.4 What This Skill Does
A single-input, two-output pipeline:

```
[Input: SPEC.md file]
        │
        ▼
┌─────────────────────────────────┐
│   Test Case Generator + Report  │
│           Template Skill        │
└─────────────────────────────────┘
        │
        ├──► Output A: Structured Test Cases (with IDs, steps, expected results)
        │
        └──► Output B: Report Template (QC fills Pass/Fail/Blocked)
                              │
                              ▼
                  [QC submits filled report]
                              │
                              ▼
                  Output C: Aggregated Test Summary Report
```

### 1.5 Business Value
- Full pipeline: Spec → Test Cases → Report in one skill invocation
- Eliminates manual test case writing and report formatting
- Standardizes QC output across all projects
- Reduces manual effort by ~70% on test documentation

### 1.6 MVP Scope
Input **one SPEC.md file** → Output **test cases + report template** ready to use.

---

## 2. Scope

### 2.1 In Scope

| # | Feature | Description |
|---|---------|-------------|
| 1 | Spec Parsing | Read and understand a `.md` spec file provided by the user |
| 2 | Test Case Generation | Auto-generate test cases covering Happy Path, Negative, Edge, and Boundary scenarios |
| 3 | Test Case ID Assignment | Assign unique, traceable test case IDs (format: `TC-XXX-NNN`) |
| 4 | Report Template Generation | Produce a fillable report template with Pass/Fail/Blocked/Skip columns |
| 5 | Result Aggregation | Aggregate QC-filled results into a final test summary report |
| 6 | Coverage Tagging | Tag each test case with scenario type (Happy Path / Negative / Edge / Boundary) |
| 7 | Priority Labeling | Mark each test case as Critical / High / Medium / Low |
| 8 | Security Test Cases | Auto-generate security-specific TCs: auth bypass, injection, broken access control, PII exposure |
| 9 | Traceability Mapping | Map each TC back to its source requirement ID for coverage tracking |
| 10 | JSON/CSV Export | Export generated TCs in JSON and CSV format for automation script consumption |

### 2.2 Out of Scope (MVP)

| # | Feature | Reason |
|---|---------|--------|
| 1 | Automated test execution | Manual QC review is required by design |
| 2 | Multi-file spec input | MVP supports single spec file only |
| 3 | Integration with JIRA/TestRail | Post-MVP roadmap item |
| 4 | AI-powered defect prediction | Out of MVP |
| 5 | Screenshot/video evidence upload | Post-MVP |

---

## 3. Rules

### 3.1 Input Rules

| Rule ID | Rule | Violation Behavior |
|---------|------|--------------------|
| R-IN-01 | Input MUST be a `.md` file | Skill returns error: `"Input must be a Markdown (.md) file"` |
| R-IN-02 | Spec file MUST contain at least one functional requirement | Skill returns warning: `"No requirements found. Please check spec content."` |
| R-IN-03 | Spec file size MUST NOT exceed 500KB | Skill returns error: `"File too large. Max size is 500KB."` |
| R-IN-04 | Spec file content MUST be UTF-8 encoded | Skill returns error: `"Unsupported encoding. Use UTF-8."` |
| R-IN-05 | Spec file MUST NOT be empty | Skill returns error: `"Empty file detected. Please provide spec content."` |

### 3.2 Test Case Generation Rules

| Rule ID | Rule |
|---------|------|
| R-TC-01 | Every functional requirement MUST produce at least 1 Happy Path test case |
| R-TC-02 | Every input field MUST produce at least 1 Negative test case |
| R-TC-03 | Every boundary value MUST have a Boundary test case (min, max, min-1, max+1) |
| R-TC-04 | Every business rule in the spec MUST map to at least 1 test case |
| R-TC-05 | Test case IDs MUST follow format: `TC-[FEATURE_CODE]-[3-digit sequence]` (e.g., `TC-AUTH-001`) |
| R-TC-06 | Each test case MUST include: ID, Title, Preconditions, Steps, Expected Result, Priority, Tags |
| R-TC-07 | Priority assignment MUST follow: Critical (auth/payment/data loss), High (core flows), Medium (secondary), Low (UI/cosmetic) |
| R-TC-08 | Duplicate test cases (same steps + expected result) MUST be deduplicated |

### 3.3 Report Template Rules

| Rule ID | Rule |
|---------|------|
| R-RP-01 | Report template MUST include all generated test case IDs pre-populated |
| R-RP-02 | Result column MUST only accept: `Pass`, `Fail`, `Blocked`, `Skip`, `N/A` |
| R-RP-03 | Report MUST include columns: TC ID, Title, Priority, Result, Actual Result (if Fail), Tester, Date |
| R-RP-04 | Summary section MUST auto-calculate: Total, Pass%, Fail%, Blocked%, Skip% |
| R-RP-05 | Report template MUST be in Markdown table format for easy editing |

### 3.4 Security Rules

| Rule ID | Rule |
|---------|------|
| R-SEC-01 | If the spec contains authentication, authorization, or payment flows, the skill MUST generate at least 1 Security TC per flow |
| R-SEC-02 | Security TCs MUST cover: SQL Injection, XSS, broken access control, session fixation, and brute force where applicable |
| R-SEC-03 | Security TCs MUST be tagged `security` and assigned `Critical` priority by default |
| R-SEC-04 | PII fields detected in the spec (email, phone, name, ID number, password) MUST be masked in all generated test data using placeholder format: `[PII:field_type]` |
| R-SEC-05 | The skill MUST NOT store, log, or echo back raw PII values from the spec in any output |
| R-SEC-06 | If a spec contains credentials or tokens in plaintext, skill MUST flag: `"Security Warning: Plaintext credential detected at [location] — remove before sharing"` |

### 3.5 Aggregation Rules

| Rule ID | Rule |
|---------|------|
| R-AG-01 | A test run is `PASSED` only if Pass% = 100% of non-skipped, non-N/A test cases |
| R-AG-02 | Any `Critical` test case with result `Fail` MUST trigger overall status = `BLOCKED — DO NOT RELEASE` |
| R-AG-03 | Any `High` test case with result `Fail` → overall status = `FAIL` |
| R-AG-04 | All `Fail` results MUST list the TC ID and Actual Result in the summary |
| R-AG-05 | `Skip` and `N/A` results are excluded from Pass% calculation |
| R-AG-06 | Any `Security` tagged TC with result `Fail` MUST trigger overall status = `BLOCKED — DO NOT RELEASE` regardless of other results |

---

## 4. User Flow

### 4.1 Primary Flow (Happy Path)

```
Step 1: QC opens Skill prompt
Step 2: QC uploads or pastes SPEC.md content
Step 3: Skill validates input (format, size, encoding, content)
Step 4: Skill parses spec → extracts features, rules, inputs, boundaries
Step 5: Skill generates test cases with IDs, steps, expected results
Step 6: Skill generates report template pre-filled with TC IDs
Step 7: QC downloads/copies test cases and report template
Step 8: QC executes tests manually and fills in Pass/Fail/Blocked/Skip
Step 9: QC submits filled report back to Skill
Step 10: Skill aggregates results → generates Test Summary Report
Step 11: QC downloads Test Summary Report
```

### 4.2 Alternate Flow — Spec Has No Requirements

```
Step 1-2: Same as primary flow
Step 3: Skill validates → finds no parseable requirements
Step 4: Skill returns: "Warning: No testable requirements detected.
         Please ensure your spec contains functional descriptions."
Step 5: Skill suggests spec template for user to fill
```

### 4.3 Alternate Flow — Invalid File Type

```
Step 1: QC uploads a .pdf or .docx file
Step 2: Skill detects non-.md format
Step 3: Skill returns: "Error: Only .md files are supported.
         Please convert your spec to Markdown format."
Step 4: Flow ends — no test cases generated
```

### 4.4 Alternate Flow — Re-run with Updated Spec

```
Step 1: QC re-uploads updated SPEC.md
Step 2: Skill detects new/changed requirements vs previous run (if session context available)
Step 3: Skill generates delta test cases for new/changed requirements
Step 4: Existing test case IDs are preserved; new TCs get next sequence number
Step 5: Merged test suite + new report template generated
```

---

## 5. Business Logic

### 5.1 Feature Code Extraction

The skill MUST derive a `FEATURE_CODE` from the spec:
- Use the first H1/H2 heading as the feature name
- Convert to uppercase alphanumeric, max 6 characters
- Example: `# User Authentication` → `AUTH`

### 5.2 Test Case Priority Matrix

| Scenario Type | Feature Area | Assigned Priority |
|---------------|-------------|-------------------|
| Login / Auth | Any | Critical |
| Payment / Checkout | Any | Critical |
| Data deletion / irreversible action | Any | Critical |
| Security (injection, access control, session) | Any | Critical |
| Core CRUD operations | Any | High |
| Search / Filter | Any | High |
| Form validation | Any | Medium |
| UI display / Responsive | Any | Low |
| Error messages wording | Any | Low |

### 5.3 Test Case Type Coverage Requirements

For each requirement, the skill MUST generate:

| Type | Trigger Condition | Minimum Count |
|------|------------------|---------------|
| Happy Path | Any functional requirement | 1 per requirement |
| Negative | Any input field or action | 1 per input field |
| Boundary | Numeric or length-limited fields | 4 per field (min, max, min-1, max+1) |
| Edge Case | Boolean states, empty states, concurrent actions | 1+ per identified edge |
| Security | Auth/payment/PII/input fields present in spec | 1+ per identified attack surface |

### 5.4 Edge Case Discovery Checklist (AI Instruction)

> This checklist instructs the AI on **how to identify edge cases** systematically from a spec. The AI MUST run through each trigger before finalizing the TC list.

**State-based triggers:**
- [ ] What happens when an entity is in an empty/null state? (empty cart, no results, zero balance)
- [ ] What happens at the very first use? (new user, first login, empty list)
- [ ] What happens when a limit is reached? (max items, max retries, max file size)
- [ ] What happens if a required dependency is missing? (no internet, no session, missing config)

**Concurrency triggers:**
- [ ] What if two users perform the same action simultaneously? (double booking, race condition)
- [ ] What if the user submits the same form twice quickly? (double submit)
- [ ] What if the session expires mid-flow? (timeout during checkout)

**Data integrity triggers:**
- [ ] What happens with special characters in input? (`<script>`, `' OR 1=1`, `../../../`)
- [ ] What happens with very long input? (1000-char name, 10MB file)
- [ ] What happens with Unicode/emoji in text fields?
- [ ] What happens if the input is all whitespace?

**Business rule conflict triggers:**
- [ ] Are there two rules that could contradict each other under certain conditions?
- [ ] What happens at the exact boundary of a business rule (e.g., exactly 5 failed attempts)?

**Security triggers:**
- [ ] Can a lower-privilege user access a higher-privilege resource by manipulating the URL/ID?
- [ ] Is there a field that renders output without escaping?
- [ ] Are tokens/sessions properly invalidated on logout?
- [ ] Can the same OTP/token be reused after expiry?

### 5.4 Aggregation Formula

```
Total TCs        = count of all test cases
Executed TCs     = Total - Skip - N/A
Pass%            = (Pass / Executed TCs) × 100
Fail%            = (Fail / Executed TCs) × 100
Blocked%         = (Blocked / Executed TCs) × 100

Overall Status:
  IF any Critical TC = Fail  → "BLOCKED — DO NOT RELEASE"
  ELSE IF any High TC = Fail → "FAIL"
  ELSE IF Pass% = 100%       → "PASS"
  ELSE                        → "PARTIAL PASS — REVIEW REQUIRED"
```

### 5.5 Test Case ID Sequencing

```
Format : TC-[FEATURE_CODE]-[NNN]
Example: TC-AUTH-001, TC-AUTH-002, ... TC-AUTH-099, TC-AUTH-100

Rules:
- Sequence resets per feature code
- IDs are never reused within a spec run
- Deleted/skipped TCs leave a gap (no renumbering)
```

---

## 6. Edge Cases

| EC ID | Scenario | Expected Behavior |
|-------|----------|-------------------|
| EC-01 | Spec file has only headings, no body content | Warning returned; no test cases generated; template spec provided |
| EC-02 | Spec has 100+ requirements | All requirements processed; TCs generated up to TC-XXX-999; overflow flagged as warning |
| EC-03 | Spec contains duplicate requirements (identical text) | Deduplicated before processing; only 1 set of TCs generated |
| EC-04 | Spec is written in a language other than English | Test cases generated in same language as spec |
| EC-05 | Spec has no input fields (read-only feature) | No Negative/Boundary TCs generated; only Happy Path and display verification TCs |
| EC-06 | Spec contains conflicting rules (e.g., "must be > 0" and "can be 0") | Conflict flagged in output: `"Conflicting rules detected at [line X] — manual review required"` |
| EC-07 | User submits report with all results blank | Aggregation blocked; error: `"Report has no filled results. Please complete at least 1 test case."` |
| EC-08 | User submits report with result values other than the allowed set | Row rejected with inline error: `"Invalid result '[value]' at TC-XXX-NNN. Allowed: Pass, Fail, Blocked, Skip, N/A"` |
| EC-09 | Spec has nested features (H1 > H2 > H3) | Each heading level parsed as a sub-feature; TC IDs namespaced accordingly (e.g., TC-AUTH-LOGIN-001) |
| EC-10 | Spec file contains code blocks or tables | Code/table content extracted as context; not treated as standalone requirements |
| EC-11 | QC re-submits a previously submitted report | Second submission overwrites first; updated summary generated; change log appended |
| EC-12 | All test cases are marked `Skip` or `N/A` | Aggregation result = `"NO EXECUTION — All cases skipped or not applicable"` |
| EC-13 | Spec contains external links or references | Links preserved as-is in TC preconditions/context; not followed or validated |
| EC-14 | Boundary field has no defined max (e.g., "enter any text") | Skill applies default text boundary: min=1 char, max=255 chars, and flags assumption |

---

## 7. Test Data

### 7.1 Valid Input Specs

| TD ID | Spec Content Summary | Expected TC Count | Notes |
|-------|---------------------|-------------------|-------|
| TD-V-01 | Single feature, 3 requirements, 2 input fields | 12–16 TCs | Happy path + negative + boundary |
| TD-V-02 | Auth feature: Login with email/password | 14–18 TCs | Includes Critical priority TCs |
| TD-V-03 | Search feature: keyword filter, 0–200 char limit | 10–14 TCs | Boundary on char limit |
| TD-V-04 | Read-only dashboard spec (no input fields) | 4–6 TCs | Happy path + display edge cases |
| TD-V-05 | 50-requirement spec | 150–250 TCs | Stress test; all requirements covered |

### 7.2 Invalid Input Specs

| TD ID | Input | Expected Error |
|-------|-------|---------------|
| TD-I-01 | Empty file (0 bytes) | `"Empty file detected."` |
| TD-I-02 | `.pdf` extension | `"Input must be a Markdown (.md) file"` |
| TD-I-03 | 600KB `.md` file | `"File too large. Max size is 500KB."` |
| TD-I-04 | File with only `# Title` heading, no body | `"No testable requirements detected."` |
| TD-I-05 | Non-UTF-8 encoded file (e.g., ISO-8859-1) | `"Unsupported encoding. Use UTF-8."` |

### 7.3 Sample Spec for Testing (Canonical Test Input)

```markdown
# User Authentication

## Login
- User enters email and password
- Email must be valid format (contain @)
- Password must be 8–64 characters
- After 5 failed attempts, account is locked for 15 minutes
- Successful login redirects to Dashboard

## Logout
- User can logout from any page
- Session is invalidated on logout
- User is redirected to Login page
```

**Expected output for this sample:**

| TC ID | Title | Type | Priority |
|-------|-------|------|----------|
| TC-AUTH-001 | Login with valid email and password | Happy Path | Critical |
| TC-AUTH-002 | Login with invalid email format | Negative | Critical |
| TC-AUTH-003 | Login with password below 8 chars (7 chars) | Boundary | Critical |
| TC-AUTH-004 | Login with password at min (8 chars) | Boundary | Critical |
| TC-AUTH-005 | Login with password at max (64 chars) | Boundary | Critical |
| TC-AUTH-006 | Login with password above max (65 chars) | Boundary | Critical |
| TC-AUTH-007 | Login with empty email | Negative | Critical |
| TC-AUTH-008 | Login with empty password | Negative | Critical |
| TC-AUTH-009 | Account locked after 5 failed attempts | Business Rule | Critical |
| TC-AUTH-010 | Locked account cannot login within 15 mins | Edge Case | Critical |
| TC-AUTH-011 | Account unlocks after 15 minutes | Edge Case | Critical |
| TC-AUTH-012 | Redirect to Dashboard on successful login | Happy Path | High |
| TC-AUTH-013 | Logout from Dashboard | Happy Path | High |
| TC-AUTH-014 | Logout from any non-dashboard page | Happy Path | High |
| TC-AUTH-015 | Session invalidated after logout | Business Rule | Critical |
| TC-AUTH-016 | Redirect to Login page after logout | Happy Path | High |

### 7.4 Sample Report Template Output

```markdown
## Test Report — User Authentication
**Run Date**: ___________   **Tester**: ___________   **Build/Version**: ___________

| TC ID       | Title                                          | Priority | Result | Actual Result (if Fail) | Notes |
|-------------|------------------------------------------------|----------|--------|--------------------------|-------|
| TC-AUTH-001 | Login with valid email and password            | Critical |        |                          |       |
| TC-AUTH-002 | Login with invalid email format                | Critical |        |                          |       |
| TC-AUTH-003 | Login with password below 8 chars              | Critical |        |                          |       |
...

### Summary
| Metric      | Count | % |
|-------------|-------|---|
| Total TCs   |  16   |   |
| Executed    |       |   |
| Pass        |       |   |
| Fail        |       |   |
| Blocked     |       |   |
| Skip / N/A  |       |   |

**Overall Status**: ___________
```

### 7.5 Sample Aggregated Summary Report Output

```markdown
## Test Summary Report — User Authentication
**Run Date**: 2026-02-24   **Tester**: Jane QA   **Build**: v2.1.0

### Results
| Metric      | Count | %      |
|-------------|-------|--------|
| Total TCs   |  16   | 100%   |
| Executed    |  14   |        |
| Pass        |  12   | 85.7%  |
| Fail        |   2   | 14.3%  |
| Blocked     |   0   | 0%     |
| Skip / N/A  |   2   |        |

**Overall Status**: ⛔ FAIL

### Failed Test Cases
| TC ID       | Title                               | Actual Result                              |
|-------------|-------------------------------------|--------------------------------------------|
| TC-AUTH-009 | Account locked after 5 failed attempts | Lock not triggered — user could still log in |
| TC-AUTH-015 | Session invalidated after logout    | Old session token still valid after logout |

### Recommendation
- 2 failed test cases must be resolved before release
- TC-AUTH-009 and TC-AUTH-015 are Critical priority — release is blocked
```

---

## 8. Output Format

### 8.1 Output A — Generated Test Cases

Each test case MUST be rendered in the following structure:

```markdown
### TC-[FEATURE_CODE]-[NNN]: [Title]
- **Type**: Happy Path | Negative | Boundary | Edge Case | Business Rule
- **Priority**: Critical | High | Medium | Low
- **Tags**: [feature area, e.g., auth, login, form-validation]

**Preconditions**:
- [List all conditions that must be true before executing this test]

**Test Steps**:
1. [Step 1]
2. [Step 2]
3. [Step N]

**Expected Result**:
- [What should happen after the final step]

**Notes** *(optional)*:
- [Any known limitations, related TCs, or test data references]
```

### 8.2 Output B — Report Template

- Format: Markdown table (`.md`)
- Pre-populated with all TC IDs and Titles from Output A
- Result column: empty cells for QC to fill
- Allowed result values: `Pass`, `Fail`, `Blocked`, `Skip`, `N/A`
- Summary section: auto-formula notation for QC to calculate manually OR auto-filled by Skill on re-submission

### 8.3 Output C — Aggregated Test Summary

- Format: Markdown report
- Includes: run metadata (date, tester, build), results table, pass/fail breakdown, failed TC list with actual results, overall status verdict
- Overall status values:
  - `PASS` — all executed TCs pass
  - `FAIL` — one or more High priority TCs fail
  - `BLOCKED — DO NOT RELEASE` — one or more Critical TCs fail
  - `PARTIAL PASS — REVIEW REQUIRED` — no Critical/High failures but Pass% < 100%
  - `NO EXECUTION` — all TCs are Skip or N/A

### 8.4 File Naming Convention

| Output | Suggested Filename |
|--------|-------------------|
| Test Cases | `TC-[FEATURE_CODE]-[YYYY-MM-DD].md` |
| Report Template | `REPORT-TEMPLATE-[FEATURE_CODE]-[YYYY-MM-DD].md` |
| Aggregated Summary | `SUMMARY-[FEATURE_CODE]-[YYYY-MM-DD]-[BUILD].md` |

---

## 9. Prompt Logic System

> This section defines the AI prompt contract — the exact instruction pattern the skill uses to ensure consistent, non-hallucinated output. QC can validate AI output against these examples.

### 9.1 System Prompt Template

```
You are a senior QA engineer. Your task is to generate structured test cases from a given feature specification.

Follow these rules strictly:
1. Only generate test cases based on requirements explicitly stated in the spec.
2. Do NOT invent requirements not present in the spec.
3. For each requirement, generate: Happy Path, Negative, Boundary (if applicable), and Edge Case test cases.
4. For any auth, payment, or PII-related feature, generate at least 1 Security test case.
5. Assign each TC a unique ID in format TC-[FEATURE_CODE]-[NNN].
6. Assign priority using the priority matrix (Critical/High/Medium/Low).
7. Mask all PII in test data using [PII:field_type] format.
8. Output ONLY valid Markdown. Do not include explanations outside of the defined TC structure.
9. If a requirement is ambiguous, flag it with: "⚠️ Ambiguous: [requirement text] — assumed [your interpretation]"
10. If the spec contains conflicting rules, flag: "⚠️ Conflict: [rule A] vs [rule B] at [location]"
```

### 9.2 User Prompt Template

```
Generate test cases for the following feature spec.

Feature Code: [FEATURE_CODE]
Spec content:
---
[PASTE SPEC CONTENT HERE]
---

Output format: Follow the TC template defined in Output Format Section 8.1.
Coverage required: Happy Path, Negative, Boundary, Edge Case, Security (if applicable).
```

### 9.3 Prompt Examples — Good vs Bad Output

#### Example: Login Feature

**Input spec:**
```
- User enters email and password
- Email must be valid format
- Password must be 8–64 characters
- After 5 failed attempts, account is locked
```

**GOOD AI Output (expected):**
```markdown
### TC-AUTH-001: Login with valid email and valid password
- **Type**: Happy Path
- **Priority**: Critical
- **Tags**: auth, login

**Preconditions**:
- User account exists with email [PII:email] and password meeting policy

**Test Steps**:
1. Navigate to login page
2. Enter valid email: [PII:email]
3. Enter valid password: [PII:password] (8–64 chars)
4. Click "Login"

**Expected Result**:
- User is authenticated and redirected to Dashboard
```

**BAD AI Output (must be rejected):**
```markdown
### TC-AUTH-001: Login works correctly
Steps: Login with username and password
Expected: It works
```
Rejection reason: Missing type, priority, tags, preconditions, concrete steps, and specific expected result.

### 9.4 Output Quality Gates

Before returning output, AI MUST self-check:

| Gate | Check |
|------|-------|
| QG-01 | Every TC has all 7 required fields (ID, Title, Type, Priority, Tags, Steps, Expected Result) |
| QG-02 | No TC steps are vague (e.g., "do the action", "fill in the form") |
| QG-03 | No TC expected result is vague (e.g., "it works", "success") |
| QG-04 | No PII appears in plaintext in test data |
| QG-05 | TC count matches coverage requirements from Section 5.3 |
| QG-06 | Each TC maps back to at least 1 source requirement |

---

## 10. Error Handling

### 10.1 Input Validation Errors

All input validation MUST happen before any processing begins. Errors are returned immediately with actionable messages.

| Error Code | Trigger | User-Facing Message | Action |
|------------|---------|---------------------|--------|
| ERR-IN-001 | File is not `.md` | `"Invalid file type. Only .md files are accepted."` | Halt; prompt re-upload |
| ERR-IN-002 | File is empty (0 bytes) | `"Empty file detected. Please provide a spec with content."` | Halt; prompt re-upload |
| ERR-IN-003 | File exceeds 500KB | `"File too large (max 500KB). Please reduce spec size or split into sections."` | Halt; suggest splitting |
| ERR-IN-004 | Non-UTF-8 encoding | `"Encoding error. Please save your file as UTF-8 and re-upload."` | Halt; prompt re-upload |
| ERR-IN-005 | No parseable requirements | `"No testable requirements found. Ensure spec uses recognizable formats (bullet lists, numbered items, or H2+ headings with descriptions)."` | Halt; show spec template |
| ERR-IN-006 | Plaintext PII detected | `"Security Warning: Sensitive data detected ([field]) at [location]. Remove or mask before processing."` | Warn; offer to auto-mask |
| ERR-IN-007 | Plaintext credential/token detected | `"Security Warning: Plaintext credential detected. Remove before sharing this spec."` | Warn; halt generation |

### 10.2 Processing Errors

| Error Code | Trigger | Behavior |
|------------|---------|----------|
| ERR-PR-001 | Conflicting rules detected in spec | Flag conflict inline; continue generating TCs for non-conflicting requirements |
| ERR-PR-002 | Feature code cannot be derived from spec headings | Use `FEAT` as default code; flag: `"⚠️ Could not derive feature code — using 'FEAT' as default"` |
| ERR-PR-003 | TC count would exceed 999 for one feature code | Split into sub-codes (e.g., `AUTH-A`, `AUTH-B`); flag overflow warning |
| ERR-PR-004 | Duplicate requirements detected | Deduplicate silently; log deduplicated count in output metadata |

### 10.3 Retry Mechanism

For scenarios involving external calls (API, LLM inference):

```
Retry Policy:
  Max retries    : 3
  Backoff        : Exponential (2s → 4s → 8s)
  Retry triggers : Network timeout, HTTP 5xx, LLM API rate limit (HTTP 429)
  No retry on    : HTTP 400 (bad input), HTTP 401/403 (auth), validation errors

On final failure (after 3 retries):
  → Return: "Processing failed after 3 attempts. Please try again later.
             Error: [error_code] — [error_message]"
  → Preserve user's input spec for re-submission without re-upload
```

### 10.4 Graceful Degradation

| Scenario | Fallback Behavior |
|----------|------------------|
| LLM times out mid-generation | Return partial TCs generated so far + warning: `"Generation incomplete — partial output returned"` |
| Report aggregation fails on malformed input | Skip invalid rows; flag: `"Row [TC ID] skipped — invalid result value '[value]'"` |
| JSON export fails | Fall back to Markdown output; flag: `"JSON export failed — Markdown output returned instead"` |

---

## 11. Data Schema — JSON/CSV Contract

> Enables QC to feed generated test cases directly into automation scripts without manual reformatting.

### 11.1 JSON Schema — Test Case Object

```json
{
  "$schema": "http://json-schema.org/draft-07/schema",
  "title": "TestCase",
  "type": "object",
  "required": ["id", "title", "type", "priority", "tags", "preconditions", "steps", "expected_result", "req_ids"],
  "properties": {
    "id":              { "type": "string", "pattern": "^TC-[A-Z]{2,6}-\\d{3}$", "example": "TC-AUTH-001" },
    "title":           { "type": "string", "minLength": 5 },
    "type":            { "type": "string", "enum": ["Happy Path", "Negative", "Boundary", "Edge Case", "Business Rule", "Security"] },
    "priority":        { "type": "string", "enum": ["Critical", "High", "Medium", "Low"] },
    "tags":            { "type": "array", "items": { "type": "string" }, "example": ["auth", "login", "security"] },
    "preconditions":   { "type": "array", "items": { "type": "string" } },
    "steps":           { "type": "array", "items": { "type": "string" }, "minItems": 1 },
    "expected_result": { "type": "string", "minLength": 10 },
    "notes":           { "type": "string" },
    "req_ids":         { "type": "array", "items": { "type": "string" }, "example": ["REQ-001", "REQ-003"] }
  }
}
```

### 11.2 JSON Output Example

```json
[
  {
    "id": "TC-AUTH-001",
    "title": "Login with valid email and password",
    "type": "Happy Path",
    "priority": "Critical",
    "tags": ["auth", "login"],
    "preconditions": [
      "User account exists with email [PII:email]",
      "User is on the Login page"
    ],
    "steps": [
      "Enter valid email: [PII:email]",
      "Enter valid password: [PII:password]",
      "Click 'Login' button"
    ],
    "expected_result": "User is authenticated and redirected to the Dashboard page",
    "notes": "",
    "req_ids": ["REQ-LOGIN-001"]
  }
]
```

### 11.3 CSV Schema — Report Template

```
Column order (fixed):
tc_id, title, type, priority, tags, result, actual_result, tester, date, notes

Rules:
- tc_id     : String, required, matches TC ID pattern
- title     : String, required
- type      : String, required, one of the allowed TC types
- priority  : String, required, one of: Critical | High | Medium | Low
- tags      : Pipe-separated string, e.g. "auth|login|security"
- result    : String, allowed values: Pass | Fail | Blocked | Skip | N/A | "" (empty = not yet executed)
- actual_result : String, required if result = Fail, else empty
- tester    : String, QC name
- date      : ISO 8601 date, e.g. 2026-02-24
- notes     : String, optional
```

**CSV Example:**
```csv
tc_id,title,type,priority,tags,result,actual_result,tester,date,notes
TC-AUTH-001,Login with valid credentials,Happy Path,Critical,auth|login,Pass,,Jane QC,2026-02-24,
TC-AUTH-002,Login with invalid email format,Negative,Critical,auth|login|validation,Fail,Error message not shown,Jane QC,2026-02-24,Bug filed: BUG-042
TC-AUTH-009,Account locked after 5 failures,Business Rule,Critical,auth|security,Skip,,,2026-02-24,Environment not ready
```

### 11.4 JSON Schema — Test Summary Report

```json
{
  "title": "TestSummaryReport",
  "type": "object",
  "properties": {
    "feature_code":    { "type": "string" },
    "run_date":        { "type": "string", "format": "date" },
    "tester":          { "type": "string" },
    "build_version":   { "type": "string" },
    "total":           { "type": "integer" },
    "executed":        { "type": "integer" },
    "pass":            { "type": "integer" },
    "fail":            { "type": "integer" },
    "blocked":         { "type": "integer" },
    "skip":            { "type": "integer" },
    "pass_percent":    { "type": "number" },
    "overall_status":  { "type": "string", "enum": ["PASS", "FAIL", "BLOCKED — DO NOT RELEASE", "PARTIAL PASS — REVIEW REQUIRED", "NO EXECUTION"] },
    "failed_cases":    { "type": "array", "items": { "type": "object", "properties": { "id": {"type": "string"}, "actual_result": {"type": "string"} } } }
  }
}
```

---

## 12. Traceability — Requirement Mapping

> Ensures every test case is traceable to a source requirement, and every requirement has test coverage. Prevents both under-testing and over-testing.

### 12.1 Requirement ID Format

Each requirement in the spec MUST be assigned an ID by the skill during parsing:

```
Format : REQ-[FEATURE_CODE]-[NNN]
Example: REQ-AUTH-001 = "User enters email and password"
         REQ-AUTH-002 = "Email must be valid format"
         REQ-AUTH-003 = "Password must be 8–64 characters"
```

### 12.2 Traceability Matrix

The skill MUST output a traceability matrix alongside the test cases:

| REQ ID | Requirement Summary | TC IDs Covering It | Coverage Status |
|--------|--------------------|--------------------|-----------------|
| REQ-AUTH-001 | User enters email and password | TC-AUTH-001, TC-AUTH-007, TC-AUTH-008 | Covered |
| REQ-AUTH-002 | Email must be valid format | TC-AUTH-002 | Covered |
| REQ-AUTH-003 | Password must be 8–64 characters | TC-AUTH-003, TC-AUTH-004, TC-AUTH-005, TC-AUTH-006 | Covered |
| REQ-AUTH-004 | After 5 failed attempts, account locked | TC-AUTH-009, TC-AUTH-010, TC-AUTH-011 | Covered |

**Coverage Status Values:**

| Status | Meaning |
|--------|---------|
| `Covered` | ≥1 TC maps to this requirement |
| `Partial` | TC exists but does not cover all sub-conditions of the requirement |
| `Uncovered` | No TC maps to this requirement — flagged as gap |
| `Over-covered` | >5 TCs for a simple requirement — flagged for review |

### 12.3 Coverage Report Rules

| Rule ID | Rule |
|---------|------|
| R-TR-01 | Every requirement MUST have at least 1 TC with status `Covered` |
| R-TR-02 | Any requirement with status `Uncovered` MUST be listed in a "Coverage Gaps" section |
| R-TR-03 | Each TC's `req_ids` field MUST reference at least 1 valid REQ ID |
| R-TR-04 | A TC referencing a non-existent REQ ID MUST be flagged: `"Orphan TC: [TC ID] references unknown [REQ ID]"` |
| R-TR-05 | Coverage summary MUST be included in Output A: Total REQs, Covered, Partial, Uncovered |

### 12.4 Coverage Summary Block (Output A Footer)

```markdown
## Coverage Summary
| Metric | Count |
|--------|-------|
| Total Requirements | 8 |
| Covered | 7 |
| Partial | 1 |
| Uncovered | 0 |
| Total TCs Generated | 16 |
| Orphan TCs | 0 |

### Coverage Gaps
_None — all requirements covered._

### Partially Covered
- REQ-AUTH-003: Password rule — boundary at max not tested for Unicode passwords
```

---

## 13. Security & PII Masking

### 13.1 PII Field Detection

The skill MUST automatically detect and mask the following PII field types in the spec and test data:

| PII Type | Detection Pattern | Masked As |
|----------|-----------------|-----------|
| Email address | `*@*.domain`, "email" keyword | `[PII:email]` |
| Password | "password", "passwd", "pwd" keyword | `[PII:password]` |
| Full name | "full name", "first name", "last name" | `[PII:name]` |
| Phone number | Numeric pattern `+XX-XXX...`, "phone" keyword | `[PII:phone]` |
| National ID / CCCD | Numeric 9–12 digit pattern near "ID" keyword | `[PII:national_id]` |
| Credit card | 16-digit pattern, "card number" keyword | `[PII:card_number]` |
| Date of birth | "DOB", "date of birth", "birthday" | `[PII:dob]` |
| Address | "address", "street", "city" keywords | `[PII:address]` |
| API key / Token | Alphanumeric 20+ chars near "key", "token", "secret" | `[REDACTED:token]` |

### 13.2 Masking Behavior Rules

| Rule ID | Rule |
|---------|------|
| R-PII-01 | Masking MUST happen before any TC content is generated or stored |
| R-PII-02 | Masked placeholders MUST be preserved in output so QC knows what type of data to use |
| R-PII-03 | The skill MUST NOT un-mask or regenerate real PII values at any point |
| R-PII-04 | If the spec contains a real email/phone as an example, it MUST be replaced with `[PII:email]` in all outputs |
| R-PII-05 | Masking is applied to: spec content parsed for TC generation, test step data, precondition data, expected result data |
| R-PII-06 | A PII detection report MUST be included in output metadata: `"PII detected and masked: [count] fields across [list of field types]"` |

### 13.3 Security Test Case Templates

The following templates are used when the skill auto-generates Security TCs:

#### SQL Injection Template
```markdown
### TC-[CODE]-[NNN]: SQL Injection attempt on [field_name]
- **Type**: Security
- **Priority**: Critical
- **Tags**: security, injection, [feature]

**Preconditions**:
- Application is running
- [field_name] accepts user input

**Test Steps**:
1. Navigate to [page/feature]
2. Enter SQL injection payload in [field_name]: `' OR '1'='1`
3. Submit the form/request

**Expected Result**:
- Input is rejected or sanitized
- No database error message is exposed to the user
- System returns a generic error: "Invalid input"
```

#### Broken Access Control Template
```markdown
### TC-[CODE]-[NNN]: Access [resource] as unauthorized user
- **Type**: Security
- **Priority**: Critical
- **Tags**: security, access-control, [feature]

**Preconditions**:
- Two accounts exist: User A (owner of resource) and User B (different account)
- User B is logged in

**Test Steps**:
1. Log in as User B
2. Obtain the resource URL/ID belonging to User A
3. Attempt to access User A's resource directly

**Expected Result**:
- Access is denied with HTTP 403 or equivalent
- User B is NOT able to view or modify User A's data
```

#### Session Security Template
```markdown
### TC-[CODE]-[NNN]: Session token invalidated after logout
- **Type**: Security
- **Priority**: Critical
- **Tags**: security, session, auth

**Preconditions**:
- User is logged in and has a valid session token

**Test Steps**:
1. Capture the current session token (from cookie/header)
2. Log out of the application
3. Attempt to use the captured session token in a new request

**Expected Result**:
- Token is rejected with HTTP 401
- Server returns: "Session expired or invalid"
- User is redirected to login page
```

### 13.4 Security Checklist (QC Sign-off)

- [ ] All PII fields in test cases are masked with `[PII:field_type]` placeholders
- [ ] No real email, phone, name, or ID number appears in any TC output
- [ ] Auth-related features have at least 1 Security TC covering brute force / account lockout
- [ ] Input fields have at least 1 Security TC covering injection (SQL/XSS)
- [ ] Session management has TC covering post-logout token invalidation
- [ ] Access control has TC covering unauthorized resource access (IDOR)
- [ ] PII detection report is present in output metadata
- [ ] No plaintext credentials or API tokens in spec or TC output

---

## Appendix A: QC Checklist

Use this checklist to validate the Skill's outputs before sign-off.

### A.1 Test Case Generation Checklist

- [ ] Every requirement in the spec has at least 1 Happy Path TC
- [ ] Every input field has at least 1 Negative TC
- [ ] All boundary fields have 4 Boundary TCs (min, max, min-1, max+1)
- [ ] Business rules each have at least 1 dedicated TC
- [ ] No duplicate TCs (same steps + expected result)
- [ ] All TC IDs follow format `TC-[CODE]-[NNN]`
- [ ] All TCs have: ID, Title, Preconditions, Steps, Expected Result, Priority, Tags
- [ ] Priority assignments match the priority matrix (Section 5.2)
- [ ] Coverage includes: Happy Path, Negative, Boundary, Edge Case types
- [ ] Auth/payment/PII features have at least 1 Security TC
- [ ] Each TC has `req_ids` field mapping back to a source requirement
- [ ] Traceability matrix is present and all REQ IDs have `Covered` status
- [ ] Coverage summary block is present at the bottom of Output A
- [ ] JSON export file is valid against the schema in Section 11.1
- [ ] CSV export file matches column order and value constraints in Section 11.3

### A.2 Edge Case Discovery Checklist (QC Validation)

> Verify the AI has not missed edge cases by checking each category:

**State-based:**
- [ ] Empty/null state TC exists (if feature has a list, cart, or data entity)
- [ ] First-use / onboarding state TC exists (if feature has a "new user" scenario)
- [ ] Maximum capacity state TC exists (if feature has limits)

**Concurrency:**
- [ ] Double-submit TC exists for any form submission
- [ ] Session timeout mid-flow TC exists for multi-step flows

**Data integrity:**
- [ ] Special character input TC exists for every free-text field
- [ ] Max-length input TC exists for every text field
- [ ] Whitespace-only input TC exists for required fields

**Business rule conflicts:**
- [ ] Conflicting rules are flagged in output (not silently ignored)
- [ ] Boundary-exact-value TC exists (e.g., exactly 5 attempts, not 4 or 6)

**Security:**
- [ ] SQL injection TC exists for every input field that queries a database
- [ ] XSS TC exists for every field whose value is rendered in the UI
- [ ] IDOR TC exists for every resource accessible by ID parameter
- [ ] Post-logout session TC exists for any authenticated feature

### A.3 Report Template Checklist

- [ ] All generated TC IDs appear in the report template
- [ ] Result column is blank (not pre-filled)
- [ ] Allowed result values are documented in the template
- [ ] Tester name, date, and build fields are present
- [ ] Summary section is present with correct metric labels
- [ ] CSV version of report template is available for automation use

### A.4 Aggregated Report Checklist

- [ ] Pass%, Fail%, Blocked% calculated correctly (excluding Skip/N/A)
- [ ] All Fail results listed with Actual Result notes
- [ ] Overall status verdict matches the aggregation logic (Section 5.4)
- [ ] `BLOCKED — DO NOT RELEASE` triggered for any Critical TC failure
- [ ] `BLOCKED — DO NOT RELEASE` triggered for any Security TC failure
- [ ] Report includes run metadata (date, tester, build)
- [ ] JSON summary report is valid against schema in Section 11.4

### A.5 Security & PII Checklist

- [ ] No PII appears in plaintext in any TC output (all masked as `[PII:field_type]`)
- [ ] PII detection report present in output metadata
- [ ] No plaintext credentials or API tokens in spec or generated output
- [ ] Security TCs cover: injection, access control, session management (where applicable)
- [ ] Security TC failures trigger `BLOCKED — DO NOT RELEASE` in summary

### A.6 Prompt Output Quality Checklist

- [ ] All TC steps are specific and actionable (no vague steps like "do the action")
- [ ] All expected results are specific and verifiable (no vague results like "it works")
- [ ] Ambiguous requirements are flagged with `⚠️ Ambiguous:` prefix
- [ ] Conflicting rules are flagged with `⚠️ Conflict:` prefix
- [ ] TC count meets minimum coverage requirements per type (Section 5.3)
- [ ] No TCs generated for requirements not present in the spec

---

## Appendix B: Glossary

| Term | Definition |
|------|-----------|
| Spec | Markdown file describing the feature's requirements, rules, and behavior |
| TC | Test Case |
| REQ ID | Unique identifier assigned to each parsed requirement (format: REQ-[CODE]-[NNN]) |
| Happy Path | Test where all inputs are valid and the expected success flow is verified |
| Negative Test | Test using invalid inputs to verify error handling |
| Boundary Test | Test at the exact min/max values and just outside them |
| Edge Case | Test for unusual but valid states (empty, concurrent, overflow) |
| Security TC | Test that attempts to exploit a security vulnerability (injection, access control, session) |
| Business Rule | A TC that maps directly to a stated rule in the spec |
| Blocked | A TC that could not be executed due to an upstream defect or dependency |
| Pass% | Percentage of executed (non-Skip/N/A) TCs that passed |
| Overall Status | Final release-readiness verdict based on aggregation rules |
| PII | Personally Identifiable Information (email, phone, name, ID number, etc.) |
| PII Masking | Replacing real PII values with typed placeholders (e.g., `[PII:email]`) in all outputs |
| Traceability Matrix | Table mapping each requirement to its covering test cases |
| Orphan TC | A test case whose `req_ids` reference a non-existent requirement |
| Coverage Gap | A requirement with no corresponding test case |
| JSON Contract | The agreed JSON schema that all TC export files must conform to |
| Quality Gate | A self-check rule the AI must pass before returning output |
| Retry Mechanism | Exponential backoff logic applied when external calls (API/LLM) fail transiently |
