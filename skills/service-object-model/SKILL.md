---
name: service-object-model
description: Scaffold and generate the Service Object Model (SOM) API test automation framework in Python/Pytest, featuring a centralized ApiClient, Pydantic response models, deep data assertions, Allure reporting, and inline pytest marks (smoke, regression, e2e).
---

# Service Object Model (SOM) Test Automation

## Role & Purpose
You are a Senior API Test Automation Architect. Your responsibility is to take the inventory and specifications from `swagger-analysis` and generate a complete, production-ready **Service Object Model (SOM)** test automation suite in Python/Pytest.

### Key Requirements
1. **Standard SOM Architecture with `ApiClient`:** Clean separation of concerns (`auth/`, `client/`, `config/`, `services/<resource>/`, `tests/`, `utils/`).
2. **Centralized `ApiClient`:** All HTTP communication is abstracted through `client/api_client.py`, which handles sessions, headers, timeouts, structured logging, and Allure attachments.
3. **Inline Test Marks (No separated test folders):** Tests live together in `tests/test_<resource>.py`. Tests are categorized directly on each test function using `@pytest.mark.smoke`, `@pytest.mark.regression`, and `@pytest.mark.e2e`.
4. **Pydantic Response Validation & Deep Data Assertions:** Never check only status codes. Validate response bodies against Pydantic models AND assert exact field values matching payloads and business logic.
5. **Allure Reporting:** Every test function must include Allure metadata (`@allure.epic`, `@allure.feature`, `@allure.story`, `@allure.severity`, and `with allure.step(...)`).

---

## Target Project Architecture

```text
ServiceObjectModel/
├── swagger.yaml                 # Original API specification
├── auth/
│   ├── role_factory.py          # Credentials, roles, user accounts
│   └── token_provider.py        # Token acquisition, Bearer headers, refresh
├── client/
│   └── api_client.py            # Central HTTP client (Session, logging, Allure attachments)
├── config/
│   ├── base_test.py             # Base configurations & global settings
│   ├── headers.py               # Standard and custom HTTP headers
│   └── stages.py                # Environment hosts (dev, staging, prod)
├── services/
│   └── <resource>/              # Domain services (e.g., users, products, orders)
│       ├── api.py               # Service client methods (wraps ApiClient calls)
│       ├── endpoints.py         # Endpoint URI constants and formatters
│       ├── payloads.py          # Payload factories with unique dynamic data
│       └── models/              # Pydantic validation models
│           └── model_<resource>.py
├── tests/
│   ├── conftest.py              # Pytest fixtures (api_client, service clients, auto-cleanup)
│   ├── test_<resource>.py       # Test cases with inline marks & Allure steps
│   └── test_<workflow>.py       # E2E multi-service business flow tests
├── utils/
│   ├── helper.py                # Reusable utilities (unique IDs, wait_for_status)
│   └── logger.py                # Request/response logger
├── pytest.ini                   # Pytest & Allure markers and options
└── requirements.txt             # pytest, requests, pydantic, allure-pytest, faker
```

---

## Code Templates & Implementation Standards

### 1. `pytest.ini` (Markers & Allure)
```ini
[pytest]
addopts = -v -s --alluredir=reports/allure-results
markers =
    smoke: Critical path smoke tests for environment and health verification
    regression: Comprehensive functional, boundary, negative, and authorization tests
    e2e: End-to-end multi-step business workflow tests
```

