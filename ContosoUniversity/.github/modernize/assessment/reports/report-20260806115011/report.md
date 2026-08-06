# Application Assessment Report

> **Project:** ContosoUniversity | **Framework:** ASP.NET MVC 5 / .NET Framework 4.8 | **Generated:** 2026-08-06

## Application Information

| Property | Value |
|---|---|
| Project | ContosoUniversity |
| Project Type | ASP.NET MVC 5 Web Application |
| Target Framework | .NET Framework 4.8 |
| Assessment Date | 2026-08-06 |
| AppCAT Version | 1.0.1127 |

## Summary

| Category | Count |
|---|---|
| Total Modernization Issues | 18 |
| Mandatory | 6 |
| Recommended | 9 |
| Optional | 5 |
| Security Findings (CVE + CWE) | 8 |
| &nbsp;&nbsp;Mandatory | 2 |
| &nbsp;&nbsp;Optional | 3 |
| &nbsp;&nbsp;Potential | 3 |

## Modernization Issues

### Mandatory Issues (6)

#### System.Messaging (MSMQ) is not supported on .NET Core / .NET 5+ {#systemmessaging-msmq-is-not-supported-on-net-core--net-5}

- **ID:** UA001  
- **Category:** Mandatory  
- **Effort:** High

The application uses System.Messaging for MSMQ-based notification queuing. System.Messaging is a Windows-only, .NET Framework library that is not available in .NET Core or .NET 5+.

**Recommendation:** Replace MSMQ with a cloud-native messaging service such as Azure Service Bus, Azure Queue Storage, or RabbitMQ via the MassTransit or NServiceBus abstraction libraries.

**API Change:** `System.Messaging.MessageQueue -> Azure.Messaging.ServiceBus.ServiceBusClient`

**Affected Files:**
- `Services/NotificationService.cs`

#### System.Web.* namespace is not available in .NET Core / .NET 5+ {#systemweb-namespace-is-not-available-in-net-core--net-5}

- **ID:** UA002  
- **Category:** Mandatory  
- **Effort:** High

The project heavily depends on System.Web.Mvc, System.Web.Optimization, System.Web.Routing and other System.Web types which do not exist in .NET Core.

**Recommendation:** Migrate to ASP.NET Core MVC. Replace System.Web.Mvc.Controller with Microsoft.AspNetCore.Mvc.Controller, update routing to attribute-based or endpoint routing, replace BundleConfig with Bundler & Minifier or Webpack.

**API Change:** `System.Web.Mvc.Controller -> Microsoft.AspNetCore.Mvc.Controller`

**Affected Files:**
- `Global.asax.cs`
- `App_Start/RouteConfig.cs`
- `App_Start/BundleConfig.cs`
- `App_Start/FilterConfig.cs`
- `Controllers/BaseController.cs`
- `Controllers/StudentsController.cs`
- `Controllers/CoursesController.cs`
- `Controllers/DepartmentsController.cs`
- `Controllers/InstructorsController.cs`
- `Controllers/NotificationsController.cs`
- `Controllers/HomeController.cs`

#### ConfigurationManager is not the recommended configuration approach in .NET Core {#configurationmanager-is-not-the-recommended-configuration-approach-in-net-core}

- **ID:** UA003  
- **Category:** Mandatory  
- **Effort:** Medium

System.Configuration.ConfigurationManager is used to read connection strings and app settings from Web.config. In .NET Core, Web.config is not the primary configuration mechanism.

**Recommendation:** Replace ConfigurationManager with Microsoft.Extensions.Configuration (IConfiguration) and use appsettings.json. Inject IConfiguration via dependency injection.

**API Change:** `System.Configuration.ConfigurationManager -> Microsoft.Extensions.Configuration.IConfiguration`

**Affected Files:**
- `Services/NotificationService.cs`
- `Global.asax.cs`

#### Global.asax / HttpApplication startup pattern not supported in .NET Core {#globalasax--httpapplication-startup-pattern-not-supported-in-net-core}

- **ID:** UA004  
- **Category:** Mandatory  
- **Effort:** Medium

The application relies on Global.asax and HttpApplication lifecycle for application startup and initialization. .NET Core uses the Program.cs / Startup.cs (or top-level statements) model.

**Recommendation:** Migrate startup code to Program.cs using the WebApplication builder pattern. Move database initialization to IHostedService or application startup.

