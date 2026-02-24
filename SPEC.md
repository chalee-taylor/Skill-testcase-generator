# SPEC.md — AI Skill: Test Case Generator

**Version:** 1.2.0
**Author:** QC Team — Cook A Skill
**Last Updated:** 2026-02-24
**Changelog:** Integrated Security category, PII Masking, Input Validation, Edge Case Checklist, Prompt Logic, Retry Mechanism, JSON/CSV Schema, and Traceability Matrix into existing sections (no new top-level sections added)

---

## 1. Skill Overview

**Skill Name:** Test Case Generator
**Goal:** Automatically generate a complete, standardized list of test cases from a product feature spec file (`.md`), covering Happy Path, Negative Cases, Edge Cases, and **Security Cases**, along with a ready-to-use test report template.

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
| `mask_pii` | `boolean` | No | `true` | Mask PII in spec before sending to AI (see 2.5) |

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

If the spec does not meet this minimum, the skill returns a **Spec Quality Warning** (see Section 6).

> **Tip:** Label business rules with IDs (e.g., `BR-001`) to enable Traceability Mapping in the output. If no IDs exist, the skill auto-assigns them during parsing.

### 2.4 Input Validation

The skill validates all parameters **before** calling the AI. This prevents wasted API calls and ambiguous outputs.

| Check | Condition | Error Code |
|---|---|---|
| `spec_content` not empty | `len(spec_content.strip()) == 0` | `ERR-001` |
| `spec_content` readable encoding | Non-binary, valid UTF-8 text | `ERR-002` |
| `spec_content` minimum length | `len >= 50 characters` | `ERR-003` |
| `output_format` valid value | `markdown` \| `json` \| `csv` | `ERR-004` |
| `coverage_level` valid value | `basic` \| `standard` \| `full` | `ERR-005` |
| Spec quality check | Has flow + rule (Section 2.3) | `WARN-001` |

**Error response format (all validation failures):**

```json
{
  "status": "error",
  "error_code": "ERR-001",
  "message": "spec_content is required and cannot be empty",
  "action": "Please provide the full content of your feature spec file.",
  "test_cases": null
}
```

### 2.5 PII Masking

When `mask_pii = true` (default), the skill automatically detects and replaces sensitive values in the spec **before** sending it to the AI. The original spec is never transmitted or stored.

| Pattern Type | Detection Rule | Replacement |
|---|---|---|
| Email addresses | `[a-zA-Z0-9._%+\-]+@[a-zA-Z0-9.\-]+\.[a-zA-Z]{2,}` | `[EMAIL_REDACTED]` |
| Phone numbers | 9–11 digit patterns with optional country code | `[PHONE_REDACTED]` |
| API keys / tokens | 20+ char alphanumeric strings in key-value context | `[API_KEY_REDACTED]` |
| JWT tokens | `eyJ...` three-part base64 pattern | `[JWT_REDACTED]` |
| Vietnamese national ID | 9 or 12 consecutive digits | `[ID_NUMBER_REDACTED]` |
| Credit card numbers | 13–19 digits, optionally space/dash separated | `[CARD_REDACTED]` |
| IP addresses | `\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}` | `[IP_REDACTED]` |
| Internal URLs | Domains ending in `.internal`, `.corp`, `.local`, `.intranet` | `[INTERNAL_URL_REDACTED]` |

Masking is **structure-preserving** — business logic and flow wording remain intact. Test cases generated use synthetic placeholder data, never original PII values.

If `mask_pii = false`, the output includes:

```
⚠️ PII MASKING DISABLED — spec was sent to AI as-is.
   Ensure no real customer data, credentials, or confidential project information is present.
```

A masking audit summary is always appended to the output:

```
PII Masking Report: 3 values redacted (EMAIL_REDACTED ×2, API_KEY_REDACTED ×1)
```

### 2.6 Input Example

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

---

## 3. Output

### 3.1 Test Case Schema

Each test case follows this schema:

