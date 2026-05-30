# Student professionalism export (MIS integration)

This document describes how **MIS** can retrieve **student professionalism** data from the LMS (IGA/iTSA) for **any date range**.

> **Primary campus for MIS:** **Kepler College** (`campus_subdomain`: `keplercollege`).  
> Use the Kepler College base URL for all requests unless you are explicitly integrating another campus later.

## Overview

| Item | Value |
|------|--------|
| **Endpoint** | `GET /api/professionalism/export/` |
| **Base URL** | Your campus LMS host, e.g. `https://kigali.{your-domain}` |
| **Authentication** | Shared API key (see below) |
| **Format** | JSON |
| **Primary campus** | **keplercollege** (Kepler College) |
| **Scope** | All enrolled students with a **MIS student ID**, all courses (optional filters) |

### What is included per student per course

| Field | Description |
|--------|-------------|
| `mis_student_id` | Student ID in MIS (e.g. `250285`) |
| `course_code` | Module/course code as MIS uses it (`mis_course_id`, or base code, or LMS code) |
| `deliverables_due_count` | Assignments, quizzes, and discussions **due** in the date range |
| `deliverables_on_time_count` | How many of those were completed on time |
| `on_time_submission_pct` | 100% minus 4% per late deliverable (0–100) |
| `academic_integrity_violation_count` | Violations in the period |
| `academic_integrity_pct` | 100% minus 4% per violation, minus manual deductions in the period |
| `positive_observation_count` | Instructor positive observations in the period |
| `negative_observation_count` | Instructor negative observations in the period |
| `observation_count` | Total observations in the period |
| `overall_score` | Average of on-time and integrity %, plus ±5% per net observation, capped at 100 |

Only students who have **`mis_student_id`** set in the LMS are returned.

---

## Authentication

Set on the server: environment variable `MIS_PROFESSIONALISM_EXPORT_API_KEY` (long random secret; never commit to git).

Send the key on every request using **one** of:

```http
Authorization: Bearer <api_key>
```

or

```http
X-MIS-API-Key: <api_key>
```

| HTTP status | Meaning |
|-------------|---------|
| `401` | Missing or invalid API key |
| `403` | Export not configured on server (no API key set) |

---

## Request

### Required query parameters

| Parameter | Format | Example |
|-----------|--------|---------|
| `from_date` | `YYYY-MM-DD` | `2026-01-01` |
| `to_date` | `YYYY-MM-DD` | `2026-04-30` |

Rules:

- `from_date` must be on or before `to_date`.
- Maximum range: **366 days**.

### Optional query parameters

| Parameter | Description |
|-----------|-------------|
| `mis_student_id` | Return only this student (e.g. `250285`) |
| `course_code` | Return only enrollments for this module code (e.g. `ENG81101`) |
| `page` | Page number (default `1`) |
| `page_size` | Rows per page (default `200`, max `1000`) |

### Example (Kepler College — production)

```http
GET /api/professionalism/export/?from_date=2026-01-01&to_date=2026-04-30&page=1&page_size=200
Host: keplercollege.igaafrica.com
Authorization: Bearer YOUR_API_KEY
Accept: application/json
```

Full URL: `https://keplercollege.igaafrica.com/api/professionalism/export/?from_date=2026-01-01&to_date=2026-04-30`

**Local development:** `http://keplercollege.localhost:8000/api/professionalism/export/...` (or `Host: keplercollege.localhost` on `127.0.0.1`).

---

## Response

### Success (`200`)

```json
{
  "campus_subdomain": "keplercollege",
  "campus_schema": "keplercollege",
  "campus_code": "KC",
  "campus_name": "Kepler College",
  "from_date": "2026-01-01",
  "to_date": "2026-04-30",
  "count": 1250,
  "page": 1,
  "page_size": 200,
  "next": "https://kigali.example.com/api/professionalism/export/?from_date=2026-01-01&to_date=2026-04-30&page=2&page_size=200",
  "results": [
    {
      "mis_student_id": "250285",
      "course_code": "ENG81101",
      "deliverables_due_count": 12,
      "deliverables_on_time_count": 11,
      "on_time_submission_pct": 96.0,
      "academic_integrity_violation_count": 0,
      "academic_integrity_pct": 100.0,
      "observation_count": 2,
      "positive_observation_count": 2,
      "negative_observation_count": 0,
      "observation_net_score": 2,
      "overall_score": 100.0
    }
  ]
}
```

