\# Target Project Structure



The Service Object Model skill should use the following project architecture as the preferred target structure.



```text

ServiceObjectModel/

│

├── swagger.yaml

│

├── auth/

│   ├── role\_factory.py

│   └── token\_provider.py

│

├── config/

│   ├── base\_test.py

│   ├── headers.py

│   └── stages.py

│

├── services/

│   │

│   ├── users/

│   │   ├── api.py

│   │   ├── endpoints.py

│   │   ├── payloads.py

│   │   └── models/

│   │       └── model\_user.py

│   │

│   └── wishlists/

│       ├── api.py

│       ├── endpoints.py

│       ├── payloads.py

│       └── models/

│           └── model\_wishlist.py

│

├── tests/

│   ├── smoke/

│   ├── regression/

│   └── e2e/

│

├── utils/

│   ├── helper.py

│   └── logger.py

│

├── conftest.py

├── pytest.ini

└── requirements.txt

```



\---



\# Directory Responsibilities



\## `swagger.yaml`



The canonical API specification used by the project.



It should contain or reference the OpenAPI/Swagger contract used to generate and validate the Service Object Model.



The skill must not modify the specification unless explicitly requested.



\---



\# `auth/`



Authentication and authorization support.



```text

auth/

├── role\_factory.py

└── token\_provider.py

```



\## `token\_provider.py`



Responsible for:



\* obtaining tokens;

\* refreshing tokens when required;

\* providing authentication credentials to API services;

\* supporting different authentication states when required;

\* preventing credentials from being hardcoded.



Example responsibilities:



```text

get\_token()

refresh\_token()

get\_expired\_token()

get\_invalid\_token()

```



The exact methods depend on the API authentication model.



\---



\## `role\_factory.py`



Responsible for creating authentication contexts for different roles or authorization states.



Examples:



```text

admin

user

manager

readonly

unauthenticated

```



Only implement roles that are documented or explicitly provided by the project.



If roles are unknown:



```text

UNKNOWN

```



Do not invent authorization roles.



\---



\# `config/`



Global test and environment configuration.



```text

config/

├── base\_test.py

├── headers.py

└── stages.py

```



\## `base\_test.py`



Contains shared test configuration or base test behavior.



Possible responsibilities:



\* common setup;

\* environment initialization;

\* API client initialization;

\* common fixtures;

\* shared test context.



Do not put resource-specific API logic here.



\---



\## `headers.py`



Centralizes HTTP header definitions.



Examples:



```text

Authorization

Content-Type

Accept

X-Request-ID

custom API headers

```



Headers that vary by test should remain configurable.



Do not hardcode authentication tokens here.



\---



\## `stages.py`



Defines supported environments/stages.



Example:



```text

local

dev

test

stage

production

```



Each stage may define:



```text

base\_url

environment configuration

feature flags

timeouts

```



Secrets must come from environment variables or an approved secret-management mechanism.



\---



\# `services/`



Contains API resource implementations.



Each logical API resource receives its own directory.



Example:



```text

services/

├── users/

└── wishlists/

```



Additional resources should follow the same pattern:



```text

services/

├── users/

├── wishlists/

├── orders/

├── products/

├── payments/

└── ...

```



Do not create a single giant service class containing all API endpoints.



\---



\# Service Directory Structure



Every service should preferably follow:



```text

<resource>/

├── api.py

├── endpoints.py

├── payloads.py

└── models/

&#x20;   └── model\_<resource>.py

```



\---



\## `api.py`



Contains the actual API operations for the resource.



Example:



```text

UserApi

├── create\_user()

├── get\_user()

├── list\_users()

├── update\_user()

└── delete\_user()

```



Responsibilities:



\* construct API requests;

\* call the shared HTTP mechanism;

\* use endpoint definitions;

\* use payload definitions;

\* return responses/models.



Do not place test assertions here.



\---



\## `endpoints.py`



Contains endpoint definitions for the resource.



Example:



```text

USERS = "/api/v1/users"

USER\_BY\_ID = "/api/v1/users/{id}"

```



The purpose is to prevent raw endpoint paths from being duplicated throughout the project.



Tests and service methods should not repeatedly hardcode the same URL.



\---



\## `payloads.py`



Contains reusable request payload builders.



Example:



```text

create\_user\_payload()

update\_user\_payload()

invalid\_user\_payload()

```



Payload builders may support:



\* valid data;

\* minimal data;

\* optional fields;

\* boundary values;

\* negative-test data.



However, large test scenarios should remain in the test layer.



\---



\## `models/`



Contains resource-specific request/response models.



Example:



```text

models/

└── model\_user.py

```



For another resource:



```text

models/

└── model\_wishlist.py

```



Models should reflect the API contract.



They should preserve documented:



\* field names;

\* field types;

\* optionality;

\* nested structures;

\* enums;

\* formats.



Do not silently introduce undocumented business rules.



\---



\# `tests/`



Test suites are separated by purpose.



