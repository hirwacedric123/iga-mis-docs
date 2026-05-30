# Partner API documentation

Welcome to the **IGA LMS** integration documentation for external systems.

## Primary integration: MIS (Kepler College)

This site documents how **MIS** can retrieve **student professionalism** data from the LMS for any date range.

| | |
|--|--|
| **Campus** | Kepler College (`keplercollege`) |
| **Production base URL** | `https://keplercollege.igaafrica.com` |
| **Main guide** | [Student professionalism export](mis-professionalism-export.md) |

!!! note "API credentials"
    The API key is **not** included in this documentation. Your LMS administrator will share it with you through a **separate secure channel** (email or internal handoff).

## Quick start

1. Read [Student professionalism export](mis-professionalism-export.md).
2. Use base URL `https://keplercollege.igaafrica.com`.
3. Call `GET /api/professionalism/export/` with `from_date`, `to_date`, and `Authorization: Bearer <api_key>`.
4. Paginate using the `next` URL until all rows are retrieved.

## Support

Contact your **IGA / LMS administrator** for API keys, production URL confirmation, and student ID (`mis_student_id`) linking issues.
