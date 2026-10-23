# CampusDesk — Student Management System

A beginner-friendly student management application built as an independently deployable React frontend and FastAPI microservices. The frontend makes requests only to the API Gateway. Authentication, students, and courses each have their own FastAPI app and SQLite database.

## Architecture

```text
React + Vite (localhost:5173)
           │ /auth, /students, /courses
           ▼
API Gateway (localhost:8000) ── routes requests to the matching service
      ├── Authentication Service (localhost:8001) ── auth.db
      ├── Student Service        (localhost:8002) ── students.db
      └── Course Service         (localhost:8003) ── courses.db
```

Each service has its own FastAPI application, OpenAPI/Swagger documentation, data models, and database. The gateway proxies requests and surfaces upstream errors; each data service validates the shared JWT before allowing protected operations. Course enrollments are stored by the course service and reference student IDs without coupling the services to one another.

## Features

- Administrator registration and login, Argon2 password hashing, signed JWTs, current-user lookup, and frontend sign-out.
- Student create, list, update, delete, and detail endpoints.
- Course create, list, update, delete, detail, student enrollment, and enrollment-list endpoints.
- Overview dashboard with student/course counts and recent student records.
- Searchable student directory, course catalog, inline validation, and API error messages.
- Independent SQLite databases and environment-configurable service URLs and secrets.
- Postman collection for the gateway API.

## Requirements

- Python 3.10+
- Node.js 20+
- npm

## Setup

Open separate PowerShell terminals from the project directory. For **each** Python service, create/activate a virtual environment and install its requirements (or use a shared environment if preferred):

```powershell
cd auth-service
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

Repeat the venv and install commands in `student-service`, `course-service`, and `api-gateway`. If PowerShell blocks activation, allow it for the current terminal with `Set-ExecutionPolicy -Scope Process Bypass`.

Generate a strong secret once:

```powershell
py -c "import secrets; print(secrets.token_urlsafe(48))"
```

Copy the generated value and set the **same value** in each backend terminal before starting that service:

```powershell
$env:JWT_SECRET_KEY = "<paste-generated-secret>"
$env:JWT_ALGORITHM = "HS256"
```

Use a unique secret in real deployments. Do not commit secrets or a populated `.env` file.

## Run each backend

Start each in its own terminal with its venv active and `JWT_SECRET_KEY` / `JWT_ALGORITHM` set:

```powershell
# From auth-service
python -m uvicorn app.main:app --reload --port 8001

# From student-service
python -m uvicorn app.main:app --reload --port 8002

# From course-service
python -m uvicorn app.main:app --reload --port 8003
```

Start the gateway from `api-gateway` in another terminal:

```powershell
$env:AUTH_SERVICE_URL = "http://127.0.0.1:8001"
$env:STUDENT_SERVICE_URL = "http://127.0.0.1:8002"
$env:COURSE_SERVICE_URL = "http://127.0.0.1:8003"
$env:CORS_ALLOWED_ORIGINS = "http://localhost:5173,http://127.0.0.1:5173"
python -m uvicorn main:app --reload --port 8000
```

The SQLite files are created automatically in each service's working directory. Optional database URL variables are `AUTH_DATABASE_URL`, `STUDENT_DATABASE_URL`, and `COURSE_DATABASE_URL`; use SQLite URLs for this project. `ACCESS_TOKEN_MINUTES` controls access-token lifetime (default 60).

## Run the frontend

In a terminal:

```powershell
cd frontend
Copy-Item .env.example .env.local
npm install
npm run dev
```

Open the Vite URL printed by the command (normally `http://localhost:5173`). The gateway permits both `localhost` and `127.0.0.1` on port 5173 by default. Set `VITE_API_URL` in `frontend/.env.local` to the gateway URL if it is not `http://localhost:8000`.

## API and Swagger UI

All application API calls go through `http://localhost:8000`.

| Service | Swagger UI | OpenAPI JSON |
| --- | --- | --- |
| Gateway | `http://localhost:8000/docs` | `http://localhost:8000/openapi.json` |
| Authentication | `http://localhost:8001/docs` | `http://localhost:8001/openapi.json` |
| Students | `http://localhost:8002/docs` | `http://localhost:8002/openapi.json` |
| Courses | `http://localhost:8003/docs` | `http://localhost:8003/openapi.json` |

Use the gateway endpoints for the normal application flow:

- `POST /auth/register`, `POST /auth/login`, `GET /auth/me`
- `POST /students`, `GET /students`, `GET /students/{student_id}`, `PUT /students/{student_id}`, `DELETE /students/{student_id}`
- `POST /courses`, `GET /courses`, `GET /courses/{course_id}`, `PUT /courses/{course_id}`, `DELETE /courses/{course_id}`
- `POST /courses/{course_id}/students/{student_id}`, `GET /courses/{course_id}/students`

The auth endpoints and `/health` are public. Student/course operations require `Authorization: Bearer <access_token>`. Use the included [Postman collection](./postman/collection.json); run Register or Login first to save the collection token.

## Screenshots

For a submission screenshot, run the frontend and backend, register an account, then capture the Overview, Students, and Courses pages in a browser. Capture API examples from Swagger UI or Postman after sending the corresponding requests.

## Project layout

```text
auth-service/app/       Authentication FastAPI app, models, schemas, database
student-service/app/    Student FastAPI app, models, schemas, database
course-service/app/     Course FastAPI app, models, schemas, database
api-gateway/            Gateway routing to backend services
frontend/src/           React pages, components, API client, styles
postman/                API collection
```
