# Security Assessment Report

**Generated:** 2026-08-06T12:02:00Z
**Project:** ContosoUniversity

## Summary

| Metric | Count |
|--------|-------|
| Total Findings | 8 |
| CVE Vulnerabilities | 1 |
| CWE Vulnerabilities | 7 |
| Total Rules Assessed | 59 |
| Rules Passed | 52 |

### By Severity

| Severity | Count |
|----------|-------|
| mandatory | 2 |
| optional | 4 |
| potential | 3 |

### By Category

| Category | Count |
|----------|-------|
| CVE | 1 |
| File & Path Security | 2 |
| Code Quality | 3 |
| Credentials & Secrets | 2 |

---

## CVE Findings (Dependency Vulnerabilities)

### CVE-2024-0056: Microsoft.Data.SqlClient and System.Data.SqlClient vulnerable to SQL Data Provider Security Feature Bypass
- **Severity:** mandatory
- **Story Points:** 1
- **Files:** `ContosoUniversity/packages.config`

[CVE-2024-0056](https://github.com/advisories/GHSA-98g6-xh36-x2p7): Microsoft.Data.SqlClient and System.Data.SqlClient vulnerable to SQL Data Provider Security Feature Bypass

Severity: HIGH

Affected dependencies:
  - Microsoft.Data.SqlClient@2.1.4 (declared at packages.config) — affected range: < 2.1.7

Recommended fix:
  - Upgrade Microsoft.Data.SqlClient to 2.1.7 or later

---

## CWE Findings (Code-Level Vulnerabilities)

### CWE-22: Improper Limitation of a Pathname to a Restricted Directory ('Path Traversal')
- **Category:** File & Path Security
- **Severity:** optional
- **Story Points:** 8
- **Files:** `ContosoUniversity/Controllers/CoursesController.cs`

In CoursesController.DeleteConfirmed (line ~229), the file path is reconstructed using `Server.MapPath(course.TeachingMaterialImagePath)` where `TeachingMaterialImagePath` is a value read from the database — but during the Create/Edit POST, the `[Bind]` attribute includes `TeachingMaterialImagePath` in the bound field list, meaning an attacker could submit a crafted value such as `~/../../sensitive/file` in the POST body, which could then be persisted to the database and subsequently used in `Server.MapPath()` to resolve a path outside the web root. There is no validation or sanitization of the `TeachingMaterialImagePath` field value before it is stored or used in file system operations.

---

### CWE-434: Unrestricted Upload of File with Dangerous Type
- **Category:** File & Path Security
- **Severity:** mandatory
- **Story Points:** 8
- **Files:** `ContosoUniversity/Controllers/CoursesController.cs`

CoursesController.Create (line ~55) and CoursesController.Edit (line ~128) accept file uploads and validate the file type using only the file extension (`Path.GetExtension(teachingMaterialImage.FileName).ToLower()`). This check can be bypassed: (1) the MIME type / Content-Type header is not verified against the actual file content, allowing an attacker to upload a file with a valid image extension (e.g., `.jpg`) that contains executable script content; (2) on some IIS configurations, double extensions (e.g., `shell.asp;.jpg`) can bypass extension-only checks. The allowed extension list (`jpg`, `jpeg`, `png`, `gif`, `bmp`) is client-supplied filename-based only — no magic byte / file signature validation is performed.

---

### CWE-477: Use of Obsolete Function
- **Category:** Code Quality
- **Severity:** optional
- **Story Points:** 1
- **Files:** `ContosoUniversity/Services/NotificationService.cs`

NotificationService uses System.Messaging.MessageQueue, which is a Windows-only legacy API that is not supported in .NET Core/.NET 5+ and has been superseded by modern cloud messaging services. The class uses MessageQueue.Create(), MessageQueue.Exists(), _queue.Send(), _queue.Receive(), and XmlMessageFormatter — all of which are part of the obsolete System.Messaging namespace. This API has no cross-platform or cloud-portable equivalent and is explicitly identified as a migration blocker by Microsoft's modernization tooling.

---

### CWE-772: Missing Release of Resource after Effective Lifetime
- **Category:** Code Quality
- **Severity:** potential
- **Story Points:** 3
- **Files:** `ContosoUniversity/Controllers/InstructorsController.cs`

InstructorsController.Dispose(bool disposing) (line ~252) calls `db.Dispose()` directly, and then calls `base.Dispose(disposing)`. BaseController.Dispose(bool disposing) also calls `db?.Dispose()` on the same instance. This results in a double-dispose of the SchoolContext DbContext object. While EF Core's DbContext.Dispose() is idempotent (safe to call multiple times without throwing), calling Dispose() on an already-disposed context leaves it in a permanently unusable state. Additionally, DepartmentsController.Dispose() has the same double-dispose pattern at line ~167.

---

### CWE-1057: Data Access Operations Outside of Expected Data Manager Component
- **Category:** Code Quality
- **Severity:** potential
- **Story Points:** 5
- **Files:** `ContosoUniversity/Controllers/StudentsController.cs`, `CoursesController.cs`, `DepartmentsController.cs`, `InstructorsController.cs`, `NotificationsController.cs`

All MVC controllers directly access the SchoolContext DbContext instance (db) inherited from BaseController, bypassing any service or repository layer. This violates separation of concerns and makes it impossible to unit test controllers without a real database connection.

---

### CWE-732: Incorrect Permission Assignment for Critical Resource
- **Category:** Credentials & Secrets
- **Severity:** optional
- **Story Points:** 5
- **Files:** `ContosoUniversity/Services/NotificationService.cs`

NotificationService constructor (line 23) grants 'Everyone' full control over the MSMQ notification queue when it is created: `_queue.SetPermissions("Everyone", MessageQueueAccessRights.FullControl)`. This allows any user or process on the Windows machine to read, write, delete, and administer the message queue.

---

### CWE-778: Insufficient Logging
- **Category:** Credentials & Secrets
- **Severity:** potential
- **Story Points:** 3
- **Files:** `ContosoUniversity/Controllers/BaseController.cs`, `ContosoUniversity/Services/NotificationService.cs`

Security-relevant events are not logged to a persistent, observable log store. BaseController.SendEntityNotification() catches all exceptions from notification delivery and silently swallows them with only a Debug.WriteLine() call. There is no logging of entity create/update/delete operations, authentication events, authorization decisions, or file upload events with user identity, timestamp, or IP address. No structured logging framework is configured.