- **`count`**: total matching enrollments (all pages).
- **`next`**: URL for the next page, or `null` on the last page.
- Paginate until `next` is `null`.

### Errors (`400`)

```json
{
  "error": "from_date is required (YYYY-MM-DD)."
}
```

---

## Date range behaviour

Metrics are calculated **only from activity in the selected period**:

| Metric | Included when |
|--------|----------------|
| On-time | Assignment/quiz/discussion **due date** falls between `from_date` and `to_date` |
| Academic integrity | Quiz proctoring events, flagged submissions, or manual deductions **dated** in the range |
| Observations | `observation_date` in the range |

If no deliverables were due in the period, `on_time_submission_pct` is **100%**.

---

## Campus (multi-tenant)

Each campus has its **own data** (separate database schema). You must scope every request to one campus.

### Option A — Campus URL (recommended)

Call the export on the **campus hostname** (same host students use for that institution):

| Environment | Example base URL |
|-------------|------------------|
| Production | `https://keplercollege.igaafrica.com` |
| Local dev | `http://keplercollege.localhost:8000` |

```http
GET https://keplercollege.igaafrica.com/api/professionalism/export/?from_date=2026-01-01&to_date=2026-04-30
Authorization: Bearer <api_key>
```

Repeat for **each campus** (Kiziba, EAUR, etc.) with that campus’s subdomain.

### Option B — `campus` query parameter

When the server host does **not** identify a campus (e.g. `127.0.0.1` or a shared gateway), pass:

| Parameter | Example | Description |
|-----------|---------|-------------|
| `campus` | `keplercollege` | Campus **subdomain** or **schema name** |

```http
GET http://127.0.0.1:8000/api/professionalism/export/?campus=keplercollege&from_date=2026-01-01&to_date=2026-04-30
Host: keplercollege.localhost
Authorization: Bearer <api_key>
```

If both host and `campus` are set, they **must match** or the API returns `400`.

### List campuses

```http
GET /api/professionalism/campuses/
Authorization: Bearer <api_key>
```

Response:

```json
{
  "count": 3,
  "campuses": [
    {
      "campus_subdomain": "keplercollege",
      "campus_schema": "keplercollege",
      "campus_code": "KC",
      "campus_name": "Kepler College",
      "base_url_hint": "https://keplercollege.{your-domain}/api/professionalism/export/"
    }
  ]
}
```

(Public picker without API key: `GET /api/public/campuses/` — mobile app use.)

### Campus fields on export response

Every export response includes:

| Field | Description |
|--------|-------------|
| `campus_subdomain` | e.g. `keplercollege` |
| `campus_schema` | Database schema name (usually same as subdomain) |
| `campus_code` | Short code e.g. `KC` |
| `campus_name` | Display name |

Each row in `results` is for that campus only.

---

## Operations checklist (LMS admin)

1. Generate a strong API key and set `MIS_PROFESSIONALISM_EXPORT_API_KEY` in production `.env`.
2. Restart the application.
3. Share this document and the API key securely with MIS (not over public chat).
4. Ensure students have **`mis_student_id`** populated (management command: `link_mis_student_ids`).
5. Ensure courses use **`mis_course_id`** when LMS codes differ from MIS module codes.

---

## Postman quick test

1. Method: **GET**
2. URL: `{{base_url}}/api/professionalism/export/?from_date=2026-01-01&to_date=2026-04-30`
3. Header: `Authorization` = `Bearer {{api_key}}`
4. Expect `200` and a `results` array.
