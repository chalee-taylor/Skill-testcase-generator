# Skill Card — Test Case Generator

---

## Skill Name
**Test Case Generator**

Automatically generates test cases from spec/PRD/BRS files, covering Happy Path, Negative, Edge Case, and Security — output is ready to use immediately.

---

## What Is Automated

QC/Tester submits a spec file (`.md`) → Skill reads the spec, analyzes business logic, detects edge cases, and generates a complete, standardized test case suite — including a Traceability Matrix and Test Report Template.

---

## BEFORE: Manual Process

| Step | Description |
|------|-------------|
| 1 | QC reads the spec manually — line by line to understand the business logic |
| 2 | Thinks through scenarios: happy path, failures, boundary values |
| 3 | Types each test case into a sheet or document |
| 4 | Reviews the list to check for missing cases |
| 5 | Reformats everything to match the team standard |

**Time:** 2–4 hours / feature (complex features can take 6–8 hours)

**Risks:**
- Edge cases and security cases are easily missed
- Each QC writes in a different format → hard to review and merge
- Quality depends entirely on individual experience

---

## AFTER: With the Skill

| Step | Description |
|------|-------------|
| 1 | QC pastes the spec content into Claude |
| 2 | Skill auto-analyzes: user flow, business rules, edge cases, security |
| 3 | Receives a complete test case suite instantly (Markdown / JSON / CSV) |
| 4 | QC reviews quickly and makes minor edits if needed |

**Time:** 15–30 minutes / feature (mostly spent on reviewing the output)

**Benefits:**
- Covers all 4 types: Happy Path, Negative, Edge Case, Security
- Consistent, standardized format across the entire team
- Auto-generated Traceability Matrix — shows which rules have no TC yet
- Nothing missed thanks to an 8-category Edge Case Checklist

---

## Tools / AI Used

| Tool | Role |
|------|------|
| **Claude (Anthropic)** | Core AI model — reads spec, reasons through logic, generates test cases |
| **Claude Code** | Environment for writing and testing SKILL.md and SPEC.md |
| **Markdown** | Format for both spec input and test case output |
| **Git / GitHub** | Version control and Pull Request review workflow |

---

## Limitations

- Vague spec with missing user flow or business rules → skill returns `WARN-001` and stops without generating output
- Handles **1 feature per run** — multi-feature specs must be split first
- Does not generate automation scripts (Selenium, Playwright, Postman)
- No direct integration with Jira, TestRail, or any bug tracker
- Does not execute tests — only generates test cases for QC to run manually
- Output quality depends on spec quality: garbage in → garbage out

---

## Expansion Roadmap

| Priority | Feature |
|----------|---------|
| 🔴 High | Detect ambiguity in the spec and suggest improvements before generating TCs |
| 🔴 High | Direct export to Jira / TestRail via API |
| 🟡 Medium | Generate automation test skeletons (Playwright / Pytest) from existing TCs |
| 🟡 Medium | Batch processing for multiple features at once |
| 🟢 Low | Auto-generate test execution report after QC fills in results |
| 🟢 Low | Support specs written in multiple languages (EN / VI / JP) |
