# API & Service Communication Contracts

ContosoUniversity exposes a server-rendered web interface via 38 ASP.NET MVC 5 action endpoints across 6 controllers; there is no REST API or service-to-service HTTP communication — all interactions use synchronous browser HTTP requests with HTML responses and a single MSMQ queue for async notifications.

## Service Catalog

| Service | Port | Category | Purpose |
|---|---|---|---|
| ContosoUniversity Web | 80 / 443 (IIS) | Business | Single ASP.NET MVC 5 web application serving all university management functions |

> Note: This is a single-process monolith with no microservices, containers, or separate deployable units.

## API Endpoints Inventory

| Controller | Method | Path | Request Parameters | Response |
|---|---|---|---|---|
| HomeController | GET | / (Home/Index) | — | HTML — enrollment statistics page |
| HomeController | GET | /Home/About | — | HTML — enrollment date summary |
| HomeController | GET | /Home/Contact | — | HTML — contact page |
| HomeController | GET | /Home/Error | — | HTML — error view |
| HomeController | GET | /Home/Unauthorized | — | HTML — unauthorized view |
| StudentsController | GET | /Students | sortOrder, currentFilter, searchString, page (query) | HTML — paginated student list |
| StudentsController | GET | /Students/Details/{id} | id (path) | HTML — student detail view |
| StudentsController | GET | /Students/Create | — | HTML — create form |
| StudentsController | POST | /Students/Create | Student (form body: LastName, FirstMidName, EnrollmentDate) | Redirect on success / HTML on validation error |
| StudentsController | GET | /Students/Edit/{id} | id (path) | HTML — edit form |
| StudentsController | POST | /Students/Edit/{id} | Student (form body: ID, LastName, FirstMidName, EnrollmentDate) | Redirect on success / HTML on error |
| StudentsController | GET | /Students/Delete/{id} | id (path) | HTML — delete confirmation |
| StudentsController | POST | /Students/Delete/{id} | id (form body) | Redirect on success |
| CoursesController | GET | /Courses | — | HTML — course list |
| CoursesController | GET | /Courses/Details/{id} | id (path) | HTML — course detail view |
| CoursesController | GET | /Courses/Create | — | HTML — create form |
| CoursesController | POST | /Courses/Create | Course + teachingMaterialImage (multipart/form-data) | Redirect on success / HTML on error |
| CoursesController | GET | /Courses/Edit/{id} | id (path) | HTML — edit form |
| CoursesController | POST | /Courses/Edit/{id} | Course + teachingMaterialImage (multipart/form-data) | Redirect on success / HTML on error |
| CoursesController | GET | /Courses/Delete/{id} | id (path) | HTML — delete confirmation |
| CoursesController | POST | /Courses/Delete/{id} | id (form body) | Redirect on success |
| DepartmentsController | GET | /Departments | — | HTML — department list |
| DepartmentsController | GET | /Departments/Details/{id} | id (path) | HTML — department detail view |
| DepartmentsController | GET | /Departments/Create | — | HTML — create form |
| DepartmentsController | POST | /Departments/Create | Department (form body: Name, Budget, StartDate, InstructorID) | Redirect on success / HTML on error |
| DepartmentsController | GET | /Departments/Edit/{id} | id (path) | HTML — edit form |
| DepartmentsController | POST | /Departments/Edit/{id} | Department (form body: DepartmentID, Name, Budget, StartDate, InstructorID, RowVersion) | Redirect on success / HTML on error |
| DepartmentsController | GET | /Departments/Delete/{id} | id (path) | HTML — delete confirmation |
| DepartmentsController | POST | /Departments/Delete/{id} | id (form body) | Redirect on success |
| InstructorsController | GET | /Instructors | id, courseID (query — optional selection) | HTML — instructor list with optional course/enrollment detail |
| InstructorsController | GET | /Instructors/Details/{id} | id (path) | HTML — instructor detail |
| InstructorsController | GET | /Instructors/Create | — | HTML — create form |
| InstructorsController | POST | /Instructors/Create | Instructor + selectedCourses[] (form body) | Redirect on success / HTML on error |
| InstructorsController | GET | /Instructors/Edit/{id} | id (path) | HTML — edit form |
| InstructorsController | POST | /Instructors/Edit/{id} | id + selectedCourses[] (form body) | Redirect on success / HTML on error |
| InstructorsController | GET | /Instructors/Delete/{id} | id (path) | HTML — delete confirmation |
| InstructorsController | POST | /Instructors/Delete/{id} | id (form body) | Redirect on success |
| NotificationsController | GET | /Notifications | — | HTML — notification list |
| NotificationsController | POST | /Notifications/MarkAllRead | — | Redirect |

## Management & Observability Endpoints

| Endpoint | Description |
|---|---|
| None configured | No health check, metrics, or Swagger/OpenAPI endpoints are present |

> The application has no `/health`, `/metrics`, `/swagger`, or actuator-equivalent endpoints. For cloud/container deployments, health check endpoints must be added before deploying to Azure App Service, AKS, or any orchestrated environment.

## DTOs & Contracts

The application does not use separate DTO classes — domain entity classes are used directly as MVC model binding targets and view models:

