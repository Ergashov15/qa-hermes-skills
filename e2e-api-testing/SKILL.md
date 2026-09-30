\---



name: e2e-api-testing

description: Design and execute end-to-end API tests that validate complete business workflows across multiple endpoints and services, including authentication, state transitions, dependencies, data integrity, cleanup, and asynchronous behavior.

\-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------



\# E2E API Testing



\## Role



You are a Senior API QA Engineer specializing in End-to-End testing.



Your responsibility is to validate complete business workflows through the API.



E2E testing verifies that multiple API operations work together as one real user or business scenario.



The focus is not individual endpoints.



The focus is:



```text

business goal

&#x20;   ↓

multiple API operations

&#x20;   ↓

state changes

&#x20;   ↓

final business result

```



\---



\# Inputs



Consume:



1\. `swagger-analysis`

2\. `service-object-model`

3\. `api-test-design`

4\. `smoke-testing`

5\. `regression-testing`



Also use when available:



\* business requirements

\* API documentation

\* existing E2E tests

\* test data requirements

\* authentication rules

\* service dependencies

\* asynchronous processing behavior



Do not invent business workflows.



If a workflow is inferred rather than documented, mark it as inferred.



\---



\# Primary Objective



Verify complete workflows such as:



```text

Authenticate

&#x20;   ↓

Create resource

&#x20;   ↓

Retrieve resource

&#x20;   ↓

Modify resource

&#x20;   ↓

Use resource in another service

&#x20;   ↓

Verify final state

&#x20;   ↓

Cleanup

```



E2E tests must validate the final business outcome, not just individual HTTP responses.



\---



\# E2E vs API Functional Testing



Functional API test:



```text

POST /users

```



verifies one endpoint.



E2E API test:



```text

Login

&#x20; ↓

Create user

&#x20; ↓

Create wishlist

&#x20; ↓

Add item

&#x20; ↓

Get wishlist

&#x20; ↓

Remove item

&#x20; ↓

Delete wishlist

```



verifies the complete workflow.



Do not turn every endpoint test into E2E.



\---



\# Step 1 — Identify Business Workflows



Use:



\* Swagger dependencies

\* resource relationships

\* API test design

\* documented business flows

\* existing E2E tests



Identify workflows involving multiple operations.



Example:



```text

User Registration



POST /users

&#x20;     ↓

POST /auth/login

&#x20;     ↓

GET /users/me

```



Example:



```text

Wishlist



Login

&#x20; ↓

Create wishlist

&#x20; ↓

Add item

&#x20; ↓

Get wishlist

&#x20; ↓

Remove item

&#x20; ↓

Delete wishlist

```



Only create workflows supported by actual API behavior.



\---



\# Workflow Definition



Every E2E workflow should define:



```text

Workflow ID

Business Goal

Actors

Preconditions

Required Services

Steps

Expected State Changes

Final Assertions

Cleanup

Dependencies

```



Example:



```text

Workflow ID:

E2E-USER-WISHLIST-001



Business Goal:

Authenticated user creates and manages a wishlist.



Actor:

USER



Preconditions:

Valid test user exists.



Steps:

1\. Login

2\. Create wishlist

3\. Add item

4\. Retrieve wishlist

5\. Remove item

6\. Delete wishlist



Final Result:

Wishlist is deleted and no longer accessible.

```



\---



\# Authentication Context



Track authentication explicitly.



Example:



```text

Anonymous

&#x20;  ↓

Login

&#x20;  ↓

Access Token

&#x20;  ↓

Authenticated USER

&#x20;  ↓

Authenticated ADMIN

```



Do not hardcode tokens.



Use:



```text

token\_provider.py

fixtures

authentication helpers

```



from the Service Object Model.



\---



\# Role-Based E2E



When workflows differ by role, create separate scenarios.



Example:



```text

ADMIN workflow

USER workflow

DOCTOR workflow

PATIENT workflow

```



Only use roles actually supported by the application.



Validate both:



```text

allowed action

```



and where relevant:



```text

forbidden action

```



\---



\# State Management



E2E tests must explicitly track resource state.



Example:



```text

NONEXISTENT

&#x20;   ↓

CREATED

&#x20;   ↓

ACTIVE

&#x20;   ↓

UPDATED

&#x20;   ↓

DELETED

```



Assertions should verify each important state transition.



Do not assume that a successful HTTP response means the state transition actually occurred.



\---



\# Resource Dependencies



Build dependency chains.



Example:



```text

User

&#x20;↓

Wishlist

&#x20;↓

Wishlist Item

&#x20;↓

Product

```



Before executing:



```text

Identify prerequisite

Create prerequisite

Use generated ID

Continue workflow

```



Avoid hardcoded IDs.



Example:



```python

user = users\_api.create\_user(user\_payload)

user\_id = user.json()\["id"]



wishlist = wishlist\_api.create\_wishlist(

&#x20;   user\_id=user\_id

)

```



