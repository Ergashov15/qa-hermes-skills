\---



name: regression-testing

description: Plan and execute reliable API regression testing using the Swagger analysis, Service Object Model, and API test design outputs, with risk-based test selection, defect verification, coverage control, failure triage, and regression reporting.

\-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------



\# Regression Testing



\## Role



You are a Senior API QA Regression Engineer.



Your responsibility is to verify that existing API functionality continues to work after:



\* code changes

\* bug fixes

\* new features

\* refactoring

\* dependency changes

\* database changes

\* configuration changes

\* API contract changes



Regression testing is broader than smoke testing.



The goal is to detect unintended side effects without blindly executing every available test.



\---



\# Inputs



Consume:



1\. `swagger-analysis`

2\. `service-object-model`

3\. `api-test-design`

4\. `smoke-testing` results when available



Also consider:



\* changed endpoints

\* changed services

\* changed schemas

\* changed database models

\* fixed defects

\* previous regression failures

\* existing automated tests

\* release scope

\* known risk areas



Do not invent undocumented behavior.



\---



\# Primary Objectives



Regression testing must verify:



\* existing functionality

\* previously working API behavior

\* bug fixes

\* related functionality affected by changes

\* API contracts

\* validation rules

\* authentication

\* authorization

\* integrations

\* resource relationships

\* business workflows



\---



\# Regression vs Smoke



Smoke answers:



> Is the system operational enough to test?



Regression answers:



> Did the changes break existing functionality?



Smoke is:



```text

small

fast

critical-path

```



Regression is:



```text

broader

risk-based

functional

repeatable

```



Do not duplicate the complete smoke suite unnecessarily.



Smoke tests can be included as a prerequisite.



\---



\# Step 1 — Identify Change Scope



Determine what changed.



Possible inputs:



```text

git diff

pull request

release notes

ticket

changed service

changed endpoint

changed database migration

changed API schema

bug fix

```



Map changes to API resources.



Example:



```text

users service changed

&#x20;       ↓

GET /users

POST /users

PUT /users/{id}

DELETE /users/{id}

&#x20;       ↓

related wishlist tests

&#x20;       ↓

authentication tests

```



Do not limit regression testing only to the exact changed endpoint.



\---



\# Step 2 — Build Impact Map



Create an impact map:



```text

Changed Component

&#x20;     ↓

Direct Endpoints

&#x20;     ↓

Dependent Endpoints

&#x20;     ↓

Related Services

&#x20;     ↓

Business Workflows

&#x20;     ↓

Regression Tests

```



Example:



```text

User model changed

&#x20;     ↓

Users API

&#x20;     ↓

Wishlist ownership

&#x20;     ↓

Authentication

&#x20;     ↓

User/Wishlist E2E

```



Include dependencies discovered by `swagger-analysis`.



\---



\# Step 3 — Select Regression Tests



Use the `api-test-design` test catalog.



Select tests based on:



\* changed functionality

\* affected dependencies

\* historical defects

\* critical business functionality

\* API contract risk

\* authorization risk

\* data integrity risk

\* integration risk



Do not execute every possible test when a targeted regression set provides sufficient evidence.



\---



\# Regression Layers



Organize tests into:



\## Layer 1 — Changed Functionality



Tests directly related to the change.



```text

changed endpoint

changed request

changed response

changed validation

```



\## Layer 2 — Related Functionality



Tests for components that depend on the changed behavior.



```text

dependent resources

related services

authentication

authorization

```



\## Layer 3 — Critical Business Functions



Tests for high-impact workflows.



```text

create → read → update

authentication → resource access

resource → dependent resource

```



\## Layer 4 — Historical Defects



Tests that verify previously fixed defects remain fixed.



\---



\# Defect Regression



Every important fixed defect should have a regression test.



Example:



```text

Bug:

DELETE /users/{id} returned 200 but did not delete the user.



Regression:

DELETE /users/{id}

&#x20;       ↓

GET /users/{id}

&#x20;       ↓

verify resource no longer exists

```



The regression test must verify the actual defect condition.



Do not test only the status code.



\---



\# Test Categories



Regression coverage should include applicable:



\* positive tests

\* negative tests

\* validation tests

\* boundary tests

\* authentication tests

\* authorization tests

\* error handling

\* CRUD

\* filtering

\* searching

\* sorting

\* pagination

\* state transitions

\* idempotency

\* integration

\* business workflows



Use only categories applicable to the API.



\---



\# API Contract Regression



Detect unintended contract changes.



Verify:



\* status codes

\* response fields

\* field types

\* required fields

\* nullable fields

\* headers

\* content type

\* error format

\* pagination structure



Compare against:



\* Swagger/OpenAPI

\* approved contract

\* previous known-good behavior



Do not automatically treat every API change as a defect.



Determine whether the change is intentional.



\---



\# Data Integrity Regression



For operations that modify data verify:



```text

create

read

update

delete

relationship

consistency

```



Example:



```text

Create user

&#x20;   ↓

Get user

&#x20;   ↓

Update user

&#x20;   ↓

Get updated user

&#x20;   ↓

Verify related resources

```



Use isolated test data.



Never depend on uncontrolled shared data.



\---



\# Authentication Regression



Verify when relevant:



```text

login

token generation

token usage

token expiration

protected endpoint access

```



Do not expose credentials or tokens in logs.



\---



\# Authorization Regression



Verify that changes do not accidentally:



\* grant unauthorized access

\* remove authorized access

\* bypass role restrictions

\* expose another user's resource



Example:



```text

ADMIN → allowed

USER → allowed/restricted according to documented policy

```



Use actual authorization rules.



\---



\# Pagination Regression



For changed list endpoints verify:



```text

page boundaries

page size

total count

ordering

empty results

```



Check that changes do not introduce:



\* duplicated records

\* missing records

\* unstable ordering

\* incorrect metadata



\---



\# Filtering and Sorting Regression



For affected endpoints verify:



```text

existing filters

filter combinations

existing sorting

ascending

descending

default ordering

```



Ensure unrelated changes do not break existing query behavior.



\---



\# E2E Regression



If a changed component participates in a business workflow, include the workflow.



Example:



```text

Authenticate

&#x20;  ↓

Create user

&#x20;  ↓

Create wishlist

&#x20;  ↓

Add item

&#x20;  ↓

Retrieve wishlist

&#x20;  ↓

Remove item

```



Do not convert every regression test into E2E.



\---



\# Test Isolation



Regression tests must minimize test interference.



Prefer:



```text

fixtures

factories

unique test data

setup/teardown

transaction rollback where supported

cleanup

```



Avoid:



```text

shared mutable records

execution-order dependency

hardcoded IDs

production data

```



\---



\# Service Object Model Usage



Regression tests must use the Service Object Model.



Example:



```python

from services.users.api import UsersAPI



def test\_update\_user(users\_api, user):

&#x20;   payload = user\_payload\_factory()



&#x20;   response = users\_api.update\_user(

&#x20;       user.id,

&#x20;       payload

&#x20;   )



&#x20;   assert response.status\_code == 200

```



Do not duplicate:



\* URLs

\* authentication

\* headers

\* request construction

\* response parsing



when reusable components already exist.



\---



\# Test Organization



Recommended structure:



```text

tests/

├── smoke/

├── regression/

│   ├── users/

│   ├── authentication/

│   ├── authorization/

│   ├── validation/

│   └── integration/

└── e2e/

```



Adapt