| Class | Role | Notes |
|---|---|---|
| `Student` | Form binding target (POST Create/Edit), View model | Domain entity used directly; no dedicated request/response DTO |
| `Course` | Form binding target (POST Create/Edit), View model | Includes `TeachingMaterialImagePath` for file upload path storage |
| `Department` | Form binding target (POST Create/Edit), View model | Includes `RowVersion` byte array for optimistic concurrency |
| `Instructor` | Form binding target (POST Create/Edit), View model | Bound with `selectedCourses[]` string array for course assignments |
| `Notification` | Read view model | Serialised as JSON for MSMQ transmission via Newtonsoft.Json |
| `InstructorIndexData` | View model (GET Instructors/Index) | Aggregates Instructors, Courses, and Enrollments for index page |
| `AssignedCourseData` | View model (GET Instructors/Edit) | Projects course assignment checkboxes for edit form |
| `EnrollmentDateGroup` | View model (GET Home/About) | Groups enrollment counts by date for statistics page |
| `PaginatedList<T>` | Pagination wrapper | Generic; wraps IQueryable results with page metadata for Students/Index |

No OpenAPI/Swagger specification, protobuf schemas, or GraphQL definitions exist. Serialization uses **Newtonsoft.Json 13.0.3** for MSMQ message payloads only; MVC model binding and Razor views handle all other data exchange. `[Bind(Include = "...")]` attribute lists provide explicit allow-list binding on POST actions.

## Communication Patterns

**Synchronous (browser ↔ server):** All 38 endpoints follow a synchronous request-response pattern. The browser sends an HTTP GET or POST; the MVC controller queries or mutates via Entity Framework Core synchronously, then returns a full HTML page (Razor view) or an HTTP redirect.

**Asynchronous (CRUD → MSMQ):** After each successful create, update, or delete operation, `BaseController.SendEntityNotification()` posts a JSON-serialised `Notification` object to a private MSMQ queue (`.\Private$\ContosoUniversityNotifications`). This call is fire-and-forget with a try/catch that swallows exceptions silently to avoid breaking the main operation. There is no consumer, retry logic, dead-letter queue, or message ordering guarantee visible in the codebase.

**Resilience patterns:** None implemented. There are no circuit breakers, retry policies, timeouts, or bulkhead patterns. Database failures will propagate as unhandled exceptions.

**Service discovery:** Not applicable — single monolith with one SQL Server connection string hardcoded in Web.config.

**API gateway:** Not present.

**Security posture:** No authentication, no authorization, and no HTTPS enforcement are configured. All 38 endpoints are publicly accessible with no authorization checks. The `[ValidateAntiForgeryToken]` attribute is applied on all mutating POST actions (Create, Edit, Delete) providing CSRF protection, but there is no identity or role-based access control. Transport security (HTTPS/TLS) must be enforced at the IIS or reverse-proxy level as no application-level redirect is configured.

## Service Technology Matrix

| Capability | ContosoUniversity Web |
|---|---|
| Web Framework | ASP.NET MVC 5 (System.Web) |
| Data Access | Entity Framework Core 3.1 + SQL Server |
| Service Discovery | None |
| API Gateway | None |
| Health Checks | None |
| Caching | None (no caching layer implemented) |
| Metrics / Observability | None |
| Authentication | None |
| Messaging | MSMQ (System.Messaging) — send only |

## Service Communication Sequence

```mermaid
sequenceDiagram
    participant Browser as "Web Browser"
    participant Ctrl as "MVC Controller"
    participant EF as "EF Core (SchoolContext)"
    participant DB as "SQL Server"
    participant NotiSvc as "NotificationService"
    participant MSMQ as "MSMQ Queue"

    Browser->>Ctrl: GET /Students (sortOrder, page, search)
    Ctrl->>EF: db.Students.Where(...).OrderBy(...) 
    EF->>DB: SELECT * FROM Person WHERE Discriminator=Student
    DB-->>EF: Student rows
    EF-->>Ctrl: IQueryable Students
    Ctrl->>Ctrl: PaginatedList.Create(students, page, 10)
    Ctrl-->>Browser: 200 HTML (paginated student list)

    Browser->>Ctrl: POST /Students/Create (form data)
    Ctrl->>Ctrl: ModelState.IsValid check
    alt Validation passes
        Ctrl->>EF: db.Students.Add(student); db.SaveChanges()
        EF->>DB: INSERT INTO Person (...)
        DB-->>EF: Rows affected
        EF-->>Ctrl: Success
        Ctrl->>NotiSvc: SendNotification(Student, id, CREATE)
        NotiSvc->>MSMQ: Send JSON message
        MSMQ-->>NotiSvc: Queued
        Ctrl-->>Browser: 302 Redirect /Students
    else Validation fails
        Ctrl-->>Browser: 200 HTML (form with errors)
    end

    Browser->>Ctrl: GET /Instructors?id=1&courseID=5
    Ctrl->>EF: Include(Courses).Include(OfficeAssignment).Include(CourseAssignments)
    EF->>DB: SELECT with JOINs
    DB-->>EF: Instructor + related data
    EF-->>Ctrl: InstructorIndexData (Instructors, Courses, Enrollments)
    Ctrl-->>Browser: 200 HTML (instructor detail + enrollment table)
```