| Field | Type | Description |
|---|---|---|
| `id` | `string` | Sequential ID — `TC-001`, `TC-002`, ... |
| `title` | `string` | Short, action-oriented — e.g., "Login with valid credentials" |
| `type` | `enum` | `Happy Path` \| `Negative` \| `Edge Case` \| `Security` |
| `priority` | `enum` | `P1 - Critical` \| `P2 - High` \| `P3 - Medium` \| `P4 - Low` |
| `rule_ref` | `string` | Business rule ID(s) this case covers — e.g., `BR-001, BR-002` |
| `precondition` | `string` | System state required before executing the test |
| `test_data` | `string` | Exact data values to use — never generic, always specific |
| `steps` | `list[string]` | Numbered, executable steps |
| `expected_result` | `string` | Specific, verifiable outcome the tester must observe |
| `actual_result` | `string` | Blank — filled by QC during execution |
| `status` | `enum` | `Pass` \| `Fail` \| `Blocked` \| `N/A` — blank by default |
| `notes` | `string` | Optional — edge case rationale or security context |

### 3.2 Coverage Summary

After the test case list, the skill generates:

| Type | Count |
|---|---|
| Happy Path | N |
| Negative | N |
| Edge Case | N |
| Security | N |
| **Total** | **N** |

### 3.3 Traceability Matrix

Always included in output (all formats). Maps every business rule to the test cases covering it.

| Rule ID | Rule Description | Covered By | Status |
|---|---|---|---|
| BR-001 | Email must be valid format | TC-001, TC-004, TC-009 | Covered |
| BR-002 | Password minimum 8 characters | TC-001, TC-006, TC-011 | Covered |
| BR-003 | Lock after 5 failed attempts | TC-003, TC-008, TC-010 | Covered |
| BR-004 | Redirect to /dashboard on success | TC-001, TC-002 | Covered |

Coverage gaps are automatically flagged:

```
⚠️ COVERAGE GAP: BR-005 "Remember Me checkbox" — no test case found.
   Action: Add expected behavior to spec, or manually write TC.

⚠️ ORPHANED TC: TC-015 has no rule_ref.
   Action: Confirm this TC is intentional. Add corresponding rule to spec if needed.

Coverage: 4 / 5 rules covered (80%) — 0 orphaned TCs
```

### 3.4 JSON & CSV Output Contract

When `output_format` is `json` or `csv`, the output must match the contracts below so QC automation scripts can consume it directly.

**JSON top-level structure (required fields):**

```json
{
  "meta": {
    "skill_version": "1.2.0",
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
      "steps": ["Navigate to /login", "Enter email: user@test.com", "Enter password: Pass@1234", "Click 'Login'"],
      "expected_result": "User redirected to /dashboard. Welcome message displayed.",
      "actual_result": "",
      "status": "",
      "notes": ""
    }
  ],
  "coverage_summary": { "happy_path": 1, "negative": 3, "edge_case": 3, "security": 2, "total": 9 },
  "traceability_matrix": [
    { "rule_id": "BR-001", "rule_desc": "Email must be valid format", "covered_by": ["TC-001", "TC-004", "TC-009"], "status": "Covered" }
  ]
}
```

**JSON field constraints:**
- `id` must match pattern `TC-\d{3,}`
- `type` must be one of: `Happy Path`, `Negative`, `Edge Case`, `Security`
- `priority` must be one of: `P1 - Critical`, `P2 - High`, `P3 - Medium`, `P4 - Low`
- `rule_ref` is an array of strings — at least one value required
- `steps` is an array of strings — at least one step required

**CSV header (exact column order):**

```
"id","title","type","priority","rule_ref","precondition","test_data","steps","expected_result","actual_result","status","notes"
```

**CSV encoding rules:**
- `steps` joined with ` | ` (pipe separator)
- `rule_ref` joined with `;`
- All fields double-quoted
- Encoding: UTF-8 with BOM (Excel-compatible)

**CSV example rows:**

