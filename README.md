# Course Management API

A REST API for managing a course catalog, departments, and student enrollments, built around Python, FastAPI, and PostgreSQL.

The project aims to demonstrate practical backend development: API design, relational data modeling, validation, business rules, automated testing, and a reproducible development environment.

> **Status: Planning.** This README defines the intended implementation. Features, routes, and project files below are proposed; they are not claims of completed functionality.

## Project Goals

- Provide a consistent API for managing courses and departments.
- Keep request handling, business logic, and database access separate.
- Enforce data integrity through validation and database constraints.
- Extend the catalog with authentication and student enrollment.
- Maintain a repository that is easy to understand, run, and test.

## Technology Stack

| Area | Technology | Purpose |
| --- | --- | --- |
| Language | Python | Application and business logic |
| API framework | FastAPI | HTTP routes and API documentation |
| Validation | Pydantic | Request and response schemas |
| Database | PostgreSQL | Relational data storage |
| ORM | SQLAlchemy | Database models and queries |
| Migrations | Alembic | Versioned database schema changes |
| Testing | pytest | Unit and integration tests |
| Environment | Docker Compose | Local API and database services |
| Automation | GitHub Actions | Automated checks on pushes and pull requests |

Dependency versions will be selected and recorded when implementation begins.


## Planned Features

### Phase 1 — Course Catalog

- [ ] Create, list, retrieve, update, and delete courses.
- [ ] Create and list departments.
- [ ] Search courses by title and filter by department.
- [ ] Paginate course lists with predictable ordering.
- [ ] Validate course fields and department references.
- [ ] Reject duplicate course IDs.
- [ ] Provide consistent application error responses.
- [ ] Add liveness and database readiness endpoints.
- [ ] Document and test the API.

### Phase 2 — Users and Enrollment

- [ ] Add registration and login with hashed passwords.
- [ ] Add token authentication and admin/student permissions.
- [ ] Restrict course and department changes to administrators.
- [ ] Allow students to enroll, view their enrollments, and withdraw.
- [ ] Prevent duplicate enrollments and enrollment beyond course capacity.
- [ ] Enforce ownership checks for student enrollment records.

The initial catalog is intended for local development. Authentication and authorization are required before exposing administrative operations publicly.

## Planned Data Model

| Entity | Main fields | Relationships |
| --- | --- | --- |
| Department | `id`, `name` | One department has many courses |
| Course | `id`, `title`, `description`, `credits`, `department_id`, `capacity` | Belongs to one department; has many enrollments |
| User | `id`, `name`, `email`, `password_hash`, `role` | A student has many enrollments |
| Enrollment | `id`, `student_id`, `course_id`, `enrolled_at` | Connects a student to a course |

Course IDs are unique strings such as `CS-101`. User email addresses are unique, and each student/course pair may appear only once in the enrollment table.

For the first version, each course has one enrollment pool. Semesters, sections, schedules, and prerequisites are outside the initial scope.


## Proposed API Routes

The initial API uses `/courses` and `/departments` as its resource paths.

| Method | Endpoint | Purpose | Success |
| --- | --- | --- | --- |
| GET | `/health/live` | Check that the application is running | `200` |
| GET | `/health/ready` | Check database readiness | `200` |
| GET | `/courses` | List, search, and filter courses | `200` |
| GET | `/courses/{course_id}` | Retrieve one course | `200` |
| POST | `/courses` | Create a course | `201` |
| PATCH | `/courses/{course_id}` | Update selected course fields | `200` |
| DELETE | `/courses/{course_id}` | Delete a course without enrollments | `204` |
| GET | `/departments` | List departments | `200` |
| POST | `/departments` | Create a department | `201` |

Planned course list parameters: `search`, `department_id`, `limit` (default `20`, maximum `100`), and `offset` (default `0`). Results will be ordered by course ID.

Phase 2 will introduce:

| Method | Endpoint | Purpose |
| --- | --- | --- |
| POST | `/auth/register` | Register a student account |
| POST | `/auth/login` | Authenticate and receive an access token |
| GET | `/users/me` | Retrieve the current user's profile |
| POST | `/courses/{course_id}/enrollments` | Enroll the current student |
| GET | `/users/me/enrollments` | List the current student's enrollments |
| DELETE | `/courses/{course_id}/enrollments/me` | Withdraw the current student |

Registration will not allow clients to assign themselves an administrator role.

