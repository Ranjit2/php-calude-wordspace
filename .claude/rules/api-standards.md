# API Standards
# Universal — applies to all frameworks.

## Versioning
All routes under `/api/v1/`. Never modify a v1 contract.
Add `/api/v2/` for breaking changes. Non-breaking additions are allowed in v1.

---

## URL design

```
GET    /api/v1/{resources}              → paginated list
GET    /api/v1/{resources}/{id}         → single resource
POST   /api/v1/{resources}              → create (returns 201)
PUT    /api/v1/{resources}/{id}         → full replace
PATCH  /api/v1/{resources}/{id}         → partial update
DELETE /api/v1/{resources}/{id}         → delete (returns 204)

Nested:
GET    /api/v1/{parents}/{id}/{children}

Actions (when a verb is needed):
POST   /api/v1/{resources}/{id}/publish
POST   /api/v1/{resources}/{id}/archive
POST   /api/v1/{resources}/{id}/send
```

---

## Response envelopes

### Single resource — 200 / 201
```json
{
  "data": {
    "id": 1,
    "status": "active",
    "created_at": "2026-05-01T10:30:00Z"
  },
  "meta": {}
}
```

### Collection — 200
```json
{
  "data": [
    { "id": 1, "status": "active" },
    { "id": 2, "status": "draft" }
  ],
  "meta": {
    "current_page": 1,
    "per_page": 15,
    "total": 43,
    "last_page": 3,
    "from": 1,
    "to": 15
  }
}
```

### Error — 4xx / 5xx
```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "The given data was invalid.",
    "details": {
      "email": ["The email field is required."],
      "title": ["The title must be at least 3 characters."]
    }
  }
}
```

---

## HTTP status codes

| Scenario                        | Code |
|---------------------------------|------|
| Read success                    | 200  |
| Created synchronously           | 201  |
| Accepted — job dispatched       | 202  |
| Deleted                         | 204  |
| Validation failed               | 422  |
| Unauthenticated                 | 401  |
| Forbidden                       | 403  |
| Not found                       | 404  |
| Conflict / duplicate            | 409  |
| Rate limited                    | 429  |
| External provider error         | 502  |
| Unhandled server error          | 500  |

---

## Machine-readable error codes

| Code                    | Meaning                              |
|-------------------------|--------------------------------------|
| VALIDATION_FAILED       | Request payload failed validation    |
| UNAUTHENTICATED         | No valid token or session            |
| FORBIDDEN               | Authenticated but not authorised     |
| NOT_FOUND               | Resource does not exist              |
| CONFLICT                | Duplicate or state conflict          |
| RATE_LIMITED            | Too many requests                    |
| EXTERNAL_SERVICE_ERROR  | Third-party API failure              |
| SERVER_ERROR            | Unhandled exception                  |

---

## Never expose in responses
- Internal file paths
- Password hashes, tokens, raw secrets
- Auto-increment IDs when using UUIDs externally
- Stack traces in production
- Internal service names or infrastructure details

---

## Pagination
Always paginate collections. Never return unbounded lists.
Default page size: 15. Maximum: 100 (enforce server-side).