**API Change:** `HttpApplication.Application_Start -> WebApplication.CreateBuilder / app.Run()`

**Affected Files:**
- `Global.asax.cs`
- `Global.asax`

#### packages.config NuGet format must be migrated to PackageReference {#packagesconfig-nuget-format-must-be-migrated-to-packagereference}

- **ID:** UA011  
- **Category:** Mandatory  
- **Effort:** High

The project uses the legacy packages.config NuGet package management format. Modern .NET SDK-style projects use PackageReference in .csproj.

**Recommendation:** Migrate from packages.config to PackageReference format using Visual Studio or the nuget migrate command. This is required before migrating to SDK-style project format.

**API Change:** `packages.config -> PackageReference in .csproj`

**Affected Files:**
- `packages.config`

#### Non-SDK project format (.csproj) must be converted to SDK-style {#non-sdk-project-format-csproj-must-be-converted-to-sdk-style}

- **ID:** UA012  
- **Category:** Mandatory  
- **Effort:** High

The project uses the old .NET Framework MSBuild project format which is not compatible with .NET 5+ SDK-style projects.

**Recommendation:** Convert to SDK-style project format by rewriting the .csproj file with <Project Sdk="Microsoft.NET.Sdk.Web">. Use the Upgrade Assistant tool for automated conversion.

**API Change:** `Old MSBuild format -> SDK-style <Project Sdk="Microsoft.NET.Sdk.Web">`

**Affected Files:**
- `ContosoUniversity.csproj`

### Recommended Issues (8)

#### Entity Framework Core 3.1 is end-of-life; upgrade to EF Core 8+ {#entity-framework-core-31-is-end-of-life-upgrade-to-ef-core-8}

- **ID:** UA005  
- **Category:** Recommended  
- **Effort:** Medium

The project uses EF Core 3.1.32 which reached end-of-life in December 2022. This version does not receive security patches.

**Recommendation:** Upgrade to EF Core 8.x (for .NET 8) or EF Core 9.x (for .NET 9+). Review breaking changes in the EF Core migration guide.

**API Change:** `Microsoft.EntityFrameworkCore 3.1.x -> Microsoft.EntityFrameworkCore 8.x`

**Affected Files:**
- `Data/SchoolContext.cs`
- `packages.config`

#### Web.config should be replaced with appsettings.json {#webconfig-should-be-replaced-with-appsettingsjson}

- **ID:** UA006  
- **Category:** Recommended  
- **Effort:** Low

Connection strings and application settings stored in Web.config are .NET Framework / IIS specific. Modern .NET applications use appsettings.json with optional environment-specific overrides.

**Recommendation:** Migrate connection strings and appSettings keys to appsettings.json. Use environment variables or Azure App Configuration for environment-specific settings.

**API Change:** `Web.config connectionStrings/appSettings -> appsettings.json`

**Affected Files:**
- `Web.config`

#### Manual dependency instantiation; no dependency injection container {#manual-dependency-instantiation-no-dependency-injection-container}

- **ID:** UA007  
- **Category:** Recommended  
- **Effort:** Low

The application manually instantiates SchoolContext and NotificationService in BaseController constructors rather than using a DI container.

**Recommendation:** Register services in Program.cs using builder.Services. Inject IDbContextFactory<SchoolContext> or SchoolContext directly through controller constructors. Register NotificationService as a scoped or singleton service.

**API Change:** `Manual new() instantiation -> IServiceProvider / constructor injection`

**Affected Files:**
- `Controllers/BaseController.cs`
- `Data/SchoolContextFactory.cs`

#### SQL Server LocalDB used for development - should use a configurable connection {#sql-server-localdb-used-for-development---should-use-a-configurable-connection}

- **ID:** UA008  
- **Category:** Recommended  
- **Effort:** Medium

The connection string points to SQL Server LocalDB which is a developer tool and not suitable for cloud or container deployments.

**Recommendation:** Use environment variables or Key Vault references for database connection strings. For cloud deployment target Azure SQL Database or Azure SQL Managed Instance.

**API Change:** `LocalDB connection string -> Environment variable / Azure Key Vault reference`

**Affected Files:**
- `Web.config`

#### Newtonsoft.Json should be replaced with System.Text.Json for new .NET targets {#newtonsoftjson-should-be-replaced-with-systemtextjson-for-new-net-targets}