```csv
"TC-001","Login with valid credentials","Happy Path","P1 - Critical","BR-001;BR-002;BR-004","User registered. Login page open.","email: user@test.com / pw: Pass@1234","Navigate to /login | Enter email | Enter password | Click Login","Redirect to /dashboard","","",""
"TC-009","SQL injection in email field","Security","P1 - Critical","BR-001","Login page open.","email: ' OR '1'='1'--","Navigate to /login | Enter SQL payload in email | Click Login","Error shown. No DB error or stack trace exposed.","","",""
```

### 3.5 Test Report Template

Appended at the end of every output, ready for QC to fill in:

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
- [ ] Security Review (if Security TCs present): ___________
- [ ] Ready for Release
```

### 3.6 Output Example (Markdown)

```markdown
## Test Cases — User Login

| ID | Title | Type | Priority | Rule Ref | Precondition | Test Data | Steps | Expected Result |
|---|---|---|---|---|---|---|---|---|
| TC-001 | Login with valid credentials | Happy Path | P1 | BR-001,BR-002,BR-004 | Registered user. Login page open. | email: user@test.com / pw: Pass@1234 | 1. Go to /login 2. Enter email 3. Enter pw 4. Click Login | Redirected to /dashboard |
| TC-002 | Login with wrong password | Negative | P1 | BR-002 | Registered user. Login page open. | email: user@test.com / pw: wrongpass | 1. Enter valid email 2. Enter wrong pw 3. Click Login | Error: "Invalid credentials". Stay on /login. |
| TC-003 | Account locked on 5th failed attempt | Edge Case | P1 | BR-003 | User failed 4 times already. | email: user@test.com / pw: wrongpass | 1. Attempt 5th login failure | Error: "Account locked for 15 minutes". |
| TC-004 | SQL injection in email field | Security | P1 | BR-001 | Login page open. | email: ' OR '1'='1'-- / pw: anything | 1. Enter SQL string as email 2. Click Login | Request rejected. No DB error exposed. |
```

---

## 4. Workflow

```
[INPUT]
  Spec content (.md)
       │
       ▼
[STEP 1] PII Masking (if mask_pii = true — see Section 2.5)
  - Detect and replace PII patterns in spec content
  - Log redaction summary for audit
       │
       ▼
[STEP 2] Input Validation (see Section 2.4)
  - Validate all input parameters against rules (ERR-001 to ERR-005)
  - Check spec meets minimum quality requirements
  - If invalid → return error JSON + stop
  - If low quality → return WARN-001 + stop
       │
       ▼
[STEP 3] Spec Parsing
  - Extract: feature name, description, user flows, business rules,
    validations, boundary conditions, roles, integrations, auth rules
  - Auto-assign Rule IDs if spec has none (BR-001, BR-002, ...)
       │
       ▼
[STEP 4] Logic Analysis
  - Identify all branches: success paths, failure paths
  - Identify all validation rules → each rule = at least 1 test case
  - Identify numeric/string boundaries → boundary value test cases
  - Identify role-based access → permission test cases
  - Identify security surface: auth, input handling, session, rate limits
  - Apply Edge Case Checklist (see 4.1 below)
       │
       ▼
[STEP 5] Test Case Generation
  - Apply Prompt Logic rules (see 4.2 below) to generate all cases
  - Happy Path cases (all fields valid → success)
  - Negative cases (invalid input, missing fields, wrong state)
  - Edge Cases (boundaries, locked states, concurrency, formats)
  - Security cases (injection, brute force, auth bypass, session abuse)
  - Assign ID, priority, rule_ref, test data to each case
  - Run Anti-Pattern Guard — rewrite any flagged test case
       │
       ▼
[STEP 6] Coverage & Traceability Check
  - Map every Rule ID → TC IDs (Traceability Matrix — see Section 3.3)
  - Flag uncovered rules + orphaned TCs
  - Ensure all types represented per coverage_level
  - Calculate coverage percentage
       │
       ▼
[STEP 7] Output Assembly
  - Format test case table (Markdown / JSON / CSV per contract in Section 3.4)
  - Append Traceability Matrix + Coverage Summary
  - Append Test Report Template
  - Append PII Masking audit summary
       │
       ▼
