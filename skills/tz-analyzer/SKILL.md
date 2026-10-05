---
name: tz-analyzer
description: Read and analyze project documentation (TZ, PRD, SRS, BRD, user stories) to detect requirement defects, ambiguities, contradictions, omissions, and edge-case gaps as early as possible (Shift-Left QA) before development or testing begins.
---

# Technical Specification (TZ) & Requirements Analyzer

## Core Mission: Shift-Left Early Defect Detection
The primary objective of this skill is **early defect detection (kamchiliklarni ertaroq aniqlash)**. 

In the software development lifecycle, fixing a requirement bug discovered in production or during late testing is **10x to 100x more costly** than catching it at the specification stage. When a user provides or uploads a project document (ТЗ / Texnik Topshiriq, PRD, SRS, User Stories, or Architecture RFC), this skill guides the agent to read it deeply, scrutinize every assumption, and uncover hidden risks, missing edge cases, and contradictions **before developers write code or QA writes tests**.

---

## Role & Mindset
You are a Lead QA Requirements Analyst and Specification Auditor. 
- You do **not** take requirements at face value.
- You actively search for what the author **forgot to mention**, what is **vague**, what is **conflicting**, and how the system might fail under stress or negative conditions.
- You transform unrefined documentation into a clear, testable, and unambiguous baseline for development.

---

## The 32-Category Requirement Defect Taxonomy

Systematically audit the document against all 32 categories:

### A. Clarity, Consistency & Feasibility
| # | Defect Category | Definition & Detection Trigger |
| :- | :--- | :--- |
| **1** | **Missing Requirement** | Essential functionality or behavior is completely omitted (e.g., password reset flow absent in auth spec). |
| **2** | **Ambiguous Requirement** | Vague phrases subject to multiple interpretations (e.g., *"system should be fast"*, *"user-friendly"*, *"handle appropriately"*). |
| **3** | **Contradictory Requirement** | Two statements conflict (e.g., Section 2 says *"Order is immutable"*, Section 5 says *"User can edit shipping address until dispatched"*). |
| **4** | **Incomplete Requirement** | A scenario is started but lacks resolution (e.g., *"If payment fails, notify user"*, but does not state order status or retry policy). |
| **5** | **Untestable Requirement** | Lacks objective pass/fail criteria; impossible to verify empirically through automation or manual testing. |
| **6** | **Non-measurable Requirement** | Lacks quantitative metrics (e.g., *"high throughput"*, *"minimal latency"* without explicit numbers). |
| **7** | **Invalid Requirement** | Technically impossible, incorrect, or violating domain protocols/standards (e.g., *"store cleartext credit card numbers"*). |
| **8** | **Unrealistic Requirement** | Unachievable within existing technology, infrastructure, or operational constraints (e.g., *"zero millisecond worldwide latency"*). |

### B. Business Logic & Data Constraints
| # | Defect Category | Definition & Detection Trigger |
| :- | :--- | :--- |
| **9** | **Business Rule Gap** | Omission of core operational logic (e.g., discount calculation formula, cancellation refund percentage, bonus point expiration). |
| **10** | **Boundary Gap** | Missing minimum, maximum, length, or boundary values (e.g., username length limits, max order items, transfer limits). |
| **11** | **Validation Gap** | Unspecified format constraints, regex patterns, character sets, allowed symbols, or mandatory types. |
| **12** | **Negative Scenario Gap** | Only happy paths described; behavior for invalid inputs, expired data, or rejected states is missing. |
| **13** | **Error Handling Gap** | Undefined HTTP status codes, missing user-facing error copy, or absence of machine-readable error codes. |
| **14** | **Permission / Authorization Gap** | Unclear Role-Based Access Control (RBAC): what `ADMIN`, `MANAGER`, `CUSTOMER`, or `GUEST` can vs. cannot execute. |
| **15** | **State Transition Gap** | Undefined state machine rules (e.g., allowed transitions between `DRAFT`, `PENDING`, `APPROVED`, `REJECTED`, `CANCELLED`). |
| **16** | **Data Definition Gap** | Missing data types, field names, precision (e.g., decimal places for currency), or structural definitions. |
| **17** | **Default Value Gap** | Undefined fallback value when an optional field or configuration is omitted. |
| **18** | **Null / Empty Value Gap** | Unclear whether fields accept `null`, empty string `""`, empty array `[]`, or require explicit deletion. |
| **19** | **Duplicate Data Gap** | Behavior undefined when duplicate entries occur (e.g., registering duplicate email, repeating SKU, re-submitting transaction). |

### C. System, Integration & Non-Functional
| # | Defect Category | Definition & Detection Trigger |
| :- | :--- | :--- |
| **20** | **API Contract Gap** | Request/Response payloads, headers, query parameters, or content-types are unspecified or inconsistent. |
| **21** | **Integration Gap** | Undefined protocols, payloads, authentication, or failure fallback for third-party systems (payment gateways, CRM, SMS). |
| **22** | **Dependency Gap** | Unclear operational sequence or prerequisite services required before an action can execute. |
| **23** | **Concurrency Gap** | Undefined behavior for simultaneous requests (race conditions, double-spending, inventory overdrafts). |
| **24** | **Timeout & Retry Gap** | Missing connection/read timeout values and exponential backoff retry policies for network or service interruptions. |
| **25** | **Performance Requirement Gap** | Missing SLAs: Response time (p95/p99), RPS (requests per second), concurrent user targets, database query limits. |
| **26** | **Security Requirement Gap** | Unspecified encryption at rest/transit, token lifetime, rate limiting, hashing algorithms, or sensitive data masking. |
| **27** | **Compatibility Gap** | Undefined client support (supported browsers, OS, mobile screen resolutions, backward API versioning). |
| **28** | **Acceptance Criteria Gap** | Missing concrete Definition of Done (DoD) or Given/When/Then scenarios for story sign-off. |
| **29** | **Logging & Audit Gap** | Undefined audit trail requirements (which user actions, IP addresses, timestamps, and changes must be recorded for compliance). |
| **30** | **Notification Gap** | Undefined triggers, recipient lists, delivery channels (SMS, Email, Push, Webhook), and localization for notifications. |
| **31** | **Data Retention & Cleanup Gap** | Missing rules on data lifecycle, archiving policies, GDPR data erasure, or temporary file cleanup. |
| **32** | **Localization & i18n Gap** | Undefined multi-language support, currency formatting, time zone handling (UTC vs local), and date formats. |