- **ID:** UA013  
- **Category:** Recommended  
- **Effort:** Medium

The project uses Newtonsoft.Json 13.x. While this library still works in .NET Core, the built-in System.Text.Json is generally preferred for new development in .NET 5+.

**Recommendation:** Consider replacing Newtonsoft.Json with System.Text.Json where possible, particularly for simple serialization scenarios. Retain Newtonsoft.Json where complex serialization is needed.

**API Change:** `Newtonsoft.Json.JsonConvert -> System.Text.Json.JsonSerializer`

**Affected Files:**
- `Services/NotificationService.cs`
- `packages.config`

#### Logging uses System.Diagnostics.Debug.WriteLine instead of ILogger {#logging-uses-systemdiagnosticsdebugwriteline-instead-of-ilogger}

- **ID:** UA015  
- **Category:** Recommended  
- **Effort:** Low

The application uses System.Diagnostics.Debug.WriteLine and a custom LoggingService instead of the .NET standard ILogger<T> abstraction.

**Recommendation:** Replace with Microsoft.Extensions.Logging.ILogger<T>. This provides structured logging, multiple sinks (console, Application Insights, etc.), and configuration-based log levels.

**API Change:** `System.Diagnostics.Debug.WriteLine -> Microsoft.Extensions.Logging.ILogger<T>`

**Affected Files:**
- `Services/LoggingService.cs`
- `Controllers/BaseController.cs`
- `Services/NotificationService.cs`

#### File uploads stored on local filesystem; not suitable for cloud scale-out {#file-uploads-stored-on-local-filesystem-not-suitable-for-cloud-scale-out}

- **ID:** UA017  
- **Category:** Recommended  
- **Effort:** Medium

The CoursesController saves uploaded files to the server local filesystem (~/Uploads). This pattern does not work in horizontally scaled or ephemeral cloud environments.

**Recommendation:** Store uploaded files in Azure Blob Storage or another cloud object store. Use the Azure.Storage.Blobs SDK and return a URL/reference rather than a file path.

**API Change:** `Server.MapPath + File.Save -> Azure.Storage.Blobs.BlobContainerClient`

**Affected Files:**
- `Controllers/CoursesController.cs`

#### Table-per-Hierarchy (TPH) inheritance for Person entity needs verification with newer EF Core {#table-per-hierarchy-tph-inheritance-for-person-entity-needs-verification-with-newer-ef-core}

- **ID:** UA018  
- **Category:** Recommended  
- **Effort:** Low

The SchoolContext configures TPH inheritance for the Person entity (Student/Instructor). EF Core 7+ changed how TPH discriminators work by default.

**Recommendation:** After upgrading to EF Core 7+, verify TPH behavior still works as expected. Consider explicit discriminator column type configuration.

**API Change:** `EF Core 3.1 TPH -> EF Core 7+ TPH (verify discriminator configuration)`

**Affected Files:**
- `Data/SchoolContext.cs`

### Optional Issues (4)

#### CSRF protection uses legacy AntiForgeryToken pattern {#csrf-protection-uses-legacy-antiforgerytoken-pattern}

- **ID:** UA009  
- **Category:** Optional  
- **Effort:** Low

The application uses Html.AntiForgeryToken() and [ValidateAntiForgeryToken] which are ASP.NET MVC 5 patterns. ASP.NET Core MVC uses IAntiforgery service and [AutoValidateAntiforgeryToken].

**Recommendation:** After migration to ASP.NET Core, use the built-in antiforgery service. Consider using [AutoValidateAntiforgeryToken] at the application level.

**API Change:** `Html.AntiForgeryToken() -> IAntiforgery / [AutoValidateAntiforgeryToken]`

**Affected Files:**
- `Controllers/StudentsController.cs`
- `Controllers/CoursesController.cs`
- `Controllers/DepartmentsController.cs`
- `Controllers/InstructorsController.cs`

#### File upload uses HttpPostedFileBase which is not available in .NET Core {#file-upload-uses-httppostedfilebase-which-is-not-available-in-net-core}

- **ID:** UA010  
- **Category:** Optional  
- **Effort:** Low

The CoursesController uses HttpPostedFileBase for file uploads, which is a System.Web type.

