# 🧠 AI Skill – Test Case Generator

## 1. Project Overview

**Skill Test Case Generator** is a project aimed at building an AI Skill that helps QC/Testers automatically generate test cases from product spec files.

The goals of this project are:

* Reduce time spent writing test cases manually
* Increase coverage (happy path, edge case, negative case)
* Standardize test case format across QC team members
* Help QC focus on reviewing rather than writing from scratch

This project was built in the context of the **Cook A Skill – QC Team** competition, where each individual creates an AI Skill to serve real-world QC workflows.

---

## 2. Problem Statement

In the current QC process:

* QC must read specs manually
* Think through test cases step by step on their own
* Edge cases are easily missed
* Each person writes in a different format
* Complex modules take a lot of time

➡️ An AI Skill is needed that can:

* Read `.md` spec files
* Analyze logic
* Generate test cases in a standardized format
* Suggest edge cases and test data

---

## 3. Target Users

* QC / Manual Testers
* QA Leads who need to review test cases
* Product teams who want to verify spec coverage

Main use cases:

* Quickly write test cases for new features
* Review specs to find missing logic
* Standardize test cases before testing

---

## 4. Expected Workflow

Desired workflow:

1. QC provides the feature spec file (markdown)
2. AI reads the spec following the Skill's instructions
3. AI analyzes:

   * User flow
   * Business logic
   * Validation rules
   * Edge cases
4. AI generates a list of test cases
5. QC reviews and edits if needed

Output must be ready to use for testing immediately.

---

## 5. Input & Output

### Input

* Feature spec file (`.md`)
* Or feature description content

Spec can be in the form of:

* PRD
* BRS
* Feature description
* User story + acceptance criteria

### Output

A list of test cases including:

* ID
* Title
* Precondition
* Steps
* Expected Result
* Priority
* Test Type (Happy / Negative / Edge)

Can be exported to Markdown / CSV / Excel.

---

## 6. Scope of Skill

### In Scope

* Generate functional test cases
* Detect basic edge cases
* Generate test data suggestions
* Standardized output format

### Out of Scope

* Automation scripts
* Performance testing scripts
* Security penetration testing
* Integration with bug tracking tools

---

## 7. Quality Expectations

The Skill must ensure:

* Full coverage:

  * Happy path
  * Negative case
  * Edge case
* Clear, step-by-step test cases
* Specific expected results
* Reasonable priority
* No duplicate test cases

---

## 8. Assumptions

* Specs are written with reasonable clarity
* QC can edit the spec before generating
* AI does not replace the final QC review

---

## 9. Limitations

The Skill may face limitations when:

* Spec is too vague
* Business rules are missing
* No user flow is provided
* Feature is too complex or depends on many systems

In these cases, QC needs to refine the spec first.

---

## 10. Next Steps

From this README, the Agent will:

1. Understand the project goals
2. Write **spec.md** for each feature
3. Write **SKILL.md** to guide the AI in generating test cases

Expected repo structure:

```
repo/
  ├── README.md
  ├── spec.md
  ├── SKILL.md
  ├── skill-card.md
  └── ai-showcase/
```

---

## 11. Success Criteria

The Skill is considered successful when:

* Feed 1 real spec → generate usable test cases
* QC only needs minor edits
* Coverage is better than manual
* Format is consistent
* Live demo runs stably

---

## 12. Vision

In the future, the Skill can be extended to:

* Detect ambiguity in specs
* Generate automation test skeletons
* Create test reports automatically
* Integrate with Jira / TestRail
* Multi-language spec parsing