[OUTPUT]
  Test cases + Traceability Matrix + Coverage Summary + Report Template
```

### 4.1 Edge Case Detection Checklist

Used in **Step 4**. The AI MUST check every item against the spec and generate at least one test case for each applicable item found.

**Boundary Values**
- [ ] Minimum valid value — e.g., password exactly 8 chars
- [ ] Maximum valid value — e.g., username exactly 50 chars
- [ ] One below minimum (must fail) — e.g., password 7 chars
- [ ] One above maximum (must fail) — e.g., username 51 chars
- [ ] Zero / empty / null on required fields
- [ ] Negative numbers where only positive are allowed
- [ ] Decimal/float where only integers are accepted

**String & Input Format**
- [ ] Leading/trailing whitespace — does system trim or reject?
- [ ] All-spaces input on required text fields
- [ ] Special characters — `!@#$%^&*()` in text inputs
- [ ] Unicode / emoji — `héllo`, `用户名`, `😀` in fields not expecting them
- [ ] Very long strings — 1000+ chars in short-text fields
- [ ] Line breaks / newlines in single-line fields
- [ ] HTML tags — `<script>alert(1)</script>` in displayed fields (also a Security case)

**State & Flow**
- [ ] Action on already-completed state — e.g., re-confirm an order
- [ ] Action on cancelled/deleted entity — e.g., edit a deleted user
- [ ] Out-of-order steps — access Step 3 without finishing Step 2
- [ ] Double submission — submit the same form twice rapidly
- [ ] Back button after completing a flow
- [ ] Session expiry mid-flow
- [ ] Concurrent actions — same user in two tabs simultaneously

**Numeric & Calculation**
- [ ] Zero as valid edge input — e.g., quantity = 0
- [ ] Max integer overflow
- [ ] Floating point precision — currency rounding edge (0.1 + 0.2)
- [ ] Negative values in amount/quantity fields
- [ ] Rounding behavior — round up, down, or truncate?

**Date & Time**
- [ ] Leap year — Feb 29 in non-leap year
- [ ] End-of-month arithmetic — Jan 31 + 1 month
- [ ] Past dates where only future dates are allowed
- [ ] Far-future dates — year 9999
- [ ] Timezone edge — midnight crossing timezones
- [ ] Date format mismatch — `DD/MM/YYYY` vs `MM/DD/YYYY`

**Permission & Role**
- [ ] Unauthenticated access to auth-only endpoints
- [ ] Lower-privilege role accessing higher-privilege action
- [ ] Accessing another user's resource (IDOR)
- [ ] Expired token / revoked permission mid-session
- [ ] Role change while session is active

**System & Integration**
- [ ] External dependency unavailable (API down — what does UI show?)
- [ ] Slow response / timeout (request > 30s)
- [ ] Partial success — some items saved, then failure
- [ ] Duplicate entry on unique-constrained fields
- [ ] Empty result set — search returns nothing (empty state UI)
- [ ] Pagination boundary — last page with 1 item, or 0 items

**Security** *(always check when `enable_security = true`)*
- [ ] SQL Injection in all text inputs
- [ ] XSS in fields whose values are later displayed
- [ ] CSRF on all state-changing actions (POST/PUT/DELETE)
- [ ] Brute force on login / OTP / PIN endpoints
- [ ] Session fixation — reuse old session token after logout
- [ ] IDOR — access resource by changing numeric ID in URL/body
- [ ] Mass assignment — inject unexpected fields in request body
- [ ] Sensitive data in URL — password/token in query string
- [ ] Missing rate limiting — rapid repeated identical requests
- [ ] Account enumeration — error messages reveal account existence

### 4.2 Prompt Logic System

Used in **Step 5**. Defines how the AI must construct each test case to ensure stable, non-vague, actionable output.

**System prompt rules (enforced for every test case):**

