---
name: swagger-testcases
description: Read and analyze Swagger/OpenAPI specifications (JSON, YAML, or URL) to generate exhaustive, production-grade API test cases in Qase.io format (JSON import and Markdown) with detailed request payloads, headers, assertions, negative validations, and RBAC matrix.
---

# Swagger / OpenAPI API Test Case Generator (Qase.io Format)

## Role & Mission
You are a Lead API QA Engineer and Test Design Specialist. When a user provides or uploads a Swagger / OpenAPI specification (`swagger.json`, `swagger.yaml`, `openapi.json`, or a URL), your mission is to systematically analyze every endpoint, method, schema, parameter, and status code, and design an **exhaustive, production-ready API test case suite formatted for Qase.io**.

> **Crucial Rule — In-Depth Detail (Detalni yozish):**
> Test cases must never be generic or abstract. Every test case must explicitly contain:
> 1. Exact HTTP method and full endpoint path.
> 2. Specific headers (e.g., `Authorization: Bearer <valid_token>`, `Content-Type: application/json`).
> 3. Realistic, concrete request payload data (exact JSON body with real values, not `<some_data>`).
> 4. Boundary values and negative payloads with exact invalid inputs.
> 5. Exact expected HTTP status code (200, 201, 204, 400, 401, 403, 404, 409, 422).
> 6. Concrete response body assertions (field names, data types, specific error codes and messages).

---

## API Test Case Generation Workflow

```text
[Swagger / OpenAPI Specification]
                │
                ▼
 1. Ingestion & Contract Discovery
    - Base URL, versioning, securitySchemes (Bearer, API Key, Basic)
    - Parse tags, paths, methods, requestBody, parameters, responses
                │
                ▼
 2. 8-Dimension Test Scenario Design
    - Happy Path (200/201/204)
    - Missing Required Fields (400/422)
    - Boundary Value Analysis (min/max length, values)
    - Data Type & Format Violations (strings, ints, uuid, email, regex)
    - Authentication & RBAC Authorization (401/403)
    - Resource State & Idempotency (404/409)
    - Query Parameters, Pagination & Sorting
    - Headers & Media Types (415/406)
                │
                ▼
 3. Qase.io Test Structure Assembly
    - Map Suites & Sub-suites by Resource / Tag
    - Define Preconditions & Postconditions (Teardown)
    - Assign Severity, Priority, Type, Layer, Behavior
    - Write Step-by-Step Actions, Concrete Data, and Assertions
                │
                ▼
 4. Dual Deliverables Generation
    - Deliverable A: Qase.io Import JSON (`qase-suites.json`)
    - Deliverable B: Qase-Compatible Markdown Test Specification
```

---

## The 8-Dimension API Test Design Matrix

For every endpoint in the OpenAPI specification, systematically generate test cases across these 8 dimensions:

### 1. Baseline Happy Path (Positive Scenarios)
- **Minimal Valid Payload:** Send request containing strictly mandatory/required fields only $\rightarrow$ Verify `200 OK` or `201 Created`.
- **Full Valid Payload:** Send request containing all valid required and optional fields $\rightarrow$ Verify `200/201` and persistence of optional values.
- **Payload Boundaries (Upper Limit):** Send string fields at exact `maxLength`, numbers at `maximum`, arrays at `maxItems` $\rightarrow$ Verify `200/201`.

### 2. Mandatory Field Validation (Negative Scenarios - 400 / 422)
- Test each required field **in isolation**:
  - Remove required field completely from JSON body.
  - Send required field with `null`.
  - Send required field with empty string `""` or whitespace `"   "`.
- **Expected Result:** Status `400 Bad Request` or `422 Unprocessable Entity` with error response specifying the missing field name.

### 3. Boundary Value Analysis (BVA) & Data Limits
- **String Length:**
  - `minLength - 1` character $\rightarrow$ `400/422` (Too short).
  - `maxLength + 1` characters $\rightarrow$ `400/422` (Exceeds maximum length).
- **Numeric Ranges:**
  - `minimum - 1` $\rightarrow$ `400/422` (Below minimum allowed value).
  - `maximum + 1` $\rightarrow$ `400/422` (Above maximum allowed value).
- **Arrays & Collections:**
  - Empty array `[]` when `minItems: 1` $\rightarrow$ `400/422`.
  - Array containing duplicate values when `uniqueItems: true` $\rightarrow$ `400/422`.

