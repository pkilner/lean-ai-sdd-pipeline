---
name: sdd_api_spec
description: Generate an API & Schema Spec — the third step in the spec-driven pipeline. Defines exact contracts including API endpoints, request/response shapes, database schemas, event schemas, and validation rules.
---

The argument passed to this skill is the feature name in kebab-case. Use it as the feature identifier throughout.

## Steps

1. Read the following for context:
   - `docs/features/{feature-name}/01_feature_brief.md` (required)
   - `docs/features/{feature-name}/02_tech_design.md` (required — do not proceed if missing)
   - `docs/mindmaps/architecture.md` (if it exists)

2. Generate `docs/features/{feature-name}/03_api_spec.md` using the template below.

3. After writing the file, present the Review Checklist to the user.

---

## Output Template

```markdown
# API & Schema Spec: {Feature Name}

> Status: Draft
> Created: {today's date}
> Feature ID: {feature-name}
> Depends on: 02_tech_design.md

## Data Models

Define all data structures introduced or modified by this feature.

### {Model Name}

| Field | Type | Required | Description |
|---|---|---|---|
| id | string (UUID) | Yes | |
| createdAt | timestamp | Yes | |
| ... | | | |

(Repeat for each model)

## Database Schema

Define schema changes for this feature's persistence layer(s) (SQL, NoSQL, on-device storage — whatever this project uses).

### Table/Collection: {name}

```sql
CREATE TABLE {table_name} (
  id TEXT PRIMARY KEY,
  ...
);
```

Include indexes where relevant.

## API Endpoints

For each endpoint:

### {METHOD} /path/to/endpoint

**Description:** What this endpoint does.

**Auth required:** Yes / No

**Request:**
```json
{
  "field": "type"
}
```

**Response (200):**
```json
{
  "field": "type"
}
```

**Error responses:**

| Code | Reason |
|---|---|
| 400 | Invalid input |
| 401 | Unauthorised |
| 404 | Not found |
| 500 | Server error |

(Repeat for each endpoint)

## Event Schemas

If this feature produces or consumes events (Kafka, SQS, pub/sub, webhooks, etc.), define them here.

### Topic/Channel: {name}

**Direction:** Produced / Consumed

**Payload:**
```json
{
  "field": "type"
}
```

(Omit this section if no events are involved)

## Sync Payload Format

Define the payload format used when syncing offline/local data to a remote store.

```json
{
  "field": "type"
}
```

(Omit this section if no sync is involved)

## Validation Rules

List all validation rules applied at API or data model boundaries.

| Field | Rule |
|---|---|

## Open Questions

List any unresolved contract questions. Leave blank if none.

---

## Review Checklist

Before running the next skill, confirm:

- [ ] All data models are fully defined (no missing fields)
- [ ] All API endpoints are specified (method, path, request, response, errors)
- [ ] Database schema is complete
- [ ] Event schemas are defined (if applicable)
- [ ] Sync payload format is defined (if applicable)
- [ ] Validation rules cover all inputs
- [ ] No open questions remain

**Next step:** When approved, run `/sdd_impl_plan {feature-name}`
```