**Recommendation:** Replace HttpPostedFileBase with IFormFile from Microsoft.AspNetCore.Http.

**API Change:** `System.Web.HttpPostedFileBase -> Microsoft.AspNetCore.Http.IFormFile`

**Affected Files:**
- `Controllers/CoursesController.cs`

#### Razor views (.cshtml) need minor syntax updates for ASP.NET Core {#razor-views-cshtml-need-minor-syntax-updates-for-aspnet-core}

- **ID:** UA014  
- **Category:** Optional  
- **Effort:** Low

Existing CSHTML views use ASP.NET MVC 5 HTML helpers (Html.BeginForm, Html.TextBoxFor, etc.). ASP.NET Core Razor uses Tag Helpers as the preferred approach.

**Recommendation:** Consider migrating HTML helpers to Tag Helpers (asp-action, asp-controller, asp-for attributes) for better readability. HTML helpers still work in ASP.NET Core.

**API Change:** `Html.BeginForm / Html.TextBoxFor -> Tag Helpers (asp-action, asp-for)`

**Affected Files:**
- `Views/Students/Create.cshtml`
- `Views/Students/Edit.cshtml`
- `Views/Courses/Create.cshtml`
- `Views/Departments/Create.cshtml`

#### No authentication or authorization implemented {#no-authentication-or-authorization-implemented}

- **ID:** UA016  
- **Category:** Optional  
- **Effort:** Low

The application has no authentication or authorization. Controllers and actions are all publicly accessible.

**Recommendation:** For production cloud deployment, add ASP.NET Core Identity, Azure AD / Entra ID authentication, or another identity provider. Use [Authorize] attributes to protect sensitive operations.

**API Change:** `No auth -> Microsoft.AspNetCore.Authentication / [Authorize] attribute`

**Affected Files:**
- `Controllers/BaseController.cs`

## Security Findings

| Severity | ID | Title | Category |
|---|---|---|---|
| mandatory | CVE-2024-0056 | Microsoft.Data.SqlClient and System.Data.SqlClient vulnerable to SQL Data Provider Security Feature Bypass | CVE |
| optional | CWE-477 | Use of Obsolete Function | Code Quality |
| potential | CWE-772 | Missing Release of Resource after Effective Lifetime | Code Quality |
| potential | CWE-1057 | Data Access Operations Outside of Expected Data Manager Component | Code Quality |
| optional | CWE-732 | Incorrect Permission Assignment for Critical Resource | Credentials & Secrets |
| potential | CWE-778 | Insufficient Logging | Credentials & Secrets |
| optional | CWE-22 | Improper Limitation of a Pathname to a Restricted Directory ('Path Traversal') | File & Path Security |
| mandatory | CWE-434 | Unrestricted Upload of File with Dangerous Type | File & Path Security |

### CVE-2024-0056 – Microsoft.Data.SqlClient and System.Data.SqlClient vulnerable to SQL Data Provider Security Feature Bypass {#cve-2024-0056-microsoftdatasqlclient-and-systemdatasqlclient-vulnerable-to-sql-data-provider-security-feature-bypass}

- **Severity:** mandatory  
- **Story Points:** 1  
- **Category:** CVE