---

## Early Detection Execution Workflow

```text
[User provides TZ / PRD / Document]
                  │
                  ▼
 1. Full Ingestion & Scope Parsing
    - Read entire document without skimming
    - Map actors, modules, workflows, and external dependencies
                  │
                  ▼
 2. Systematic 32-Taxonomy Scan
    - Scrutinize every requirement statement against categories 1 to 32
    - Flag contradictions across different sections
    - Identify missing negative & boundary edge cases
                  │
                  ▼
 3. Severity & Impact Classification
    - Rate as Blocker, Critical, Major, or Minor
    - Detail exact developer and QA consequences if not fixed
                  │
                  ▼
 4. Generate Stakeholder Clarification Questions
    - Formulate concrete, copy-paste-ready questions for PM / BA / Architect
                  │
                  ▼
 5. Propose Industry Standard Technical Solutions
    - Suggest concrete enums, state machine flows, schemas, or error formats
                  │
                  ▼
[Deliver Comprehensive Early Audit Report]
```

---

## Defect Severity Classification

- **Blocker:** Architecture or development cannot start safely without resolving this (e.g., contradictory business flow, missing core calculation formula).
- **Critical:** High risk of data loss, financial discrepancy, security breach, or untestable feature (e.g., concurrency race condition, missing auth check).
- **Major:** Functional ambiguity leading to developer misunderstanding or testing discrepancies (e.g., missing validation limits, undefined error status).
- **Minor:** Non-blocking omission or cosmetic inconsistency with reasonable obvious default.

---

## Output Contract: Early Requirements Audit Report

Produce the analysis using this structured template:

```markdown
# Early Requirements Quality Audit Report (Shift-Left QA)
- **Document Analyzed:** [Document Title / Version / Path]
- **Auditor:** Lead QA Requirements Analyst
- **Requirements Health Score:** [0% - 100%]
- **Specification Status:** [APPROVED FOR DEV / ACTION REQUIRED / BLOCKED]

### Executive Summary
Concise assessment of specification maturity, high-risk operational blindspots, and key areas needing clarification before code implementation begins.

---

### Defect & Gap Findings Matrix
| ID | Taxonomy Category | Section / Location | Finding & Ambiguity Description | Severity | Impact on QA / Dev |
| :--- | :--- | :--- | :--- | :---: | :--- |
| `GAP-001` | **Contradictory Requirement** | Sec 3.2 vs Sec 7.1 | Sec 3.2 states orders cannot be edited once placed; Sec 7.1 allows updating item quantity before dispatch. | **Blocker** | Developers cannot implement order state machine; QA cannot assert immutability. |
| `GAP-002` | **Boundary Gap** | Sec 4.1 "User Registration" | Phone number field has no min/max length or country code specification. | **Major** | DB schema sizing unknown; test cases cannot determine boundary limits. |
| `GAP-003` | **Concurrency Gap** | Sec 6.3 "Promo Code Usage" | No constraint defined for simultaneous redemption of a single-use coupon from multiple sessions. | **Critical** | Risk of coupon reuse / financial loss via race conditions. |
| `GAP-004` | **Error Handling Gap** | Sec 5.2 "Payment Processing" | Document only says "show error". No HTTP code, error key, or retry advice specified. | **Major** | Frontend/API contracts will diverge; automated error assertion impossible. |

---

### Clarification Questions for Stakeholders (Ready for PM / BA)
Clear, numbered questions to eliminate all ambiguities before development:
1. *Regarding GAP-001 (Order Editing):* Can a customer modify order items after placement? If yes, up to which state (`PENDING` vs `PROCESSING`)?
2. *Regarding GAP-002 (Phone Format):* Should phone numbers strictly follow E.164 format (`+998XXXXXXXXX`), and is country code mandatory?
3. *Regarding GAP-003 (Concurrency):* Should promo code verification employ distributed database locking (pessimistic lock) to prevent race condition reuse?

---

### Recommended Technical Solutions
Concrete recommendations for detected gaps (proposed schemas, state machines, error formats):
- **For GAP-001:** Enforce state machine transition: `DRAFT -> SUBMITTED (immutable) -> PROCESSING -> COMPLETED`.
- **For GAP-004:** Adopt standard RFC 7807 problem details: `{ "type": "payment_failed", "status": 402, "detail": "Insufficient funds" }`.
```

---

## Downstream Value
- **To `swagger-analysis`:** Supplies clarified boundaries, data types, and status codes to ensure OpenAPI contracts are complete.
- **To `service-object-model`:** Prevents building automation on faulty assumptions; directly fuels negative scenarios, boundary tests, and Pydantic validation rules.
