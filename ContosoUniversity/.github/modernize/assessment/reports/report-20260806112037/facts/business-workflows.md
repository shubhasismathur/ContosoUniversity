# Core Business Workflows

ContosoUniversity manages university administration workflows including student enrollment, course management, instructor assignment, and department administration.

## Domain Entities

| Entity | Service / Bounded Context | Description | Key Relationships |
|---|---|---|---|
| Student | Academic Records | Represents enrolled learners | Links to Enrollment records |
| Instructor | Academic Staffing | Represents teaching staff | Links to CourseAssignment and OfficeAssignment |
| Course | Curriculum | Represents taught subjects | Belongs to Department; links to Enrollment |
| Department | Administration | Represents academic departments | Owns multiple Courses; optional administrator |
| Enrollment | Academic Records | Student-course relationship with grade | Connects Student and Course |
| Notification | Operations | Change event record for UI/admin queue | Produced by CRUD workflows |

## Service-to-Domain Mapping

| Service | Domain Context | Owned Entities | External Dependencies |
|---|---|---|---|
| ContosoUniversity web app | University administration | Student, Instructor, Course, Department, Enrollment, Notification | SQL Server, MSMQ |

## Primary Workflows

### Workflow 1: Student Registration and Update

1. User opens student create/edit screen.
2. Controller validates enrollment date and model state.
3. Controller saves student changes through `SchoolContext`.
4. Base controller emits notification event for create/update/delete.

### Workflow 2: Course Management with Teaching Material Upload

1. User submits course create/edit form with optional image.
2. Controller validates extension and size, writes file under `Uploads/TeachingMaterials`.
3. Course data is persisted in SQL Server.
4. Notification event is sent for the operation.

### Workflow 3: Instructor Assignment

1. User updates instructor profile and selected course assignments.
2. Controller computes additions/removals and updates join table entries.
3. Changes are saved and notification emitted.

## Cross-Service Data Flows

The application is monolithic, so data composition occurs in-process rather than across independent services. Controllers query related entities via EF includes and return merged view models. Notification publication to MSMQ is the only asynchronous cross-boundary flow; if queue operations fail, the main CRUD operation still completes with error logging.

## Business Workflow Sequence

```mermaid
sequenceDiagram
    participant Staff
    participant StudentsCtrl as "StudentsController"
    participant Db as "SchoolContext"
    participant BaseCtrl as "BaseController"
    participant NotifySvc as "NotificationService"
    participant Queue as "MSMQ"

    Staff->>StudentsCtrl: Submit student create form
    StudentsCtrl->>StudentsCtrl: Validate enrollment date and model
    alt Validation passes
        StudentsCtrl->>Db: Save student
        Db-->>StudentsCtrl: Student persisted
        StudentsCtrl->>BaseCtrl: SendEntityNotification(Student, CREATE)
        BaseCtrl->>NotifySvc: Build notification payload
        alt Queue available
            NotifySvc->>Queue: Enqueue notification
            Queue-->>NotifySvc: Success
        else Queue unavailable
            Note over NotifySvc: Notification failure logged; workflow continues
        end
        StudentsCtrl-->>Staff: Redirect to student list
    else Validation fails
        StudentsCtrl-->>Staff: Return form with validation errors
    end
```

## Business Rules & Decision Logic

- Enrollment and hire dates must be within SQL-supported date ranges.
- Course teaching materials must match approved image extensions and size limits.
- Department edits handle optimistic concurrency (`RowVersion`) and surface current DB values after conflict.
- Notification side effects are non-blocking and should not fail the main business transaction.