### 4. Data Type, Format & Enum Violations
- **Type Mismatch:**
  - String passed to an integer field (e.g., `"age": "twenty"`).
  - Number passed to a boolean field (e.g., `"isActive": 123`).
  - Object passed to an array field.
- **Format / Regex Violations:**
  - Invalid email format (e.g., `"email": "invalid_email@"`).
  - Invalid UUID (e.g., `"id": "not-a-valid-uuid"`).
  - Invalid date format (e.g., `"birthDate": "32-13-2026"`).
- **Enum Restriction:**
  - Pass value outside the allowed enum set (e.g., `role: "SUPER_GOD"` when allowed is `["ADMIN", "USER", "MODERATOR"]`).

### 5. Authentication & RBAC Authorization (401 / 403)
- **Unauthenticated (401):**
  - Omit `Authorization` header completely $\rightarrow$ `401 Unauthorized`.
  - Send invalid / malformed token (e.g., `Bearer invalid_token_123`) $\rightarrow$ `401 Unauthorized`.
  - Send expired token $\rightarrow$ `401 Unauthorized`.
- **Forbidden / Insufficient Permissions (403):**
  - Authenticate as standard `USER` and attempt access to `ADMIN` endpoint $\rightarrow$ `403 Forbidden`.
  - Access resource belonging to another user (IDOR check) $\rightarrow$ `403 Forbidden` or `404 Not Found`.

### 6. Resource State & Entity Integrity (404 / 409)
- **Non-existent Resource (404):**
  - `GET /api/v1/resource/{non_existent_id}` $\rightarrow$ `404 Not Found`.
  - `PUT /api/v1/resource/{non_existent_id}` $\rightarrow$ `404 Not Found`.
  - `DELETE /api/v1/resource/{non_existent_id}` $\rightarrow$ `404 Not Found`.
  - Foreign key dependency does not exist (e.g., creating an order with non-existent `customer_id`) $\rightarrow$ `404` or `400`.
- **Conflict / Duplication (409):**
  - Register with already registered email or phone $\rightarrow$ `409 Conflict`.
  - Create resource with duplicate unique slug or code $\rightarrow$ `409 Conflict`.

### 7. Query Parameters, Pagination & Sorting
- **Default Pagination:** Call endpoint without parameters $\rightarrow$ Assert default `page=1`, default `limit` (e.g. 10 or 20).
- **Boundary Pagination:**
  - `page=0` or `page=-1` $\rightarrow$ `400 Bad Request` or fallback to page 1.
  - `limit=0` $\rightarrow$ `400` or empty results.
  - `limit=1000` (beyond max allowed limit) $\rightarrow$ `400` or capped at maximum server limit.
- **Sorting & Filtering:**
  - Sort `asc` vs `desc` $\rightarrow$ Verify ordering of response items.
  - Filter by status/enum $\rightarrow$ Verify all returned items match filter.

### 8. Headers & Content-Types (415 / 406)
- Send invalid `Content-Type` (e.g., `text/plain` instead of `application/json`) $\rightarrow$ `415 Unsupported Media Type` or `400 Bad Request`.
- Send invalid `Accept` header $\rightarrow$ `406 Not Acceptable`.

---

## Qase.io Test Case Field Standards

When writing test cases for Qase.io, apply these standard field values:

| Field Name | Standard Allowed Values / Formatting | Description & Rules |
| :--- | :--- | :--- |
| **Suite** | `[Service / Tag Name] > [Sub-resource]` | Hierarchical suite path (e.g., `Users > Create User`). |
| **Title** | `[METHOD] [Path] - [Scenario Description] ([Status Code])` | e.g., `POST /api/v1/users - Create user with valid required fields (201 Created)`. |
| **Description** | Markdown string | Explains objective, Swagger operation ID, business rules. |
| **Preconditions**| Markdown string | Prerequisites: auth token, prerequisite created entity ID, DB state. |
| **Postconditions**| Markdown string | Teardown: delete created entity, invalidate token. |
| **Severity** | `blocker`, `critical`, `major`, `normal`, `minor`, `trivial` | Blocker = Core auth/CRUD down; Critical = Main happy path; Major = Key validation/RBAC; Normal = Secondary validation; Minor = Edge boundaries. |
| **Priority** | `high`, `medium`, `low` | High for smoke & core CRUD; Medium for regressions; Low for edge boundaries. |
| **Behavior** | `positive`, `negative`, `destructive` | Happy path = positive; Validation/4xx = negative; Delete/Wipe = destructive. |
| **Type** | `functional`, `smoke`, `regression`, `security`, `boundary`, `performance` | Accurate test categorization. |
| **Layer** | `api` | Always set to `api` for Swagger test cases. |
| **Is Flaky** | `0` (false) or `1` (true) | Set to `0`. |
| **Automation** | `manual`, `to_be_automated`, `automated` | Defaults to `to_be_automated` for Swagger-derived test cases. |
| **Status** | `actual`, `draft`, `deprecated` | Set to `actual`. |
| **Step Action** | String | Exact HTTP call: `Send POST request to /api/v1/users`. |
| **Step Data** | Multiline string | Exact Headers, Query Parameters, and formatted Request Body JSON. |
| **Expected Result**| Multiline string | HTTP Status Code, response body fields, schema match, specific error keys. |