[CVE-2024-0056](https://github.com/advisories/GHSA-98g6-xh36-x2p7): Microsoft.Data.SqlClient and System.Data.SqlClient vulnerable to SQL Data Provider Security Feature Bypass

Severity: HIGH

Affected dependencies:
  - Microsoft.Data.SqlClient@2.1.4 (declared at packages.config) — affected range: < 2.1.7

Recommended fix:
  - Upgrade Microsoft.Data.SqlClient to 2.1.7 or later

**Files:**
- `ContosoUniversity/packages.config`

### CWE-477 – Use of Obsolete Function {#cwe-477-use-of-obsolete-function}

- **Severity:** optional  
- **Story Points:** 1  
- **Category:** Code Quality

The code uses deprecated or obsolete functions, which suggests that the code has not been actively reviewed or maintained.

**Details:** NotificationService uses System.Messaging.MessageQueue, which is a Windows-only legacy API that is not supported in .NET Core/.NET 5+ and has been superseded by modern cloud messaging services. The class uses MessageQueue.Create(), MessageQueue.Exists(), _queue.Send(), _queue.Receive(), and XmlMessageFormatter — all of which are part of the obsolete System.Messaging namespace. This API has no cross-platform or cloud-portable equivalent and is explicitly identified as a migration blocker by Microsoft's modernization tooling.

**Files:**
- `ContosoUniversity/Services/NotificationService.cs`

### CWE-772 – Missing Release of Resource after Effective Lifetime {#cwe-772-missing-release-of-resource-after-effective-lifetime}

- **Severity:** potential  
- **Story Points:** 3  
- **Category:** Code Quality

The product does not release a resource after its effective lifetime has ended, i.e., after the resource is no longer needed.

**Details:** InstructorsController.Dispose(bool disposing) (line ~252) calls `db.Dispose()` directly, and then calls `base.Dispose(disposing)`. BaseController.Dispose(bool disposing) also calls `db?.Dispose()` on the same instance. This results in a double-dispose of the SchoolContext DbContext object. While EF Core's DbContext.Dispose() is idempotent (safe to call multiple times without throwing), calling Dispose() on an already-disposed context leaves it in a permanently unusable state, and any subsequent operation on the context after the first Dispose() call would throw ObjectDisposedException. Additionally, DepartmentsController.Dispose() has the same double-dispose pattern at line ~167.

**Files:**
- `ContosoUniversity/Controllers/InstructorsController.cs`

### CWE-1057 – Data Access Operations Outside of Expected Data Manager Component {#cwe-1057-data-access-operations-outside-of-expected-data-manager-component}

- **Severity:** potential  
- **Story Points:** 5  
- **Category:** Code Quality

The product uses a dedicated, central data manager component as required by design, but it contains code that performs data-access operations that do not use this data manager.

**Details:** All MVC controllers directly access the SchoolContext DbContext instance (db) inherited from BaseController, bypassing any service or repository layer. EF Core DbContext is intended to be the data access abstraction, but accessing it directly from controllers couples the presentation layer to the data access layer. For example, StudentsController.Index() executes LINQ queries directly against db.Students, CoursesController.Create() calls db.Courses.Add() and db.SaveChanges() directly, and InstructorsController.Edit() calls db.Entry().State = EntityState.Deleted directly. This violates separation of concerns and makes it impossible to unit test controllers without a real database connection.

**Files:**
- `ContosoUniversity/Controllers/StudentsController.cs`
- `ContosoUniversity/Controllers/CoursesController.cs`
- `ContosoUniversity/Controllers/DepartmentsController.cs`
- `ContosoUniversity/Controllers/InstructorsController.cs`
- `ContosoUniversity/Controllers/NotificationsController.cs`

### CWE-732 – Incorrect Permission Assignment for Critical Resource {#cwe-732-incorrect-permission-assignment-for-critical-resource}

- **Severity:** optional  
- **Story Points:** 5  
- **Category:** Credentials & Secrets

The product specifies permissions for a security-critical resource in a way that allows that resource to be read or modified by unintended actors.

**Details:** NotificationService constructor (line 23) grants 'Everyone' full control over the MSMQ notification queue when it is created: `_queue.SetPermissions("Everyone", MessageQueueAccessRights.FullControl)`. This allows any user or process on the Windows machine — including unauthenticated network users if the queue is remotely accessible — to read, write, delete, and administer the message queue. An attacker could inject arbitrary notification messages, drain the queue, or cause denial-of-service by flooding it. The queue should be restricted to the application's service account identity.

**Files:**
- `ContosoUniversity/Services/NotificationService.cs`

### CWE-778 – Insufficient Logging {#cwe-778-insufficient-logging}

- **Severity:** potential  
- **Story Points:** 3  
- **Category:** Credentials & Secrets

When a security-critical event occurs, the product either does not record the event or omits important details about the event when logging it.

**Details:** Security-relevant events are not logged to a persistent, observable log store: (1) BaseController.SendEntityNotification() catches all exceptions from notification delivery and silently swallows them with only a Debug.WriteLine() call — these failures are invisible in production environments where the debugger is not attached; (2) NotificationService.SendNotification() and ReceiveNotification() catch exceptions and log only to Debug.WriteLine(), which is not written to any log file or monitoring system in production; (3) There is no logging of entity create/update/delete operations, authentication events, authorization decisions, or file upload events with user identity, timestamp, or IP address. The application has no structured logging framework (Serilog, NLog, Microsoft.Extensions.Logging providers) configured.

**Files:**
- `ContosoUniversity/Controllers/BaseController.cs`
- `ContosoUniversity/Services/NotificationService.cs`

### CWE-22 – Improper Limitation of a Pathname to a Restricted Directory ('Path Traversal') {#cwe-22-improper-limitation-of-a-pathname-to-a-restricted-directory-path-traversal}

- **Severity:** optional  
- **Story Points:** 8  
- **Category:** File & Path Security

The product uses external input to construct a pathname that is intended to identify a file or directory that is located underneath a restricted parent directory, but the product does not properly neutralize special elements within the pathname that can cause the pathname to resolve to a location that is outside of the restricted directory.

**Details:** In CoursesController.DeleteConfirmed (line ~229), the file path is reconstructed using `Server.MapPath(course.TeachingMaterialImagePath)` where `TeachingMaterialImagePath` is a value read from the database — but during the Create/Edit POST, the `[Bind]` attribute includes `TeachingMaterialImagePath` in the bound field list, meaning an attacker could submit a crafted value such as `~/../../sensitive/file` in the POST body, which could then be persisted to the database and subsequently used in `Server.MapPath()` to resolve a path outside the web root. There is no validation or sanitization of the `TeachingMaterialImagePath` field value before it is stored or used in file system operations.

**Files:**
- `ContosoUniversity/Controllers/CoursesController.cs`

### CWE-434 – Unrestricted Upload of File with Dangerous Type {#cwe-434-unrestricted-upload-of-file-with-dangerous-type}

- **Severity:** mandatory  
- **Story Points:** 8  
- **Category:** File & Path Security

The product allows the upload or transfer of dangerous file types that are automatically processed within its environment.

**Details:** CoursesController.Create (line ~55) and CoursesController.Edit (line ~128) accept file uploads and validate the file type using only the file extension (`Path.GetExtension(teachingMaterialImage.FileName).ToLower()`). This check can be bypassed: (1) the MIME type / Content-Type header is not verified against the actual file content, allowing an attacker to upload a file with a valid image extension (e.g., `.jpg`) that contains executable script content; (2) on some IIS configurations, double extensions (e.g., `shell.asp;.jpg`) can bypass extension-only checks. The allowed extension list (`jpg`, `jpeg`, `png`, `gif`, `bmp`) is client-supplied filename-based only — no magic byte / file signature validation is performed.

**Files:**
- `ContosoUniversity/Controllers/CoursesController.cs`

## Codebase Insights

The following supplementary analysis documents were generated alongside this report:

- [API & Service Communication Contracts](facts/api-service-contracts.md) – Endpoint inventory, communication patterns, and security posture
- [Architecture Diagram](facts/architecture-diagram.md) – Application layers, technology stack, and component relationships
- [Assessment Overview](facts/assessment-overview.md) – Navigation index for all supplementary documents
- [Business Workflows](facts/business-workflows.md) – Domain entities, primary workflows, and business rules
- [Configuration Inventory](facts/configuration-inventory.md) – All configuration sources, secrets assessment, and framework versions
- [Data Architecture](facts/data-architecture.md) – Entity model, database configuration, and PII classification
- [Dependency Map](facts/dependency-map.md) – All external NuGet dependencies grouped by functional category

## Next Steps

Based on the assessment findings, the recommended migration path is:

1. **Migrate project format** – Convert from legacy MSBuild `.csproj` to SDK-style and migrate `packages.config` to `PackageReference`
2. **Migrate to ASP.NET Core** – Replace `System.Web.*`, `Global.asax`, and `Web.config` with ASP.NET Core MVC, `Program.cs`, and `appsettings.json`
3. **Replace MSMQ** – Migrate `System.Messaging` to Azure Service Bus or another cloud-native messaging service
4. **Upgrade Entity Framework Core** – Move from EF Core 3.1 (EOL) to EF Core 8.x
5. **Implement secrets management** – Move connection strings to Azure Key Vault / environment variables
6. **Add authentication** – Implement ASP.NET Core Identity or Entra ID authentication
7. **Upgrade Microsoft.Data.SqlClient** – Fix CVE-2024-0056 by upgrading to ≥ 2.1.7
8. **Address security findings** – Fix CWE-434 (file upload validation), CWE-22 (path traversal), CWE-732 (MSMQ permissions)
