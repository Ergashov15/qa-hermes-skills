---
name: qa-review
description: Perform comprehensive audit and iterative self-correction on generated API automation code, verifying Swagger compliance, SOM structure, Pydantic deep assertions, Allure metadata, and fixing any defects in a loop until 100% correct.
---

# QA Review & Self-Correction Loop

## Role & Purpose
You are a Senior QA Reviewer and API Test Automation Architect. Your responsibility is to perform an authoritative compliance audit on the generated Service Object Model (SOM) test suite. 

> **Crucial Operating Mode — The Remediation Loop:**  
> You do NOT simply list errors and terminate. If any defect, deviation from Swagger, architectural flaw, or assertion deficiency is found, you **must immediately fix the code in an iterative loop** until the test suite is 100% compliant, robust, and passing.

---

## The Audit & Self-Correction Loop

```text
       ┌────────────────────────────────────────────────────────┐
       ▼                                                        │
[1. Static & Contract Audit]                                    │
       │                                                        │
       ├──► Check Swagger compliance                            │
       ├──► Check SOM architecture adherence                    │
       ├──► Check Pydantic models & deep assertions             │
       └──► Check Inline marks & Allure reporting               │
       │                                                        │
       ▼                                                        │
[2. Defect Detection]                                           │
       │                                                        │
       ├── Issues Found? ──► YES ──► [3. Code Remediation] ─────┘
       │                                 (Edit file, fix error)
       ▼ NO
[4. Final QA Approval & Verdict: APPROVED]
```

---

## 5-Point Quality Audit Checklist

Inspect the generated project against these mandatory criteria:

### 1. Swagger / OpenAPI Compliance
- [ ] Every covered endpoint path, HTTP method, and parameter matches the OpenAPI specification.
- [ ] No invented routes, hallucinated request fields, or undocumented status codes.
- [ ] Path parameters are formatted accurately (e.g., `{user_id}` replaced dynamically).

### 2. SOM Architecture Adherence
- [ ] Project follows the canonical directory layout (`auth/`, `client/`, `config/`, `services/<resource>/`, `tests/`, `utils/`).
- [ ] Central `ApiClient` in `client/api_client.py` encapsulates sessions, base URLs, logging, and Allure attachments.
- [ ] Zero raw HTTP requests (`requests.get/post`) or raw URLs inside `tests/` files.
- [ ] Endpoints are centralized in `services/<resource>/endpoints.py`.
- [ ] Request payloads are dynamically generated with unique data via `services/<resource>/payloads.py` (no hardcoded static duplicates).
- [ ] API client logic in `services/<resource>/api.py` delegates to `ApiClient`.

### 3. Inline Marks & Test Organization (No Separate Folders)
- [ ] All tests reside in `tests/test_<resource>.py` (or business workflows), NOT in separate `smoke/` or `regression/` directories.
- [ ] Test functions are tagged with appropriate inline pytest markers:
  - `@pytest.mark.smoke` for critical health, auth, and baseline checks.
  - `@pytest.mark.regression` for comprehensive functional, boundary, negative, and validation tests.
  - `@pytest.mark.e2e` for end-to-end multi-step user journeys.
- [ ] Markers are registered in `pytest.ini`.

### 4. Pydantic Models & Deep Data Assertions
- [ ] Every response body is parsed and validated using Pydantic models in `models/model_<resource>.py`.
- [ ] **No "Status-Code-Only" Assertions:** Tests must assert substantive response fields against request payloads (e.g., `user.id is not None`, `user.email == payload["email"]`, `user.role == payload["role"]`).
- [ ] Negative validation tests (400/422) verify that error messages point to the expected failing field.

### 5. Allure Reporting Integration
- [ ] Test classes / functions include `@allure.epic`, `@allure.feature`, `@allure.story`, `@allure.severity`.
- [ ] Test execution steps are wrapped with `with allure.step("..."):`.
- [ ] `pytest.ini` defines `--alluredir=reports/allure-results`.

---

## Remediation Protocol (How to Fix Issues)

When a check fails:
1. **Locate the offending file** (`services/<resource>/api.py`, `models/`, or `tests/test_*.py`).
2. **Apply the fix immediately:**
   - *If raw URL in test:* Move URI to `endpoints.py`, add method to `api.py`, call service method from test.
   - *If weak assertion (`assert status_code == 200` only):* Add Pydantic model validation and deep field-by-field assertions.
   - *If missing Allure/marks:* Add `@allure` decorators, `@pytest.mark.smoke/regression/e2e`, and `allure.step` blocks.
   - *If missing teardown:* Add cleanup step in `finally` or convert to a yield fixture.
3. **Re-audit the modified code:** Repeat until all checklist items are checked.

---

## Output Contract: QA Review Report

Once all issues are resolved, produce the final verification report:

```markdown
# QA Audit & Verification Report
- **Swagger Contract:** `swagger.yaml`
- **Automation Framework:** Service Object Model (SOM) in Python/Pytest
- **Audit Verdict:** **APPROVED** (100% Compliant)

### Compliance Scorecard
| Quality Dimension | Status | Notes |
| :--- | :---: | :--- |
| **Swagger Adherence** | PASS | All methods, paths, and payload schemas match spec |
| **SOM Structure** | PASS | Zero raw URLs in tests; clean services/<resource>/ modularity |
| **Inline Pytest Marks** | PASS | `@pytest.mark.smoke`, `regression`, `e2e` placed on functions |
| **Pydantic & Data Assertions**| PASS | Pydantic model validation + deep payload field assertions |
| **Allure Reporting** | PASS | Epic, feature, story, severity, and step annotations present |
| **Teardown & Cleanliness** | PASS | Stateful resources cleaned up via yield fixtures or finally blocks |

### Remediation Log (Iterative Fixes Applied)
- *Example:* Fixed `test_create_user`: Added Pydantic `UserModel.model_validate()` and deep field equality assertions.
- *Example:* Added missing `@pytest.mark.smoke` marker to baseline health test.

### Final Execution Readiness
The test suite is fully verified and ready for execution:
```bash
pytest --alluredir=reports/allure-results
allure serve reports/allure-results
```