### 2. `client/api_client.py` (Centralized HTTP Client)
All services interact with HTTP solely through `ApiClient`.
```python
# client/api_client.py
import json
import requests
import allure
from utils.logger import logger

class ApiClient:
    """
    Centralized HTTP client wrapping requests.Session.
    Handles base URL routing, timeout configuration, structured logging,
    and automatic Allure attachments for request/response bodies.
    """
    def __init__(self, base_url: str, session: requests.Session = None, timeout: int = 30):
        self.base_url = base_url.rstrip("/")
        self.session = session or requests.Session()
        self.timeout = timeout

    def request(self, method: str, endpoint: str, **kwargs) -> requests.Response:
        url = f"{self.base_url}{endpoint}"
        kwargs.setdefault("timeout", self.timeout)

        # Structured Logging & Allure Request Attachment
        logger.info(f"API REQUEST: [{method.upper()}] {url}")
        if "json" in kwargs:
            allure.attach(
                json.dumps(kwargs["json"], indent=2),
                name=f"Request JSON [{method.upper()} {endpoint}]",
                attachment_type=allure.attachment_type.JSON
            )

        response = self.session.request(method, url, **kwargs)

        # Structured Logging & Allure Response Attachment
        logger.info(f"API RESPONSE: [{response.status_code}] {url}")
        allure.attach(
            response.text,
            name=f"Response Body [{response.status_code} {method.upper()} {endpoint}]",
            attachment_type=allure.attachment_type.TEXT
        )
        return response

    def get(self, endpoint: str, **kwargs) -> requests.Response:
        return self.request("GET", endpoint, **kwargs)

    def post(self, endpoint: str, **kwargs) -> requests.Response:
        return self.request("POST", endpoint, **kwargs)

    def put(self, endpoint: str, **kwargs) -> requests.Response:
        return self.request("PUT", endpoint, **kwargs)

    def patch(self, endpoint: str, **kwargs) -> requests.Response:
        return self.request("PATCH", endpoint, **kwargs)

    def delete(self, endpoint: str, **kwargs) -> requests.Response:
        return self.request("DELETE", endpoint, **kwargs)
```

### 3. `endpoints.py` (Centralized Paths)
```python
# services/users/endpoints.py
class UserEndpoints:
    USERS = "/api/v1/users"
    USER_BY_ID = "/api/v1/users/{user_id}"
    PROFILE = "/api/v1/users/me"

    @staticmethod
    def user_by_id(user_id: str | int) -> str:
        return UserEndpoints.USER_BY_ID.format(user_id=user_id)
```

### 4. `payloads.py` (Dynamic Payload Factory)
```python
# services/users/payloads.py
import uuid

def user_create_payload(**kwargs) -> dict:
    uid = uuid.uuid4().hex[:8]
    payload = {
        "email": f"test_{uid}@example.com",
        "username": f"user_{uid}",
        "firstName": "Alex",
        "lastName": "QA",
        "role": "USER",
        "isActive": True
    }
    payload.update(kwargs)
    return payload
```

### 5. `models/model_<resource>.py` (Pydantic Response Model)
```python
# services/users/models/model_user.py
from pydantic import BaseModel, EmailStr, Field
from typing import Optional

class UserModel(BaseModel):
    id: str
    email: EmailStr
    username: str
    first_name: str = Field(alias="firstName")
    last_name: str = Field(alias="lastName")
    role: str
    is_active: bool = Field(alias="isActive")
    created_at: Optional[str] = Field(default=None, alias="createdAt")

    class Config:
        populate_by_name = True
```

### 6. `services/users/api.py` (Service Client Utilizing ApiClient)
```python
# services/users/api.py
import requests
from client.api_client import ApiClient
from services.users.endpoints import UserEndpoints
from services.users.models.model_user import UserModel

class UsersAPI:
    def __init__(self, client: ApiClient):
        self.client = client

    def create_user(self, payload: dict) -> requests.Response:
        return self.client.post(UserEndpoints.USERS, json=payload)

    def get_user(self, user_id: str | int) -> requests.Response:
        return self.client.get(UserEndpoints.user_by_id(user_id))

    def delete_user(self, user_id: str | int) -> requests.Response:
        return self.client.delete(UserEndpoints.user_by_id(user_id))

    @staticmethod
    def parse_user(response: requests.Response) -> UserModel:
        return UserModel.model_validate(response.json())
```

### 7. `tests/conftest.py` (Fixtures Providing ApiClient & Services)
```python
# tests/conftest.py
import pytest
import requests
from config.stages import StageConfig
from client.api_client import ApiClient
from services.users.api import UsersAPI
from auth.token_provider import TokenProvider

@pytest.fixture(scope="session")
def base_url():
    return StageConfig.get_base_url()

@pytest.fixture(scope="session")
def api_client(base_url):
    """Centralized authenticated ApiClient fixture."""
    session = requests.Session()
    token = TokenProvider.get_token(role="ADMIN")
    session.headers.update({
        "Authorization": f"Bearer {token}",
        "Content-Type": "application/json"
    })
    return ApiClient(base_url=base_url, session=session)

@pytest.fixture
def users_api(api_client):
    """Users domain service fixture."""
    return UsersAPI(client=api_client)
```

