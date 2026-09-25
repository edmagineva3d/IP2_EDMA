# Vertical Slice Architecture — Student Management API

A simple API demonstrating Vertical Slice Architecture in .NET 10, built for educational purposes.

---

## Tech Stack

| Component      | Technology                          |
| -------------- | ----------------------------------- |
| Framework      | .NET 10                             |
| Architecture   | Vertical Slice + Clean Architecture |
| ORM            | EF Core 10                          |
| Database       | InMemory (for demo)                 |
| Validation     | FluentValidation                    |
| CQRS           | Custom ICommand/IQuery (No MediatR) |
| API Docs       | Scalar (No Swashbuckle)             |
| Result Pattern | Domain.Common.Result<T>             |

---

## Project Structure

```text
VerticalSlice/
├── VerticalSlice.sln
├── Domain/
│   ├── Common/
│   │   ├── BaseEntity.cs
│   │   └── Result.cs
│   └── Entities/
│       ├── Student.cs
│       ├── Course.cs
│       ├── Admission.cs
│       └── AdmissionCourse.cs
│
├── Application/
│   ├── Abstractions/
│   │   ├── Data/
│   │   │   └── IAppDbContext.cs
│   │   └── Messaging/
│   │       ├── ICommand.cs
│   │       ├── ICommandHandler.cs
│   │       ├── IQuery.cs
│   │       └── IQueryHandler.cs
│   │
│   └── Features/
│       ├── Students/
│       │   ├── CreateStudent/
│       │   └── GetStudentById/
│       │
│       ├── Courses/
│       │   ├── CreateCourse/
│       │   ├── GetCourseById/
│       │   ├── UpdateCourse/
│       │   └── DeleteCourse/
│       │       ├── DeleteCourseCommand.cs
│       │       └── DeleteCourseCommandHandler.cs
│       │
│       └── Admissions/
│           ├── CreateAdmission/
│           └── GetAdmissionById/
│
├── Infrastructure/
│   └── Persistence/
│       └── AppDbContext.cs
│
└── Api/
    ├── Features/
    │   ├── Students/
    │   │   └── StudentEndpoints.cs
    │   ├── Courses/
    │   │   └── CourseEndpoints.cs
    │   └── Admissions/
    │       └── AdmissionEndpoints.cs
    ├── Common/
    │   └── Extensions/
    │       └── ResultExtensions.cs
    └── Program.cs
```

---

# Getting Started

## Prerequisites

* .NET 10 SDK
* Visual Studio 2022, Visual Studio 2026, VS Code with C# extension, or JetBrains Rider

## Run the API

Open PowerShell in the project folder:

```powershell
dotnet run
```

The API runs at:

```text
http://localhost:5205
```

Scalar:

```text
http://localhost:5205/scalar/v1
```

---

# API Endpoints

## Students

| Method | Endpoint             | Description          |
| ------ | -------------------- | -------------------- |
| POST   | `/api/students`      | Create a new student |
| GET    | `/api/students/{id}` | Get student by ID    |
| GET    | `/api/students`      | Get all students     |

## Courses

| Method | Endpoint            | Description               |
| ------ | ------------------- | ------------------------- |
| POST   | `/api/courses`      | Create a new course       |
| GET    | `/api/courses/{id}` | Get course by ID          |
| GET    | `/api/courses`      | Get all courses           |
| PUT    | `/api/courses/{id}` | Update an existing course |
| DELETE | `/api/courses/{id}` | Delete a course           |

## Admissions

| Method | Endpoint               | Description                      |
| ------ | ---------------------- | -------------------------------- |
| POST   | `/api/admissions`      | Create admission with courses    |
| GET    | `/api/admissions/{id}` | Get admission by ID with courses |

---

# Delete Course Feature

The Delete Course feature was added for Lab Sheet 05.

Endpoint:

```text
DELETE /api/courses/{id}
```

The feature uses a non-generic `Result` because a successful DELETE does not return response data.

## Delete Behavior

| Situation                              | Status Code      |
| -------------------------------------- | ---------------- |
| Course successfully deleted            | `204 No Content` |
| Course does not exist                  | `404 Not Found`  |
| Course is already used by an admission | `409 Conflict`   |

A course that is already referenced by an admission cannot be deleted.

---

# PowerShell Testing Commands

The following commands were used to test the API successfully.

## 1. Get All Students

```powershell
curl.exe -s http://localhost:5205/api/students
```

## 2. Get All Courses

```powershell
curl.exe -s http://localhost:5205/api/courses
```

---

# Screenshot 1 — Successful Delete

First, create a temporary course:

```powershell
$body = @{
    title = "Temporary Course"
    code = "TM101"
    credits = 3
} | ConvertTo-Json

Invoke-RestMethod -Uri "http://localhost:5205/api/courses" -Method Post -ContentType "application/json" -Body $body
```

Temporary Course ID:

```text
32107b50-6ab0-470f-83bf-3a3e6dee7940
```

Delete the temporary course:

```powershell
curl.exe -i -X DELETE http://localhost:5205/api/courses/32107b50-6ab0-470f-83bf-3a3e6dee7940
```

Expected result:

```text
HTTP/1.1 204 No Content
```

---

# Screenshot 2 — Delete the Same Course Again

Run the DELETE command again:

```powershell
curl.exe -i -X DELETE http://localhost:5205/api/courses/32107b50-6ab0-470f-83bf-3a3e6dee7940
```