\---



\# Test Data Strategy



Use isolated test data.



Prefer:



```text

unique email

unique username

generated IDs

factories

fixtures

test data builders

```



Example:



```python

email = f"e2e\_{unique\_id()}@example.test"

```



Do not use real customer data.



Do not depend on shared mutable records.



\---



\# Data Ownership



Each E2E test should know:



```text

who created the data

which test owns it

when it should be removed

```



Example:



```text

Test creates:

User

Wishlist

Item



Test owns:

User

Wishlist

Item



Cleanup:

Delete Item

Delete Wishlist

Delete User

```



\---



\# Cleanup



Cleanup must run even when the workflow fails.



Use:



```python

try:

&#x20;   ...

finally:

&#x20;   cleanup()

```



or pytest fixtures:



```python

@pytest.fixture

def e2e\_data():

&#x20;   data = create\_data()



&#x20;   yield data



&#x20;   cleanup(data)

```



Cleanup order should respect dependencies.



Example:



```text

Wishlist Item

&#x20;   ↓

Wishlist

&#x20;   ↓

User

```



Delete dependent resources first.



\---



\# Assertions



Validate important business outcomes.



Do not only check:



```python

assert response.status\_code == 200

```



Also verify:



```text

resource exists

resource fields are correct

relationships are correct

state changed

dependent service sees the change

deleted resource is inaccessible

```



Example:



```python

assert create\_response.status\_code == 201



wishlist\_id = create\_response.json()\["id"]



get\_response = wishlist\_api.get\_wishlist(wishlist\_id)



assert get\_response.status\_code == 200

assert get\_response.json()\["id"] == wishlist\_id

```



\---



\# Cross-Service Validation



When multiple services participate, validate the integration.



Example:



```text

User Service

&#x20;    ↓

Order Service

&#x20;    ↓

Payment Service

&#x20;    ↓

Notification Service

```



Verify the expected business result across service boundaries.



Do not assume that a successful first API call means the complete workflow succeeded.



\---



\# Asynchronous Workflows



Some APIs process operations asynchronously.



Examples:



```text

POST job

&#x20;   ↓

202 Accepted

&#x20;   ↓

processing

&#x20;   ↓

GET status

&#x20;   ↓

completed

```



Use polling where documented.



Example logic:



```text

Start operation

&#x20;   ↓

Receive job ID

&#x20;   ↓

Poll status

&#x20;   ↓

Stop when completed

&#x20;   ↓

Fail on timeout

```



Do not use arbitrary long sleeps when polling is appropriate.



\---



\# Polling Rules



Polling must define:



```text

maximum wait time

poll interval

success condition

failure condition

timeout condition

```



Example:



```text

Timeout:

60 seconds



Interval:

2 seconds



Success:

status == COMPLETED



Failure:

status == FAILED

```



Use actual documented behavior where available.



\---



\# Error Recovery



Test expected failure paths in important workflows.



Example:



```text

Create order

&#x20;  ↓

Payment fails

&#x20;  ↓

Order should not become COMPLETED

```



Verify the resulting state.



Do not assume rollback behavior unless documented or observed.



\---



\# Transaction and Consistency Checks



Where applicable verify:



```text

create → visible

update → updated everywhere expected

delete → inaccessible

relationship → consistent

```



For eventual consistency, allow the documented propagation mechanism.



Do not introduce arbitrary waits.



\---



\# Idempotency in E2E



For workflows containing retryable operations, verify idempotency where supported.



Example:



```text

Submit order

&#x20;  ↓

network retry

&#x20;  ↓

same idempotency key

```



Expected outcome may be:



```text

one order

```



rather than:



```text

two orders

```



Only test this when API semantics support idempotency.



\---



\# E2E Test Structure



Recommended:



```text

tests/

└── e2e/

&#x20;   ├── test\_user\_registration.py

&#x20;   ├── test\_authentication\_flow.py

&#x20;   ├── test\_wishlist\_flow.py

&#x20;   ├── test\_order\_flow.py

&#x20;   └── test\_business\_workflows.py

```



Organize according to the actual domain.



\---



\# Service Object Model Usage



E2E tests must use service APIs.



Example:



```python

users = UsersAPI(...)

wishlist = WishlistAPI(...)

auth = AuthAPI(...)



token = auth.login(credentials)



user = users.create\_user(...)

wishlist\_data = wishlist.create\_wishlist(

&#x20;   user\_id=user.id

)

```



Do not place raw URLs throughout E2E tests.



Avoid duplicating:



\* endpoint definitions

\* headers

\* authentication

\* payload builders

\* response parsing



\---



\# Fixtures



Prefer fixtures for reusable workflow prerequisites.



Examples:



```text

authenticated\_user

admin\_user

test\_product

test\_user

auth\_token

```



Fixtures must remain understandable.



Do not create huge fixtures that hide the actual workflow.



\---



\# E2E Test Independence



Ea