```text

tests/

├── smoke/

├── regression/

└── e2e/

```



\## `smoke/`



Contains critical-path tests that quickly determine whether the API is operational.



Examples:



```text

authentication

health-critical endpoints

core CRUD operations

critical business operations

```



Smoke tests must remain focused and fast.



\---



\## `regression/`



Contains broader API coverage intended to detect regressions.



Possible coverage:



```text

positive

negative

validation

authorization

CRUD

boundary

error handling

pagination

filtering

sorting

```



\---



\## `e2e/`



Contains multi-service business workflows.



Example:



```text

Login

&#x20; ↓

Create user

&#x20; ↓

Create wishlist

&#x20; ↓

Add product

&#x20; ↓

Retrieve wishlist

&#x20; ↓

Verify state

```



E2E tests may use multiple service objects.



Do not duplicate service implementation inside E2E tests.



\---



\# `utils/`



Reusable technical utilities.



```text

utils/

├── helper.py

└── logger.py

```



\## `helper.py`



Contains generic reusable utilities that do not belong to a specific API resource.



Examples:



\* data generation;

\* date helpers;

\* random identifiers;

\* response utilities;

\* polling helpers.



Do not turn `helper.py` into a dumping ground for unrelated API logic.



\---



\## `logger.py`



Centralized logging.



Should support useful debugging information such as:



```text

request method

request URL

request ID

status code

response timing

failure context

```



Sensitive information must be masked.



Never log:



```text

passwords

access tokens

refresh tokens

API keys

client secrets

```



\---



\# `conftest.py`



Contains shared pytest fixtures.



Possible fixtures:



```text

api\_client

token

authenticated\_user

admin\_user

test\_data

environment

```



Fixtures should provide reusable test context.



Avoid creating resource-specific API methods in `conftest.py`.



\---



\# `pytest.ini`



Contains pytest configuration.



Possible configuration:



```text

markers

test discovery

logging

default options

```



Keep suite configuration centralized.



\---



\# `requirements.txt`



Contains Python dependencies required by the automation project.



Dependencies must be:



\* necessary;

\* compatible;

\* reproducible;

\* explicitly versioned or constrained according to project standards.



Do not add libraries without a concrete reason.



\---



\# Architecture Dependency Rules



The preferred dependency direction is:



```text

tests

&#x20; ↓

services

&#x20; ↓

endpoints / payloads / models

&#x20; ↓

auth / config / utilities

```



The following rules apply:



\### Tests may use services



```text

tests → services

```



\### Services may use configuration/auth/utilities



```text

services → auth

services → config

services → utils

```



\### Tests should not duplicate endpoint paths



Avoid:



```text

tests → raw URL

```



Prefer:



```text

tests → UserApi → endpoints.py

```



\### E2E tests may compose multiple services



```text

e2e

&#x20;↓

UserApi

&#x20;↓

WishlistApi

&#x20;↓

ProductApi

```



\### Resource services should not depend heavily on other resource services



If a cross-resource workflow is required, prefer composing services in the test/E2E layer.



\---



\# Adding New API Resources



When Swagger analysis identifies a new resource, create:



```text

services/

└── <resource>/

&#x20;   ├── api.py

&#x20;   ├── endpoints.py

&#x20;   ├── payloads.py

&#x20;   └── models/

&#x20;       └── model\_<resource>.py

```



Example:



```text

services/

└── orders/

&#x20;   ├── api.py

&#x20;   ├── endpoints.py

&#x20;   ├── payloads.py

&#x20;   └── models/

&#x20;       └── model\_order.py

```



Do not modify unrelated services unless required by an actual dependency.



\---



\# Structural Validation



After generating or modifying the Service Object Model, verify:



```text

\[ ] swagger.yaml exists

\[ ] auth structure exists

\[ ] config structure exists

\[ ] every resource has api.py

\[ ] every resource has endpoints.py

\[ ] every resource has payloads.py

\[ ] every resource has models/

\[ ] resource models match API schemas

\[ ] smoke directory exists

\[ ] regression directory exists

\[ ] e2e directory exists

\[ ] utilities are centralized

\[ ] conftest.py exists

\[ ] pytest.ini exists

\[ ] requirements.txt exists

```



The exact files may differ only when the existing project architecture provides an equivalent implementation.



\---



\# Target Architecture Principle



The Service Object Model is the reusable foundation for the other QA skills:



```text

swagger-analysis

&#x20;      ↓

service-object-model

&#x20;      ↓

api-test-design

&#x20;      ↓

&#x20;┌─────┴─────┐

&#x20;↓           ↓

smoke     regression

&#x20;└─────┬─────┘

&#x20;      ↓

&#x20;  e2e-api-testing

&#x20;      ↓

&#x20;   qa-review

```



The Service Object Model must therefore remain:



\* reusable;

\* deterministic;

\* test-framework compatible;

\* independent of individual test scenarios;

\* easy to extend when Swagger changes;

\* easy for other QA skills to consume.