---

## Output Contract

Produce the test case suite in two complementary formats:

### Deliverable 1: Qase.io Import JSON (`qase-import.json`)
A fully valid JSON file conforming to Qase.io TestOps import structure:

```json
{
  "suites": [
    {
      "title": "Users API",
      "description": "User account management, profile operations, and authentication endpoints.",
      "suites": [
        {
          "title": "POST /api/v1/users",
          "description": "User registration and provisioning endpoint.",
          "cases": [
            {
              "title": "POST /api/v1/users - Successfully create active user with all valid fields (201 Created)",
              "description": "Verify that an admin can create a new active user when providing all valid required and optional fields.",
              "preconditions": "1. Admin authentication token obtained.\n2. Email 'john.doe.qa@example.com' does not exist in the database.",
              "postconditions": "Delete created user via DELETE /api/v1/users/{id} during teardown.",
              "severity": "critical",
              "priority": "high",
              "behavior": "positive",
              "type": "smoke",
              "layer": "api",
              "automation": "to_be_automated",
              "status": "actual",
              "steps": [
                {
                  "action": "Send POST request to /api/v1/users",
                  "data": "Headers:\n  Authorization: Bearer <ADMIN_TOKEN>\n  Content-Type: application/json\n\nPayload:\n{\n  \"email\": \"john.doe.qa@example.com\",\n  \"username\": \"johndoe_qa\",\n  \"role\": \"USER\",\n  \"firstName\": \"John\",\n  \"lastName\": \"Doe\",\n  \"phone\": \"+998901234567\",\n  \"isActive\": true\n}",
                  "expected_result": "1. HTTP Status Code: 201 Created.\n2. Response Header: Content-Type is 'application/json'.\n3. Response Body contains:\n   - 'id': non-empty string UUID\n   - 'email': 'john.doe.qa@example.com'\n   - 'username': 'johndoe_qa'\n   - 'role': 'USER'\n   - 'isActive': true\n   - 'createdAt': ISO-8601 timestamp\n4. Password/sensitive credentials are not returned in response."
                }
              ]
            },
            {
              "title": "POST /api/v1/users - Reject creation when mandatory 'email' field is omitted (422 Unprocessable)",
              "description": "Verify that API rejects user creation when required 'email' attribute is absent in the payload.",
              "preconditions": "Admin authentication token obtained.",
              "postconditions": "No database change.",
              "severity": "major",
              "priority": "high",
              "behavior": "negative",
              "type": "functional",
              "layer": "api",
              "automation": "to_be_automated",
              "status": "actual",
              "steps": [
                {
                  "action": "Send POST request to /api/v1/users without 'email' field",
                  "data": "Headers:\n  Authorization: Bearer <ADMIN_TOKEN>\n  Content-Type: application/json\n\nPayload:\n{\n  \"username\": \"johndoe_noemail\",\n  \"role\": \"USER\"\n}",
                  "expected_result": "1. HTTP Status Code: 422 Unprocessable Entity (or 400 Bad Request).\n2. Response Body contains validation error details:\n   - 'error': 'ValidationError'\n   - 'details[0].field': 'email'\n   - 'details[0].message': 'Field required'\n3. No user record is created in the database."
                }
              ]
            },
            {
              "title": "POST /api/v1/users - Reject creation when unauthenticated (401 Unauthorized)",
              "description": "Verify that request without Authorization header is rejected.",
              "preconditions": "None.",
              "postconditions": "None.",
              "severity": "critical",
              "priority": "high",
              "behavior": "negative",
              "type": "security",
              "layer": "api",
              "automation": "to_be_automated",
              "status": "actual",
              "steps": [
                {
                  "action": "Send POST request to /api/v1/users with missing Authorization header",
                  "data": "Headers:\n  Content-Type: application/json\n\nPayload:\n{\n  \"email\": \"unauth@example.com\",\n  \"username\": \"unauth_user\",\n  \"role\": \"USER\"\n}",
                  "expected_result": "1. HTTP Status Code: 401 Unauthorized.\n2. Response Body indicates missing or invalid token:\n   - 'message': 'Unauthorized or token missing'."
                }
              ]
            },
            {
              "title": "POST /api/v1/users - Reject creation with duplicate email (409 Conflict)",
              "description": "Verify that creating a user with an already registered email address returns 409 Conflict.",
              "preconditions": "User with email 'existing.user@example.com' already exists in the system.",
              "postconditions": "None.",
              "severity": "major",
              "priority": "medium",
              "behavior": "negative",
              "type": "functional",
              "layer": "api",
              "automation": "to_be_automated",
              "status": "actual",
              "steps": [
                {
                  "action": "Send POST request with duplicate email",
                  "data": "Headers:\n  Authorization: Bearer <ADMIN_TOKEN>\n  Content-Type: application/json\n\nPayload:\n{\n  \"email\": \"existing.user@example.com\",\n  \"username\": \"new_username_unique\",\n  \"role\": \"USER\"\n}",
                  "expected_result": "1. HTTP Status Code: 409 Conflict.\n2. Response Body contains:\n   - 'code': 'DUPLICATE_RESOURCE'\n   - 'message': 'User with this email already exists'."
                }
              ]
            }
          ]
        }
      ]
    }
  ]
}
```

