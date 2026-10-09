# QA Hermes Skills

A comprehensive suite of expert QA and Test Automation skills for Antigravity & Hermes AI agents.

## Available Skills

| Skill | Path | Description |
| :--- | :--- | :--- |
| **`swagger-analysis`** | [`skills/swagger-analysis/SKILL.md`](file:///C:/Users/s_ergashov/qa-hermes-skills/skills/swagger-analysis/SKILL.md) | Parses Swagger/OpenAPI specs to extract endpoints, schemas, and contracts for SOM automation. |
| **`swagger-testcases`** | [`skills/swagger-testcases/SKILL.md`](file:///C:/Users/s_ergashov/qa-hermes-skills/skills/swagger-testcases/SKILL.md) | Generates exhaustive, production-grade **API test cases in Qase.io format** (JSON & Markdown) from Swagger/OpenAPI specs. |
| **`tz-analyzer`** | [`skills/tz-analyzer/SKILL.md`](file:///C:/Users/s_ergashov/qa-hermes-skills/skills/tz-analyzer/SKILL.md) | Shift-Left requirement defect auditor (34 defect taxonomy, typos, casing) with styled Word (.docx) audit report generation. |
| **`doc-testcases`** | [`skills/doc-testcases/SKILL.md`](file:///C:/Users/s_ergashov/qa-hermes-skills/skills/doc-testcases/SKILL.md) | Generates exhaustive, detailed **functional & business logic test cases in Qase.io format** (JSON & Markdown) from documentation. |
| **`service-object-model`** | [`skills/service-object-model/SKILL.md`](file:///C:/Users/s_ergashov/qa-hermes-skills/skills/service-object-model/SKILL.md) | Scaffolds Python/Pytest Service Object Model (SOM) test automation suites with Pydantic and Allure. |
| **`qa-review`** | [`skills/qa-review/SKILL.md`](file:///C:/Users/s_ergashov/qa-hermes-skills/skills/qa-review/SKILL.md) | Automated audit and iterative self-correction loop for generated SOM test automation suites. |

---

## End-to-End QA Workflow

```text
       [Project Documentation: TZ / PRD]             [Swagger / OpenAPI Specification]
                       │                                             │
                       ├──► tz-analyzer (Early Defects)               ├──► swagger-analysis (Contract Extraction)
                       │                                             │
                       ▼                                             ▼
                 doc-testcases                               swagger-testcases
              (Qase.io Functional)                            (Qase.io API)
                       │                                             │
                       └──────────────────────┬──────────────────────┘
                                              │
                                              ▼
                                    service-object-model
                                 (Automated Pytest Suites)
                                              │
                                              ▼
                                          qa-review
                                 (100% Audit & Self-Correction)
```
