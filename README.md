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


