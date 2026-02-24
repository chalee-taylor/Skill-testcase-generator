# SPEC.md — AI Skill: Test Case Generator

**Version:** 1.1.0
**Author:** QC Team — Cook A Skill
**Last Updated:** 2026-02-24
**Changelog:** Added Security category, Edge Case Checklist, Prompt Logic System, Input Validation & Retry Mechanism, JSON/CSV Data Schema Contract, Traceability Mapping, PII Masking

---

## Table of Contents

1. [Skill Overview](#1-skill-overview)
2. [Input](#2-input)
3. [Output](#3-output)
4. [Workflow](#4-workflow)
5. [Edge Case Detection Checklist](#5-edge-case-detection-checklist)
6. [Prompt Logic System](#6-prompt-logic-system)
7. [Priority Assignment Rules](#7-priority-assignment-rules)
8. [Error Handling — Input Validation & Retry Mechanism](#8-error-handling--input-validation--retry-mechanism)
9. [Data Schema — JSON & CSV Contract](#9-data-schema--json--csv-contract)
10. [Traceability — Mapping ID](#10-traceability--mapping-id)
11. [Security & PII Masking](#11-security--pii-masking)
12. [Coverage Levels](#12-coverage-levels)
13. [Constraints & Assumptions](#13-constraints--assumptions)
14. [Out of Scope](#14-out-of-scope)
15. [Success Criteria](#15-success-criteria)

---

## 1. Skill Overview

**Skill Name:** Test Case Generator
**Goal:** Automatically generate a complete, standardized list of test cases from a product feature spec file (`.md`), covering Happy Path, Negative Cases, Edge Cases, and Security Cases, along with a ready-to-use test report template.

**Problem it solves:**
- QC manually reads specs and writes test cases → slow, inconsistent, prone to missing edge cases and security gaps
- Different QC members produce different formats → hard to review and merge
- This skill reads the spec, analyzes logic, and generates structured test cases instantly with stable, predictable quality

---

## 2. Input

### 2.1 Primary Input

| Field | Type | Required | Default | Description |
|---|---|---|---|---|
| `spec_content` | `string` (Markdown) | Yes | — | Full content of the feature spec file |
| `feature_name` | `string` | No | Auto-extracted | Name of the feature being tested |
| `output_format` | `enum` | No | `markdown` | `markdown` \| `json` \| `csv` |
| `coverage_level` | `enum` | No | `standard` | `basic` \| `standard` \| `full` |
| `enable_security` | `boolean` | No | `true` | Include Security test cases in output |
| `mask_pii` | `boolean` | No | `true` | Mask PII before sending spec to AI (see Section 11) |

### 2.2 Accepted Spec Formats

The `spec_content` can come from any of these document types:

- **PRD** (Product Requirements Document)
- **BRS** (Business Requirements Specification)
- **User Story** + Acceptance Criteria
- **Feature Description** (free-form markdown)

### 2.3 Minimum Spec Quality Requirements

For the skill to produce meaningful output, the spec MUST contain at least:

- [ ] A description of what the feature does
- [ ] At least one user action or flow (e.g., "user clicks X → system does Y")
- [ ] At least one business rule or validation condition

If the spec does not meet this minimum, the skill returns a **Spec Quality Warning** (see Section 8).

### 2.4 Input Example

```markdown
## Feature: User Login

### Description
Users can log in to the system using email + password.

### Business Rules
- BR-001: Email must be valid format
- BR-002: Password must be at least 8 characters
- BR-003: After 5 failed attempts, account is locked for 15 minutes
- BR-004: On success, redirect to Dashboard

### User Flow
1. User opens /login
2. User enters email + password
3. User clicks "Login"
4. System validates credentials
5. On success → redirect to /dashboard
6. On failure → show error message
```

> **Note:** Business rules should be labeled with IDs (e.g., `BR-001`) to enable Traceability Mapping (Section 10). If no IDs exist, the skill auto-assigns them.

---

## 3. Output

### 3.1 Test Case List

Each test case follows this schema:

| Field | Type | Description |
|---|---|---|
| `id` | `string` | Sequential ID — `TC-001`, `TC-002`, ... |
| `title` | `string` | Short, action-oriented — e.g., "Login with valid credentials" |
| `type` | `enum` | `Happy Path` \| `Negative` \| `Edge Case` \| **`Security`** |
| `priority` | `enum` | `P1 - Critical` \| `P2 - High` \| `P3 - Medium` \| `P4 - Low` |
| `rule_ref` | `string` | Business rule ID(s) this case covers — e.g., `BR-001, BR-002` |
| `precondition` | `string` | System state required before executing the test |
| `test_data` | `string` | Specific data values (emails, passwords, amounts, etc.) |
| `steps` | `list[string]` | Numbered steps to execute the test |
| `expected_result` | `string` | What the system must do upon completion |
| `actual_result` | `string` | Blank — filled by QC during execution |
| `status` | `enum` | `Pass` \| `Fail` \| `Blocked` \| `N/A` — blank by default |
| `notes` | `string` | Optional — edge case context, security rationale, or known limitations |

### 3.2 Coverage Summary

| Type | Count |
|---|---|
| Happy Path | N |
| Negative | N |
| Edge Case | N |
| Security | N |
| **Total** | **N** |

### 3.3 Test Report Template

```
## Test Execution Report

- Feature: [feature_name]
- Tester: ___________
- Date: ___________
- Environment: ___________
- Build/Version: ___________

## Summary

| Total TCs | Pass | Fail | Blocked | Not Run |
|---|---|---|---|---|
| | | | | |

## Defects Found

| Bug ID | TC ID | Rule Ref | Description | Severity | Status |
|---|---|---|---|---|---|
| | | | | | |

## Sign-off

- [ ] QC Lead Review: ___________
- [ ] Security Review (if applicable): ___________
- [ ] Ready for Release
```

### 3.4 Output Example (Markdown)

```markdown
## Test Cases — User Login

| ID | Title | Type | Priority | Rule Ref | Precondition | Test Data | Steps | Expected Result |
|---|---|---|---|---|---|---|---|---|
| TC-001 | Login with valid credentials | Happy Path | P1 | BR-001, BR-002, BR-004 | User registered. Login page open. | email: user@test.com / pw: Pass@1234 | 1. Go to /login 2. Enter email + pw 3. Click Login | Redirect to /dashboard |
| TC-002 | Login with wrong password | Negative | P1 | BR-002 | User registered. Login page open. | email: user@test.com / pw: wrongpass | 1. Enter valid email 2. Enter wrong pw 3. Click Login | Error: "Invalid credentials" |
| TC-003 | Account locked after 5 failed attempts | Edge Case | P1 | BR-003 | User failed 4 times. | email: user@test.com / pw: wrongpass | 1. Attempt 5th login failure | Error: "Account locked for 15 minutes" |
| TC-004 | SQL injection in email field | Security | P1 | BR-001 | Login page open. | email: ' OR '1'='1 / pw: anything | 1. Enter SQL string as email 2. Click Login | Request rejected. No DB error exposed. |
| TC-005 | Brute-force bypass attempt | Security | P1 | BR-003 | Login page open. | Rapid automated requests | 1. Send 100 login requests in 10s | Rate limiting applied. Account locked or CAPTCHA triggered. |
```

---

## 4. Workflow

```
[INPUT]
  Spec content (.md)
       │
       ▼
[STEP 0] PII Masking (if mask_pii = true)
  - Detect & replace PII patterns before sending to AI
  - (see Section 11 for masking rules)
       │
       ▼
[STEP 1] Input Validation
  - Check spec_content is non-empty and valid Markdown
  - Check spec meets minimum quality requirements
  - If invalid → return Input Validation Error + stop
  - If low quality → return Spec Quality Warning + stop
       │
       ▼
[STEP 2] Spec Parsing
  - Extract: feature name, description, user flows, business rules,
    validations, boundary conditions, roles, integrations, auth rules
  - Auto-assign Rule IDs if spec has none (BR-001, BR-002, ...)
       │
       ▼
[STEP 3] Logic Analysis (guided by Edge Case Checklist — Section 5)
  - Identify all branches: success paths, failure paths
  - Identify all validation rules → each rule = at least 1 test case
  - Identify numeric/string boundaries → boundary value test cases
  - Identify role-based access → permission test cases
  - Identify security surface: auth, input handling, session, rate limits
       │
       ▼
[STEP 4] Test Case Generation (guided by Prompt Logic — Section 6)
  - Happy Path cases (all fields valid → success)
  - Negative cases (invalid input, missing fields, wrong state)
  - Edge Cases (boundaries, locked states, empty/overflow, concurrency)
  - Security cases (injection, brute force, auth bypass, session abuse)
  - Assign ID, priority, rule_ref, test data to each case
       │
       ▼
[STEP 5] Traceability Check (Section 10)
  - Map every Rule ID to at least 1 TC ID
  - Flag any rule with no covering test case
  - Flag any TC with no rule_ref (orphaned test case)
       │
       ▼
[STEP 6] Coverage Check
  - Ensure all types are represented per coverage_level
  - Flag any spec sections with no test case generated
       │
       ▼
[STEP 7] Output Assembly
  - Format test case table (Markdown / JSON / CSV)
  - Append Traceability Matrix
  - Append Coverage Summary
  - Append Test Report Template
       │
       ▼
[OUTPUT]
  Complete test case list + traceability matrix + report template
```

---

## 5. Edge Case Detection Checklist

This checklist is used in **Step 3** of the workflow. The AI MUST check every item against the spec and generate at least one test case for each applicable item.

### 5.1 Boundary Values

- [ ] **Minimum valid value** — e.g., password exactly 8 characters
- [ ] **Maximum valid value** — e.g., username exactly 50 characters
- [ ] **One below minimum** — e.g., password 7 characters (should fail)
- [ ] **One above maximum** — e.g., username 51 characters (should fail)
- [ ] **Zero / empty / null** — e.g., empty email field
- [ ] **Negative numbers** where only positive are allowed
- [ ] **Decimal/float** where only integers are allowed

### 5.2 String & Input Format

- [ ] **Leading/trailing whitespace** — "  user@test.com  " — does system trim or reject?
- [ ] **All spaces** — "   " as a required field
- [ ] **Special characters** — `!@#$%^&*()` in text fields
- [ ] **Unicode / emoji** — `héllo`, `用户名`, `😀` in fields not expecting them
- [ ] **Very long strings** — 1000+ character input in short text fields
- [ ] **Line breaks / newlines** in single-line fields
- [ ] **HTML tags** — `<script>alert(1)</script>` in text fields (also a Security case)

### 5.3 State & Flow

- [ ] **Action on already-completed state** — e.g., confirm an already-confirmed order
- [ ] **Action on cancelled/deleted entity** — e.g., edit a deleted user
- [ ] **Out-of-order steps** — e.g., access Step 3 without completing Step 2
- [ ] **Double submission** — submit a form twice in rapid succession
- [ ] **Back button / browser navigation** — go back after completing a flow
- [ ] **Session expiry mid-flow** — session times out while filling a form
- [ ] **Concurrent actions** — same user submits from two tabs simultaneously

### 5.4 Numeric & Calculation

- [ ] **Zero as input** where zero is a valid edge — e.g., order quantity = 0
- [ ] **Max integer overflow** — very large numbers
- [ ] **Floating point precision** — e.g., 0.1 + 0.2 ≠ 0.3 in currency
- [ ] **Negative values** in amount/quantity fields
- [ ] **Rounding behavior** — does system round up, down, or truncate?

### 5.5 Date & Time

- [ ] **Leap year dates** — Feb 29 in non-leap years
- [ ] **End-of-month dates** — e.g., Jan 31 + 1 month = Feb 28 or Mar 3?
- [ ] **Past dates** where only future dates are allowed
- [ ] **Far-future dates** — year 9999
- [ ] **Timezone edge cases** — midnight crossing timezones
- [ ] **Date format mismatch** — `DD/MM/YYYY` vs `MM/DD/YYYY`

### 5.6 Permission & Role

- [ ] **Unauthenticated access** to authenticated-only endpoints
- [ ] **Lower-privilege role** accessing higher-privilege action
- [ ] **Accessing another user's resource** (IDOR check)
- [ ] **Expired token / revoked permission** mid-session
- [ ] **Role change mid-session** — user's role is changed while logged in

### 5.7 System & Integration

- [ ] **Dependency unavailable** — external API is down (what does the UI show?)
- [ ] **Slow network / timeout** — request takes > 30s
- [ ] **Partial success** — 3 out of 5 items saved, then failure
- [ ] **Data already exists** — duplicate entry on unique fields
- [ ] **Empty list / zero results** — search returns nothing (empty state UI)
- [ ] **Pagination edge** — last page has exactly 1 item, or 0 items

### 5.8 Security (always check when `enable_security = true`)

- [ ] **SQL Injection** in all text inputs
- [ ] **XSS (Cross-Site Scripting)** in displayed fields
- [ ] **CSRF** on state-changing actions (POST/PUT/DELETE)
- [ ] **Brute force** on login/OTP/PIN fields
- [ ] **Session fixation / hijacking** — reuse old session token after logout
- [ ] **Insecure Direct Object Reference (IDOR)** — access resource by changing ID
- [ ] **Mass assignment** — inject unexpected fields in API body
- [ ] **Sensitive data in URL** — passwords/tokens exposed in query string
- [ ] **Missing rate limiting** — rapid repeated requests
- [ ] **Account enumeration** — error messages reveal if account exists

---

## 6. Prompt Logic System

This section defines the exact prompt structure and few-shot examples the AI uses to generate test cases. The goal: **stable, non-vague, actionable output every time**.

### 6.1 System Prompt Template

```
You are a senior QC engineer specializing in test case design.
Your task is to read a product feature spec and generate a complete list of test cases.

RULES:
1. Each test case MUST have: ID, Title, Type, Priority, Rule Ref, Precondition, Test Data, Steps (numbered), Expected Result.
2. Steps must be specific and executable — never write "verify the system works". Write exactly what the tester must DO.
3. Expected Result must be verifiable — never write "should work correctly". Write exactly what the tester must SEE.
4. Test Data must contain actual example values — never write "enter valid data". Write the exact data (e.g., email: user@test.com).
5. Cover ALL 4 types: Happy Path, Negative, Edge Case, Security.
6. Each business rule in the spec must be referenced by at least one test case (use rule_ref field).
7. Do NOT generate duplicate test cases. If two rules produce the same scenario, merge them into one TC with multiple rule_refs.
8. Follow the Edge Case Checklist: boundaries, state/flow, format, date, permission, security.
9. Assign priority using the Priority Rules table.
10. Output in the exact format requested (markdown table / JSON / CSV). Do not add commentary outside the format.
```

### 6.2 Few-Shot Examples

#### Example A — Good test case (Acceptable)

```
ID: TC-003
Title: Login blocked after 5th consecutive failed attempt
Type: Edge Case
Priority: P1 - Critical
Rule Ref: BR-003
Precondition: User account exists. User has already failed login 4 times within the current 15-minute window.
Test Data: email: locked@test.com / password: wrongpass99
Steps:
  1. Navigate to /login
  2. Enter email: locked@test.com
  3. Enter password: wrongpass99
  4. Click "Login" button
Expected Result: System displays error message "Your account has been locked for 15 minutes due to too many failed attempts." Login button is disabled. No redirect occurs.
```

#### Example B — Bad test case (Rejected — too vague)

```
ID: TC-003
Title: Test login failure
Type: Negative
Steps: 1. Enter wrong data 2. Submit
Expected Result: System shows error
```

> **Why rejected:** Steps are not executable (what is "wrong data"?). Expected result is not verifiable (what exact error?). No test data, no precondition, no rule_ref.

#### Example C — Good security test case

```
ID: TC-009
Title: SQL injection attempt in email field does not expose database errors
Type: Security
Priority: P1 - Critical
Rule Ref: BR-001
Precondition: Login page is accessible. No active session.
Test Data: email: ' OR '1'='1'-- / password: anything
Steps:
  1. Navigate to /login
  2. Enter email field: ' OR '1'='1'--
  3. Enter password: anything
  4. Click "Login"
Expected Result: System returns "Invalid credentials" error. No SQL error message or stack trace is displayed. Request is logged as suspicious in server logs.
```

#### Example D — Good edge case for boundary value

```
ID: TC-011
Title: Password at minimum length boundary (exactly 8 characters) is accepted
Type: Edge Case
Priority: P2 - High
Rule Ref: BR-002
Precondition: User registration page is open.
Test Data: email: boundary@test.com / password: Abcd1234 (exactly 8 chars)
Steps:
  1. Navigate to /register
  2. Enter email: boundary@test.com
  3. Enter password: Abcd1234
  4. Click "Register"
Expected Result: Registration succeeds. User is redirected to /dashboard or confirmation page. No validation error shown.
```

### 6.3 Anti-Pattern Guard

Before finalizing output, the AI must self-check each test case against this list. If any item is true, the test case must be rewritten:

| Anti-Pattern | Example of Violation |
|---|---|
| Vague steps | "Enter valid information and submit" |
| Unverifiable expected result | "System should work correctly" |
| Missing test data | "Use any email and password" |
| Duplicate scenario | Two TCs both test "empty email field" |
| No rule reference | TC with no `rule_ref` value |
| Security case with no exploit vector | "Test if login is secure" |

---

## 7. Priority Assignment Rules

| Scenario | Priority |
|---|---|
| Core happy path — feature cannot work without this | P1 - Critical |
| All Security cases (auth bypass, injection, IDOR, brute force) | P1 - Critical |
| Account lockout, rate limiting enforcement | P1 - Critical |
| All validation rules that block the main flow | P2 - High |
| Boundary values at min/max limits | P2 - High |
| Permission / role access control | P2 - High |
| UI/UX messaging (error messages, empty states) | P3 - Medium |
| Optional fields, non-blocking validations | P3 - Medium |
| Cosmetic / low-impact flows | P4 - Low |

---

## 8. Error Handling — Input Validation & Retry Mechanism

### 8.1 Input Validation (before AI call)

The skill validates input before sending anything to the AI. This prevents wasted API calls and unclear outputs.

| Validation Rule | Condition | Error Response |
|---|---|---|
| `spec_content` not empty | `len(spec_content.strip()) == 0` | `ERR-001: spec_content is required and cannot be empty` |
| `spec_content` is valid text | Contains non-binary, readable characters | `ERR-002: spec_content contains invalid characters or encoding` |
| `spec_content` minimum length | `len(spec_content) >= 50 characters` | `ERR-003: spec_content is too short to generate meaningful test cases (min 50 chars)` |
| `output_format` valid value | Must be `markdown`, `json`, or `csv` | `ERR-004: Invalid output_format. Allowed values: markdown, json, csv` |
| `coverage_level` valid value | Must be `basic`, `standard`, or `full` | `ERR-005: Invalid coverage_level. Allowed values: basic, standard, full` |
| Spec quality check | Must have flow + rule (Section 2.3) | `WARN-001: Spec Quality Warning — see details below` |

**Input Validation Error Response Format:**

```json
{
  "status": "error",
  "error_code": "ERR-001",
  "message": "spec_content is required and cannot be empty",
  "action": "Please provide the full content of your feature spec file.",
  "test_cases": null
}
```

### 8.2 Spec Quality Warning Response

```
⚠️ WARN-001: SPEC QUALITY WARNING

The provided spec is missing critical information to generate reliable test cases.

Missing elements detected:
- [ ] No user flow or action sequence found
- [ ] No business rules or validation conditions found

Recommendation — add the following to your spec before re-running:
1. At least one user flow (step-by-step: "User does X → System does Y")
2. At least one business rule or validation (e.g., "Password must be 8+ chars")
3. Expected system behavior on both success and failure

Test case generation has been paused. No output produced.
```

### 8.3 Retry Mechanism (for AI/API failures)

When the underlying AI call fails (network timeout, rate limit, model error), the skill applies exponential backoff retry:

```
Attempt 1 → fail → wait 2s
Attempt 2 → fail → wait 4s
Attempt 3 → fail → wait 8s
Attempt 4 → fail → wait 16s
Attempt 5 → fail → STOP → return ERR-010
```

| Error Type | Retry? | Max Attempts | Error Code |
|---|---|---|---|
| Network timeout | Yes | 5 | ERR-010 |
| AI rate limit (429) | Yes | 5 | ERR-011 |
| AI model error (500) | Yes | 3 | ERR-012 |
| Invalid response format from AI | Yes (re-prompt once) | 2 | ERR-013 |
| Auth error (401/403) | No — stop immediately | 1 | ERR-014 |
| Input validation failure | No — stop immediately | 1 | ERR-001 to ERR-005 |

**Max Retry Exhausted Response:**

```json
{
  "status": "error",
  "error_code": "ERR-010",
  "message": "AI service is temporarily unavailable after 5 retry attempts.",
  "action": "Please try again in a few minutes. If the issue persists, contact support.",
  "test_cases": null
}
```

### 8.4 Partial Output Handling

If the AI generates output but it fails the Anti-Pattern Guard (Section 6.3), the skill:
1. Flags the offending test cases with `status: "needs_review"`
2. Returns all test cases (including flagged ones)
3. Appends a QC Review Required notice at the top of the output

```
⚠️ QC REVIEW REQUIRED

The following test cases were flagged as potentially low quality and require manual review before use:
- TC-007: Vague expected result — "system should work correctly"
- TC-012: Missing test data

Please refine these cases before execution.
```

---

## 9. Data Schema — JSON & CSV Contract

This contract ensures QC teams can run automation scripts directly against the output without manual reformatting.

### 9.1 JSON Schema (output_format = "json")

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "TestCaseOutput",
  "type": "object",
  "required": ["meta", "test_cases", "coverage_summary", "traceability_matrix"],
  "properties": {
    "meta": {
      "type": "object",
      "properties": {
        "skill_version": { "type": "string", "example": "1.1.0" },
        "feature_name":  { "type": "string" },
        "generated_at":  { "type": "string", "format": "date-time" },
        "coverage_level":{ "type": "string", "enum": ["basic", "standard", "full"] },
        "total_rules":   { "type": "integer" },
        "total_tcs":     { "type": "integer" }
      }
    },
    "test_cases": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["id", "title", "type", "priority", "rule_ref", "precondition", "test_data", "steps", "expected_result"],
        "properties": {
          "id":              { "type": "string", "pattern": "^TC-\\d{3,}$" },
          "title":           { "type": "string", "minLength": 5 },
          "type":            { "type": "string", "enum": ["Happy Path", "Negative", "Edge Case", "Security"] },
          "priority":        { "type": "string", "enum": ["P1 - Critical", "P2 - High", "P3 - Medium", "P4 - Low"] },
          "rule_ref":        { "type": "array", "items": { "type": "string" }, "example": ["BR-001", "BR-002"] },
          "precondition":    { "type": "string" },
          "test_data":       { "type": "string" },
          "steps":           { "type": "array", "items": { "type": "string" }, "minItems": 1 },
          "expected_result": { "type": "string" },
          "actual_result":   { "type": "string", "default": "" },
          "status":          { "type": "string", "enum": ["Pass", "Fail", "Blocked", "N/A", ""], "default": "" },
          "notes":           { "type": "string", "default": "" }
        }
      }
    },
    "coverage_summary": {
      "type": "object",
      "properties": {
        "happy_path": { "type": "integer" },
        "negative":   { "type": "integer" },
        "edge_case":  { "type": "integer" },
        "security":   { "type": "integer" },
        "total":      { "type": "integer" }
      }
    },
    "traceability_matrix": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "rule_id":    { "type": "string" },
          "rule_desc":  { "type": "string" },
          "covered_by": { "type": "array", "items": { "type": "string" } },
          "status":     { "type": "string", "enum": ["Covered", "Not Covered"] }
        }
      }
    }
  }
}
```

### 9.2 JSON Output Example

```json
{
  "meta": {
    "skill_version": "1.1.0",
    "feature_name": "User Login",
    "generated_at": "2026-02-24T08:30:00Z",
    "coverage_level": "full",
    "total_rules": 4,
    "total_tcs": 9
  },
  "test_cases": [
    {
      "id": "TC-001",
      "title": "Login with valid email and password",
      "type": "Happy Path",
      "priority": "P1 - Critical",
      "rule_ref": ["BR-001", "BR-002", "BR-004"],
      "precondition": "User account exists and is active. Login page is open.",
      "test_data": "email: user@test.com / password: Pass@1234",
      "steps": [
        "Navigate to /login",
        "Enter email: user@test.com",
        "Enter password: Pass@1234",
        "Click 'Login' button"
      ],
      "expected_result": "User is redirected to /dashboard. Welcome message 'Hello, User' is displayed.",
      "actual_result": "",
      "status": "",
      "notes": ""
    }
  ],
  "coverage_summary": {
    "happy_path": 1,
    "negative": 3,
    "edge_case": 3,
    "security": 2,
    "total": 9
  },
  "traceability_matrix": [
    {
      "rule_id": "BR-001",
      "rule_desc": "Email must be valid format",
      "covered_by": ["TC-001", "TC-004", "TC-009"],
      "status": "Covered"
    }
  ]
}
```

### 9.3 CSV Schema (output_format = "csv")

```
id,title,type,priority,rule_ref,precondition,test_data,steps,expected_result,actual_result,status,notes
```

**CSV Rules:**
- `steps` are joined with ` | ` separator (e.g., `"Step 1 | Step 2 | Step 3"`)
- `rule_ref` are joined with `;` separator (e.g., `"BR-001;BR-002"`)
- All fields are double-quoted
- Encoding: UTF-8 with BOM (for Excel compatibility)
- First row is always the header

**CSV Example:**

```csv
"id","title","type","priority","rule_ref","precondition","test_data","steps","expected_result","actual_result","status","notes"
"TC-001","Login with valid credentials","Happy Path","P1 - Critical","BR-001;BR-002;BR-004","User registered. Login page open.","email: user@test.com / pw: Pass@1234","Navigate to /login | Enter email | Enter password | Click Login","Redirect to /dashboard","","",""
"TC-009","SQL injection in email field","Security","P1 - Critical","BR-001","Login page open.","email: ' OR '1'='1'--","Navigate to /login | Enter SQL string in email | Click Login","Error shown. No DB error exposed.","","",""
```

---

## 10. Traceability — Mapping ID

Traceability ensures every business rule is tested and every test case has a reason to exist.

### 10.1 Rule ID Auto-Assignment

If the input spec has no rule IDs, the skill auto-assigns them during Spec Parsing:

```
Business Rules found in spec → assigned IDs:
  "Email must be valid format"             → BR-001
  "Password must be at least 8 characters" → BR-002
  "After 5 failed attempts, lock account"  → BR-003
  "On success, redirect to Dashboard"      → BR-004
```

### 10.2 Traceability Matrix Output

The matrix is always included in the output (all formats):

| Rule ID | Rule Description | Covered By | Status |
|---|---|---|---|
| BR-001 | Email must be valid format | TC-001, TC-004, TC-009 | Covered |
| BR-002 | Password minimum 8 characters | TC-001, TC-006, TC-011 | Covered |
| BR-003 | Lock after 5 failed attempts | TC-003, TC-008, TC-010 | Covered |
| BR-004 | Redirect to /dashboard on success | TC-001, TC-002 | Covered |

### 10.3 Coverage Gap Detection

After building the matrix, the skill flags:

**Uncovered Rules** — Rules with no test case:
```
⚠️ COVERAGE GAP: BR-005 "Remember Me checkbox functionality" has no test case.
  Reason: Rule found in spec but no testable behavior described.
  Action: Add expected behavior for "Remember Me" to the spec, or manually add TC.
```

**Orphaned Test Cases** — TCs with no rule_ref:
```
⚠️ ORPHANED TC: TC-015 has no rule_ref.
  Action: Verify this TC is intentional. If so, add the corresponding rule to spec.
```

### 10.4 Coverage Percentage

```
Coverage Report:
  Total Rules:     5
  Covered Rules:   4
  Coverage:        80%

  Total TCs:       9
  Orphaned TCs:    0
```

---

## 11. Security & PII Masking

### 11.1 Why PII Masking

When QC pastes real spec files into the AI skill, those specs may contain:
- Real customer names, emails, phone numbers
- Internal API keys or credentials accidentally pasted
- Internal project code names or confidential business logic

PII Masking runs **before** the spec is sent to the AI, replacing sensitive data with safe placeholders.

### 11.2 PII Detection & Masking Rules

| Pattern | Regex / Rule | Replacement |
|---|---|---|
| Email addresses | `[a-zA-Z0-9._%+\-]+@[a-zA-Z0-9.\-]+\.[a-zA-Z]{2,}` | `[EMAIL_REDACTED]` |
| Phone numbers | `(\+?\d{1,3}[\s\-]?)?\(?\d{3}\)?[\s\-]?\d{3}[\s\-]?\d{4}` | `[PHONE_REDACTED]` |
| API keys / tokens | Strings 20+ chars matching `[A-Za-z0-9_\-]{20,}` in key-value context | `[API_KEY_REDACTED]` |
| JWT tokens | Strings matching `eyJ[A-Za-z0-9_\-]+\.[A-Za-z0-9_\-]+\.[A-Za-z0-9_\-]+` | `[JWT_REDACTED]` |
| Vietnamese national ID | 9 or 12 consecutive digits | `[ID_NUMBER_REDACTED]` |
| Credit card numbers | 13-19 digits, optionally separated by spaces/dashes | `[CARD_REDACTED]` |
| IP addresses | `\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}` | `[IP_REDACTED]` |
| Internal URLs | Patterns matching `https?://[^/]*\.(internal|corp|local|intranet)` | `[INTERNAL_URL_REDACTED]` |

### 11.3 Masking Behavior

- Masking is **lossless for structure** — the logical meaning of the spec is preserved, only the values are redacted
- Masked spec is used **only for AI processing** — the original spec is never stored or logged
- The test cases generated use **synthetic test data**, never the original PII values
- If `mask_pii = false`, a warning is shown:

```
⚠️ PII MASKING DISABLED
  mask_pii is set to false. Your spec will be sent to the AI as-is.
  Ensure no real customer data, credentials, or confidential information is present.
```

### 11.4 Masking Audit Log

For compliance, the skill logs a redaction summary (not the actual values):

```
PII Masking Report:
  Fields redacted: 3
  Types detected:
    - EMAIL_REDACTED: 2 occurrences
    - API_KEY_REDACTED: 1 occurrence
  Spec sent to AI: [masked version]
  Original spec: [retained in local session only, not transmitted]
```

### 11.5 Security Test Cases — Standard Set

When `enable_security = true`, the skill always generates this minimum set of security test cases (in addition to spec-specific ones):

| Scenario | Type | Priority | Always Generated? |
|---|---|---|---|
| SQL Injection in all text inputs | Security | P1 | Yes |
| XSS in fields that display user input | Security | P1 | Yes |
| Unauthenticated access to protected endpoint | Security | P1 | If auth rules found in spec |
| Brute force on login/OTP/PIN | Security | P1 | If auth rules found in spec |
| IDOR — accessing another user's resource by ID | Security | P1 | If user-specific resources found |
| Session token still valid after logout | Security | P1 | If session/auth found in spec |
| Sensitive data in URL parameters | Security | P2 | If redirects/tokens found in spec |
| Rate limiting on API endpoints | Security | P2 | If API calls found in spec |

---

## 12. Coverage Levels

| Level | Happy Path | Negative | Edge Case | Security | Traceability Matrix |
|---|---|---|---|---|---|
| `basic` | Yes | No | No | No | No |
| `standard` | Yes | Yes | No | Yes (minimal) | Yes |
| `full` | Yes | Yes | Yes (full checklist) | Yes (full set) | Yes |

---

## 13. Constraints & Assumptions

- **Constraint:** Generates **functional + security manual test cases** only. Does not generate automation scripts, performance tests, or penetration test scripts.
- **Constraint:** Test cases reflect what is **explicitly stated** in the spec. Implicit business logic not in the spec is not inferred (except for security standard set).
- **Assumption:** Spec is written in English or Vietnamese. Mixed-language specs are supported.
- **Assumption:** QC reviews output before execution. The skill accelerates, not replaces, QC judgment.
- **Assumption:** Each run handles **one feature** at a time. Multi-feature specs should be split before input.
- **Assumption:** PII Masking is enabled by default. Disabling it is the user's explicit responsibility.

---

## 14. Out of Scope

- Selenium / Playwright / Cypress automation scripts
- API test scripts (Postman collections, REST Assured)
- Performance / load / stress test plans
- Penetration testing scripts or exploit code
- Bug reports or defect logs (separate tool)
- Integration with Jira, TestRail, or any external tool

---

## 15. Success Criteria

The skill output is considered successful when:

- [ ] Every business rule maps to at least 1 test case (`rule_ref` populated)
- [ ] Every user flow step is exercised in at least 1 test case
- [ ] All 4 types represented: Happy Path, Negative, Edge Case, Security — for `standard` / `full` levels
- [ ] All test cases have all required fields (no blanks in required columns)
- [ ] No test case matches any Anti-Pattern in Section 6.3
- [ ] Traceability Matrix is complete — no uncovered rules, no orphaned TCs
- [ ] Coverage percentage ≥ 100% (all rules covered)
- [ ] PII Masking report generated (when `mask_pii = true`)
- [ ] JSON output validates against schema in Section 9.1
- [ ] CSV output is parseable by standard CSV parsers (Excel, Python `csv`, etc.)
- [ ] QC requires only minor edits before using the output
