# QA Hermes Skills

A comprehensive suite of expert QA and Test Automation skills for Antigravity & Hermes AI agents.

## Available Skills

| Skill | Path | Description |
| :--- | :--- | :--- |
| **`swagger-analysis`** | [`skills/swagger-analysis/SKILL.md`](file:///C:/Users/s_ergashov/qa-hermes-skills/skills/swagger-analysis/SKILL.md) | Universal Swagger/OpenAPI analyzer (URL, JSON, YAML) extracting Pydantic v2 models, payloads, and CRUD dependency graphs for SOM automation. |
| **`swagger-testcases`** | [`skills/swagger-testcases/SKILL.md`](file:///C:/Users/s_ergashov/qa-hermes-skills/skills/swagger-testcases/SKILL.md) | Universal API test case designer in **Qase.io format** (JSON & Markdown) ingesting Swagger/OpenAPI (URL, JSON, YAML) with $ref resolution. |
| **`tz-analyzer`** | [`skills/tz-analyzer/SKILL.md`](file:///C:/Users/s_ergashov/qa-hermes-skills/skills/tz-analyzer/SKILL.md) | Shift-Left requirement defect auditor (34 defect taxonomy, typos, casing) with styled Word (.docx) audit report generation. |
| **`doc-testcases`** | [`skills/doc-testcases/SKILL.md`](file:///C:/Users/s_ergashov/qa-hermes-skills/skills/doc-testcases/SKILL.md) | Universal test case designer in **Qase.io format** (JSON & Markdown) ingesting all document types (TZ, PRD, SRS, BRD, User Stories, BPMN, UI) across all formats (.docx, PDFs, images). |
| **`service-object-model`** | [`skills/service-object-model/SKILL.md`](file:///C:/Users/s_ergashov/qa-hermes-skills/skills/service-object-model/SKILL.md) | Scaffolds Python/Pytest Service Object Model (SOM) test automation suites with Pydantic and Allure. |
| **`qa-review`** | [`skills/qa-review/SKILL.md`](file:///C:/Users/s_ergashov/qa-hermes-skills/skills/qa-review/SKILL.md) | Authoritative QA auditor & Quality Gatekeeper reviewing `tz-analyzer`, `doc-testcases`, and `swagger-testcases` against global standards (ISO 29119, ISTQB, RFC). |

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
                                          qa-review
                           (Global Standards & Quality Gatekeeper)
                                              │
                                              ▼
                                    service-object-model
                                 (Automated Pytest Suites)
```