Expected result:

```text
HTTP/1.1 404 Not Found
```

Example error:

```json
{
  "status": 404,
  "detail": "Course not found.",
  "errorCode": "Course.NotFound"
}
```

---

# Screenshot 3 — Delete a Course in Use

Get the current student list:

```powershell
curl.exe -s http://localhost:5205/api/students
```

Student ID used:

```text
f0e10c92-0bc8-435a-865c-f439e554f605
```

Get the course list:

```powershell
curl.exe -s http://localhost:5205/api/courses
```

Course ID used:

```text
8208724b-1e38-4d24-8a12-efe0323b65a8
```

Create an admission using the course:

```powershell
$admissionBody = @{
    studentId = "f0e10c92-0bc8-435a-865c-f439e554f605"
    academicYear = "2026-2027"
    courseIds = @("8208724b-1e38-4d24-8a12-efe0323b65a8")
} | ConvertTo-Json

Invoke-RestMethod -Uri "http://localhost:5205/api/admissions" -Method Post -ContentType "application/json" -Body $admissionBody
```

Then try to delete the course:

```powershell
curl.exe -i -X DELETE http://localhost:5205/api/courses/8208724b-1e38-4d24-8a12-efe0323b65a8
```

Expected result:

```text
HTTP/1.1 409 Conflict
```

Example error:

```json
{
  "status": 409,
  "detail": "Cannot delete a course that is part of an existing admission.",
  "errorCode": "Course.InUse"
}
```

---

# Screenshot 4 — Full CRUD Test

After restarting the API to reset the InMemory database:

```text
Ctrl + C
```

Then:

```powershell
dotnet run
```

## Create

```powershell
$body = @{
    title = "Full CRUD Test"
    code = "FC101"
    credits = 3
} | ConvertTo-Json

$created = Invoke-RestMethod -Uri "http://localhost:5205/api/courses" -Method Post -ContentType "application/json" -Body $body

$ID = $created.id

"Created: $ID"
```

## GET All


$r = Invoke-WebRequest -Uri "http://localhost:5205/api/courses" -Method Get
"GET all: $($r.StatusCode)"
```

## GET One

```powershell
$r = Invoke-WebRequest -Uri "http://localhost:5205/api/courses/$ID" -Method Get
"GET one: $($r.StatusCode)"


## PUT


$updateBody = @{
    title = "Updated Full CRUD Test"
    code = "FC101"
    credits = 4
} | ConvertTo-Json

$r = Invoke-WebRequest -Uri "http://localhost:5205/api/courses/$ID" -Method Put -ContentType "application/json" -Body $updateBody

"PUT: $($r.StatusCode)"


## DELETE


$r = Invoke-WebRequest -Uri "http://localhost:5205/api/courses/$ID" -Method Delete

"DELETE: $($r.StatusCode)"


Successful output:


Created: f2d99038-6912-409d-8af7-9704318488a6
GET all: 200
GET one: 200
PUT: 200
DELETE: 204


---

# Route Naming

The API uses named routes for GET-by-ID endpoints:


GetCourseById
GetStudentById
GetAdmissionById


These route names are used consistently when generating URLs for the corresponding resources.

---

# Architecture Rules

1. **Domain has ZERO external dependencies**
2. **All data access through DbContext**
3. **No Repository pattern**
4. **Use Result/Result<T> for business errors**
5. **Use records for DTOs**
6. **Always pass CancellationToken through async chains**
7. **Use commands for write operations**
8. **Handlers contain the business logic**
9. **Do not use controllers**
10. **Do not use MediatR**
11. **Do not delete courses that are referenced by admissions**

---

# Key Concepts

## Vertical Slice Architecture

Code is organized by feature instead of technical layers.

Examples:


CreateStudent
GetStudentById
CreateCourse
GetCourseById
UpdateCourse
DeleteCourse
CreateAdmission
GetAdmissionById


Each feature contains the code needed for that specific operation.

## Result Pattern

Instead of throwing exceptions for expected business errors, the application returns Result objects.


return Result<T>.Success(response);

For failures:


return Result<T>.Failure(
    Error.NotFound("Code", "Message")
);


## Custom CQRS

The project uses custom CQRS interfaces without MediatR.

* `ICommand<TResponse>` — Write operations
* `ICommandHandler<TCommand, TResponse>` — Handles commands
* `IQuery<TResponse>` — Read operations
* `IQueryHandler<TQuery, TResponse>` — Handles queries

The Delete Course command uses:


ICommand<Result>


because a successful DELETE returns no data.

---

# Seed Data

The API automatically seeds the following data on startup.

## Students

* John Doe
* Jane Smith

## Courses

* Introduction to Programming — CS101 — 3 credits
* Data Structures — CS201 — 4 credits
* Database Systems — CS301 — 3 credits

---

# Common Mistakes to Avoid

1. **Don't use MediatR** — The project uses custom ICommand/IQuery.
2. **Don't throw exceptions for expected business errors** — Use Result.
3. **Don't skip validation.**
4. **Don't forget endpoint registration in Program.cs.**
5. **Don't put business logic in endpoints** — Handlers do the work.
6. **Don't use controllers** — The project uses Minimal APIs.
7. **Don't use Repository pattern** — Use DbContext directly.
8. **Don't delete a course that is already used by an admission.**

---



# License

This project is for educational purposes.

---

**Developed by Elvin Manuel R. Luces, MIT**