## Example API Contract

The examples below describe the planned behavior.

### Create a Course

`POST /courses`

```json
{
  "id": "CS-101",
  "title": "Introduction to Programming",
  "description": "Programming fundamentals using Python.",
  "credits": 3,
  "department_id": 1,
  "capacity": 40
}
```

Assuming department `1` exists, the API will return `201 Created` with the stored course fields and a `Location: /courses/CS-101` header.


### List Courses

`GET /courses?department_id=1&limit=20&offset=0`

```json
{
  "items": [
    {
      "id": "CS-101",
      "title": "Introduction to Programming",
      "description": "Programming fundamentals using Python.",
      "credits": 3,
      "department_id": 1,
      "capacity": 40
    }
  ],
  "total": 1,
  "limit": 20,
  "offset": 0
}
```

### Application Error

For a missing course, return `404 Not Found`:

```json
{
  "error": {
    "code": "COURSE_NOT_FOUND",
    "message": "Course CS-999 was not found."
  }
}
```

| Status | Intended use |
| --- | --- |
| `401 Unauthorized` | Missing or invalid authentication in Phase 2 |
| `403 Forbidden` | Authenticated user lacks permission |
| `404 Not Found` | Requested resource does not exist |
| `409 Conflict` | Duplicate record, full course, or deletion blocked by enrollments |
| `422 Unprocessable Entity` | Invalid fields or nonexistent department reference |
| `500 Internal Server Error` | Unexpected server failure; no internal details returned |
| `503 Service Unavailable` | Database readiness check fails |

Application errors will use the envelope above. Request validation errors will retain FastAPI's standard `detail` response structure.


## Business Rules

- Course IDs are immutable; course IDs and titles must not be blank.
- Credits and capacity must be positive integers.
- Every course must reference an existing department.
- A course with enrollments cannot be deleted.
- Capacity cannot be reduced below the current enrollment count.
- Enrollment requires a student account and an existing course with space available.
- Duplicate enrollment must be prevented with a database uniqueness constraint.
- Capacity checks and enrollment writes must be performed atomically to prevent concurrent requests from overbooking a course.


## Planned Project Structure

| Path | Responsibility |
| --- | --- |
| `app/main.py` | Application entry point and router registration |
| `app/core/` | Configuration, logging, and security utilities |
| `app/database/` | Database engine and session management |
| `app/models/` | SQLAlchemy database models |
| `app/schemas/` | Pydantic request and response schemas |
| `app/routes/` | HTTP endpoints |
| `app/services/` | Business rules and transaction coordination |
| `app/repositories/` | Database queries and persistence |
| `app/exceptions/` | Application exceptions and response handlers |
| `migrations/` | Alembic migration scripts |
| `tests/` | Unit and integration tests |
| `.github/workflows/` | GitHub Actions workflows |
| `.env.example` | Documented configuration placeholders |
| `pyproject.toml` | Project metadata and dependencies |
| `Dockerfile` | API container definition |
| `compose.yaml` | Local API and PostgreSQL services |

Routes will handle HTTP concerns, services will enforce business rules, and repositories will handle database operations. Services will own transactions that span multiple operations.

## Development Setup

Runnable setup instructions will be added when the application, dependency configuration, migrations, and container files exist. The intended workflow is:

1. Clone the repository.
2. Copy `.env.example` to `.env` and configure the local database.
3. Build and start the PostgreSQL and API services.
4. Apply Alembic migrations and seed sample departments.
5. Open the interactive API documentation and run the tests.

The planned local API address is `http://127.0.0.1:8000`, with Swagger UI at `/docs` and ReDoc at `/redoc`.

Configuration will be supplied through environment variables. Real credentials and `.env` files must not be committed. Exact commands and supported runtime versions will be documented after verification.

## Testing Plan

- Course creation, retrieval, listing, partial updates, and deletion.
- Missing resources, invalid fields, duplicate IDs, and invalid department references.
- Search, filtering, pagination, and response schemas.
- Database constraints and migration behavior against a separate test database.
- Liveness and readiness behavior when the database is unavailable.
- Authentication, role checks, and enrollment ownership in Phase 2.
- Duplicate enrollment, full capacity, and concurrent enrollment requests.

GitHub Actions will run automated checks on pushes and pull requests. Test results and coverage will be reported only after the test suite exists and runs successfully.