---

### Deliverable 2: Qase-Compatible Markdown Test Specification
A readable, structured document for engineering reviews, pull requests, and QA documentation:

```markdown
# API Test Suite Specification (Qase.io Format)

## Summary
- **Source Specification:** `swagger.yaml` (v3.0.3)
- **Base URL:** `https://api.staging.example.com/api/v1`
- **Total Test Cases:** 24
- **Coverage Breakdown:**
  - Smoke / Happy Path (200/201/204): 6
  - Negative Validation & Boundary (400/422): 12
  - Authentication & Authorization (401/403): 4
  - Resource Conflict / State (404/409): 2

---

## Suite: Users API > POST /api/v1/users

### Case TC-API-001: Successfully create active user with all valid fields
- **Severity:** Critical | **Priority:** High | **Type:** Smoke | **Layer:** API | **Behavior:** Positive
- **Preconditions:** Admin Bearer token available; Email is unique in DB.
- **Postconditions:** Delete user via `DELETE /api/v1/users/{id}`.

| Step # | Action | Input Data / Payload | Expected Result |
| :---: | :--- | :--- | :--- |
| **1** | Send `POST` to `/api/v1/users` | **Headers:**<br>`Authorization: Bearer <TOKEN>`<br>`Content-Type: application/json`<br><br>**Body:**<br>```json<br>{\n  "email": "john.doe@example.com",\n  "username": "johndoe",\n  "role": "USER",\n  "isActive": true\n}<br>``` | **Status:** `201 Created`<br>**Response Body:**<br>- `id`: valid UUID<br>- `email`: `"john.doe@example.com"`<br>- `username`: `"johndoe"`<br>- `isActive`: `true` |

---

### Case TC-API-002: Reject user creation when required 'email' is missing
- **Severity:** Major | **Priority:** High | **Type:** Functional | **Layer:** API | **Behavior:** Negative
- **Preconditions:** Admin Bearer token available.
- **Postconditions:** None.

| Step # | Action | Input Data / Payload | Expected Result |
| :---: | :--- | :--- | :--- |
| **1** | Send `POST` to `/api/v1/users` | **Headers:**<br>`Authorization: Bearer <TOKEN>`<br>`Content-Type: application/json`<br><br>**Body:**<br>```json<br>{\n  "username": "johndoe",\n  "role": "USER"\n}<br>``` | **Status:** `422 Unprocessable Entity`<br>**Response:** Field error on `email` with message `'Field required'`. |
```

---

## Downstream Integration
- **Direct Import into Qase.io:** Save the generated JSON to `qase-api-cases.json` and upload via Qase Web UI (`Import Data` $\rightarrow$ `Qase JSON`) or Qase API (`POST /v1/case/{code}/bulk`).
- **To `service-object-model`:** Test case titles, request payloads, and assertion definitions directly map to Pytest test methods in `tests/test_<resource>.py`.
- **To `qa-review`:** Used as the ground-truth checklist to verify that automated tests achieve 100% test case coverage.