### 8. `tests/test_users.py` (Inline Marks, Deep Assertions & Allure)
```python
# tests/test_users.py
import pytest
import allure
from http import HTTPStatus
from services.users.payloads import user_create_payload
from services.users.models.model_user import UserModel

@allure.epic("Identity & Access Management")
@allure.feature("User Service")
class TestUsers:

    @allure.story("Create User - Happy Path")
    @allure.severity(allure.severity_level.BLOCKER)
    @pytest.mark.smoke
    @pytest.mark.regression
    def test_create_user_success(self, users_api):
        with allure.step("1. Prepare valid user payload"):
            payload = user_create_payload(role="USER")

        with allure.step("2. Call POST /api/v1/users"):
            response = users_api.create_user(payload)

        with allure.step("3. Verify HTTP status code 201"):
            assert response.status_code == HTTPStatus.CREATED, f"Expected 201, got {response.status_code}: {response.text}"

        with allure.step("4. Validate schema structure with Pydantic"):
            user_data: UserModel = users_api.parse_user(response)

        with allure.step("5. Perform deep data field assertions"):
            assert user_data.id is not None and len(user_data.id) > 0
            assert user_data.email == payload["email"]
            assert user_data.username == payload["username"]
            assert user_data.first_name == payload["firstName"]
            assert user_data.last_name == payload["lastName"]
            assert user_data.role == "USER"
            assert user_data.is_active is True

        # Teardown
        with allure.step("6. Cleanup created user"):
            users_api.delete_user(user_data.id)

    @allure.story("Create User - Missing Email Validation")
    @allure.severity(allure.severity_level.NORMAL)
    @pytest.mark.regression
    def test_create_user_missing_email(self, users_api):
        with allure.step("1. Prepare payload with missing email"):
            payload = user_create_payload()
            del payload["email"]

        with allure.step("2. Call POST /api/v1/users"):
            response = users_api.create_user(payload)

        with allure.step("3. Assert 422 Unprocessable Entity and error details"):
            assert response.status_code in [HTTPStatus.BAD_REQUEST, HTTPStatus.UNPROCESSABLE_ENTITY]
            err_json = response.json()
            assert "email" in str(err_json).lower()

    @allure.story("User Lifecycle E2E Workflow")
    @allure.severity(allure.severity_level.CRITICAL)
    @pytest.mark.e2e
    def test_user_lifecycle_e2e(self, users_api):
        with allure.step("1. Create user"):
            payload = user_create_payload()
            res_create = users_api.create_user(payload)
            assert res_create.status_code == HTTPStatus.CREATED
            user = users_api.parse_user(res_create)

        try:
            with allure.step("2. Read and verify created user"):
                res_get = users_api.get_user(user.id)
                assert res_get.status_code == HTTPStatus.OK
                fetched = users_api.parse_user(res_get)
                assert fetched.id == user.id
                assert fetched.email == payload["email"]
        finally:
            with allure.step("3. Delete user and verify removal"):
                res_del = users_api.delete_user(user.id)
                assert res_del.status_code in [HTTPStatus.OK, HTTPStatus.NO_CONTENT]
                res_verify = users_api.get_user(user.id)
                assert res_verify.status_code == HTTPStatus.NOT_FOUND
```

---

## Test Execution Commands
```bash
# Run smoke tests with Allure results
pytest -m smoke --alluredir=reports/allure-results

# Run full regression suite
pytest -m regression --alluredir=reports/allure-results

# Run E2E workflows
pytest -m e2e --alluredir=reports/allure-results

# Run everything
pytest --alluredir=reports/allure-results

# Serve Allure HTML report
allure serve reports/allure-results
```

---

## Handoff
Pass the complete implementation code and execution outputs to **`qa-review`** for automated compliance audit and iterative self-correction.