```
1. Every test case MUST include: ID, Title, Type, Priority, Rule Ref,
   Precondition, Test Data, Steps (numbered), Expected Result.
2. Steps must be executable — never "verify it works". Write exactly what the tester must DO.
3. Expected Result must be verifiable — never "should work correctly". Write exactly what the tester must SEE.
4. Test Data must contain actual example values — never "enter valid data". Use real examples (email: user@test.com).
5. Cover all 4 types: Happy Path, Negative, Edge Case, Security.
6. Every business rule must be referenced by at least one TC (rule_ref field).
7. Do not duplicate scenarios — merge identical cases, combine rule_refs.
8. Apply the Edge Case Checklist (Section 4.1) systematically.
9. Assign priority per the Priority Rules table (Section 5).
10. Output in the exact requested format. No commentary outside the format.
```

**Few-shot quality examples:**

| Quality | Example |
|---|---|
| Good — Edge Case | *Title:* "Account locked on 5th consecutive failed login" / *Steps:* 1. Fail login 4× 2. Enter email: user@test.com, pw: wrong 3. Click Login / *Expected:* "Account locked for 15 minutes" message appears. Login button disabled. |
| Bad — Rejected | *Steps:* "Enter wrong data and submit" / *Expected:* "System shows error" — **Rejected: vague steps, unverifiable result, no test data** |
| Good — Security | *Title:* "SQL injection in email field does not expose DB errors" / *Test Data:* `email: ' OR '1'='1'--` / *Expected:* "Invalid credentials" shown, no SQL error or stack trace displayed |
| Good — Boundary | *Title:* "Password at exactly 8 chars (minimum) is accepted" / *Test Data:* `password: Abcd1234` (8 chars) / *Expected:* Registration succeeds, no validation error |

**Anti-Pattern Guard — rewrite required if any of these are true:**

| Anti-Pattern | Violation Example |
|---|---|
| Vague steps | "Enter valid information and submit" |
| Unverifiable expected result | "System should work correctly" |
| Missing concrete test data | "Use any email and password" |
| Duplicate scenario | Two TCs both testing "empty email field" |
| Missing rule_ref | TC with no business rule referenced |
| Security TC with no exploit vector | "Test if login is secure" |

---

## 5. Priority Assignment Rules

| Scenario | Priority |
|---|---|
| Core happy path — feature cannot function without this | P1 - Critical |
| All Security cases (injection, auth bypass, IDOR, brute force) | P1 - Critical |
| Account lockout and rate limiting enforcement | P1 - Critical |
| All validation rules that block the main user flow | P2 - High |
| Boundary values at minimum/maximum limits | P2 - High |
| Permission and role access control | P2 - High |
| UI/UX messaging — error messages, empty states | P3 - Medium |
| Optional fields and non-blocking validations | P3 - Medium |
| Cosmetic and low-impact flows | P4 - Low |

---

## 6. Error Handling & Spec Quality Warning

### 6.1 Input Validation Errors

When parameter validation fails (Section 2.4), generation stops immediately. No retry for these — fix the input first.

| Error Code | Trigger | Message |
|---|---|---|
| `ERR-001` | Empty `spec_content` | `spec_content is required and cannot be empty` |
| `ERR-002` | Invalid encoding | `spec_content contains invalid characters or encoding` |
| `ERR-003` | Too short (< 50 chars) | `spec_content too short to generate meaningful test cases` |
| `ERR-004` | Invalid `output_format` | `Invalid output_format — allowed: markdown, json, csv` |
| `ERR-005` | Invalid `coverage_level` | `Invalid coverage_level — allowed: basic, standard, full` |

### 6.2 Spec Quality Warning

When spec passes input validation but fails the minimum quality check (Section 2.3):

```
⚠️ WARN-001: SPEC QUALITY WARNING

Missing elements detected:
- [ ] No user flow or action sequence found
- [ ] No business rules or validation conditions found

Add to your spec before re-running:
1. At least one user flow ("User does X → System does Y")
2. At least one business rule or validation condition
3. Expected behavior on both success and failure

Test case generation paused. No output produced.
```

