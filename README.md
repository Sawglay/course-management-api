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

