# JSON Schema Examples

A collection of production-style JSON Schema definitions covering three common real-world use cases: extracting structured data from complex PDFs, modeling multi-period financial reports, and defining API contracts for data submission services.

All schemas are written in **JSON Schema draft-07** and follow consistent conventions around field typing, required vs. optional fields, nested structure design, and embedded validation logic.

---

## Schemas

### 1. `pdf_extraction_schema.json` - Multi-Section PDF Data Extraction

Designed for extracting key data points from large, variable PDF documents (25-150+ pages). The schema handles documents where the number of sections is not fixed and each section may or may not contain subsections, tables, and free-text fields.

Key design decisions:
- **Recursive `$ref`** on the `subsections` property allows unlimited nesting depth without duplicating the section definition
- **Per-field `data_type` and `confidence` score** so downstream consumers know how reliable each extracted value is
- **Conditional required validation** using `if/then`: if a field is marked `required: true`, its value cannot be null
- **`null_reason` field** for every nullable value. Instead of silently dropping missing data, the schema requires an explanation (`missing_in_source`, `redacted`, `not_applicable`)
- **`validation_summary` block** at the root level tracks total fields extracted, missing required fields, and categorized errors with field paths

---

### 2. `financial_report_schema.json` - Financial Report Extraction

Models structured data extracted from multi-period financial report PDFs, where each fiscal period contains its own income statement and line-item table.

Key design decisions:
- **Repeating `fiscal_periods` array**: each period is independently validated, so a report with 4 quarters produces 4 fully validated period objects
- **Sum constraint documentation**: the `totals.total_revenue` field is explicitly documented as needing to equal the sum of `fiscal_periods[*].income_statement.revenue`. This is enforced in the extraction pipeline rather than the schema itself (JSON Schema draft-07 does not support cross-field arithmetic), but the constraint is clearly stated in the field description so any consumer of the schema knows what to check
- **`["number", "null"]` union types** on optional financial figures. Many line items are legitimately absent in interim reports, so null is valid but typed null rather than a missing key
- **ISO format enforcement**: `reporting_currency` uses a regex pattern for ISO 4217 codes; dates use `"format": "date"`
- **`additionalProperties: false`** at every level to prevent unvalidated fields from silently passing through

---

### 3. `api_contract_schema.json` - Structured Data Submission API Contract

Defines the expected request payload for a FastAPI endpoint that accepts extracted document data from multiple pipeline sources. The contract is designed to be the single source of truth for both the producing service (extraction pipeline) and the consuming service (storage/processing layer).

Key design decisions:
- **Discriminated union on `document_type`**: the `payload.data` object's expected shape varies by document type. The `document_type` enum drives validation routing in the application layer, and the schema makes the relationship explicit
- **Reusable `$definitions`** for `field_value` and `address`: shared structures referenced with `$ref` rather than duplicated across properties
- **`request_id` as UUID**: `"format": "uuid"` enforces idempotency key format at the schema level
- **Role-based `submitted_by`**: the enum on `role` ensures only known actor types can submit, which feeds into downstream audit logging
- **Strict `additionalProperties: false` throughout**: any field not explicitly defined in the schema causes validation to fail, making contract drift visible immediately rather than silently

---

## Conventions Used Across All Schemas

| Convention | Approach |
|---|---|
| Draft version | JSON Schema draft-07 |
| Required vs optional | Explicit `required` array at every object level |
| Nullable fields | `["type", "null"]` union, never just omitting the key |
| Undocumented fields | `additionalProperties: false` on all objects |
| Field documentation | Every property has a `description` explaining what it holds and why |
| Validation logic | Embedded via `if/then`, `pattern`, `format`, `minimum`, `enum`, and `minLength` |
| Reuse | `$ref` and `definitions` for any structure used more than once |

---

## Author

**Diluwara Khatun** - Backend Engineer  
[LinkedIn](https://www.linkedin.com/in/diluwara-khatun-9b4810265/) . [GitHub](https://github.com/diluwara)