### 6.3 AI/API Failure — Retry Mechanism

When the AI service call fails, the skill retries with exponential backoff before giving up:

```
Attempt 1 → fail → wait 2s
Attempt 2 → fail → wait 4s
Attempt 3 → fail → wait 8s
Attempt 4 → fail → wait 16s
Attempt 5 → fail → STOP → return error
```

| Error Type | Retry? | Max Attempts | Error Code |
|---|---|---|---|
| Network timeout | Yes | 5 | `ERR-010` |
| Rate limit (HTTP 429) | Yes | 5 | `ERR-011` |
| AI model error (HTTP 500) | Yes | 3 | `ERR-012` |
| Malformed AI response | Yes — re-prompt once | 2 | `ERR-013` |
| Auth error (HTTP 401/403) | No | 1 | `ERR-014` |

Max retries exhausted response:

```json
{
  "status": "error",
  "error_code": "ERR-010",
  "message": "AI service unavailable after 5 attempts.",
  "action": "Please retry in a few minutes. Contact support if issue persists.",
  "test_cases": null
}
```

### 6.4 Partial Output — Quality Flag

If generated test cases fail the Anti-Pattern Guard (Section 4.2) after retry, they are flagged rather than dropped:

- Flagged TCs get `status: "needs_review"` in the output
- All test cases (including flagged) are returned
- A warning header is prepended:

```
⚠️ QC REVIEW REQUIRED
The following test cases were flagged as low quality — review before execution:
  TC-007: Vague expected result
  TC-012: Missing test data
```

---

## 7. Coverage Levels

| Level | Happy Path | Negative | Edge Cases | Security | Traceability Matrix |
|---|---|---|---|---|---|
| `basic` | Yes | No | No | No | No |
| `standard` | Yes | Yes | No | Minimal (SQL Injection + XSS always) | Yes |
| `full` | Yes | Yes | Full checklist (Section 4.1) | Full checklist (Section 4.1) | Yes |

**Security minimum for `standard` level:** SQL Injection and XSS are always generated for any feature with text input fields, regardless of whether auth rules exist in the spec.

---

## 8. Constraints & Assumptions

- **Constraint:** Generates **functional and security manual test cases** only. Does not produce automation scripts, performance tests, or penetration test scripts.
- **Constraint:** Test cases reflect what is **explicitly stated** in the spec. Implicit business logic not written in the spec is not inferred (except the security minimum set in Section 7).
- **Assumption:** Spec is written in English or Vietnamese. Mixed-language specs are supported.
- **Assumption:** QC reviews and validates output before test execution. The skill is an accelerator, not a replacement for QC judgment.
- **Assumption:** Each run handles **one feature at a time**. Multi-feature specs should be split before input.
- **Assumption:** `mask_pii = true` by default. Disabling PII masking is the user's explicit responsibility.

---

## 9. Out of Scope

- Selenium / Playwright / Cypress automation scripts
- API test scripts (Postman collections, REST Assured)
- Performance / load / stress test plans
- Penetration testing scripts or exploit code
- Bug reports or defect logs (handled by a separate tool)
- Integration with Jira, TestRail, or any external test management system

---

## 10. Success Criteria

The skill output is considered successful when:

- [ ] Every business rule in the spec maps to at least 1 test case (`rule_ref` populated)
- [ ] Every user flow step is exercised in at least 1 test case
- [ ] All 4 types are represented: Happy Path, Negative, Edge Case, Security — for `standard` and `full` levels
- [ ] All test cases have all required fields — no blanks in required columns
- [ ] No test case matches any Anti-Pattern from Section 4.2
- [ ] Traceability Matrix is complete — no uncovered rules, no orphaned TCs (Section 3.3)
- [ ] Coverage percentage ≥ 100% (all rules covered)
- [ ] PII Masking audit summary included when `mask_pii = true`
- [ ] JSON output satisfies all field constraints in Section 3.4
- [ ] CSV output is parseable by Excel, Python `csv`, or any standard CSV parser
- [ ] QC requires only minor edits before using the output
