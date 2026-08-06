# Modernization Plan: ContosoUniversity Azure Migration

**Project**: ContosoUniversity

---

## Technical Framework

- **Language**: C# / .NET Framework 4.8
- **Framework**: ASP.NET MVC 5.2.9
- **Build Tool**: MSBuild / NuGet
- **Database**: SQL Server (LocalDB / MSSQLLocalDB) via Entity Framework Core 3.1.32
- **Key Dependencies**: Microsoft.AspNet.Mvc 5.2.9, Microsoft.EntityFrameworkCore.SqlServer 3.1.32, System.Messaging (MSMQ), ASP.NET Web Optimization, Newtonsoft.Json 13.0.3

---

## Overview

> This migration modernizes the ContosoUniversity ASP.NET MVC 4.8 application from on-premises Windows infrastructure to Azure cloud-native services. The application currently relies on SQL Server LocalDB for data persistence, MSMQ (Microsoft Message Queuing) for the real-time admin notification system, and the local file system (`/Uploads/TeachingMaterials/`) for teaching material image storage — all of which are Windows-specific dependencies incompatible with containerized or cloud hosting. The new architecture will:
>
> - Replace MSMQ with **Azure Service Bus** for reliable, cloud-native message queuing and admin notifications, enabling the app to run outside Windows
> - Replace local file system storage with **Azure Blob Storage** for teaching material image uploads, providing durable and scalable file management
> - Migrate the SQL Server database to **Azure SQL Database** with Managed Identity authentication, eliminating embedded connection string credentials
> - Upgrade the project from .NET Framework 4.8 to **.NET 10** (latest LTS) to enable modern Azure SDK support and container deployment
> - Deploy the modernized application to **Azure Container Apps**
>
> The migration follows a phased approach: upgrade the runtime first, then migrate each service dependency to its Azure equivalent, remediate security vulnerabilities, and finally deploy to Azure Container Apps.

---

## Migration Impact Summary

| Application          | Original Service          | New Azure Service            | Authentication    | Comments                                      |
|----------------------|---------------------------|------------------------------|-------------------|-----------------------------------------------|
| ContosoUniversity    | SQL Server (LocalDB)      | Azure SQL Database           | Managed Identity  | EF Core provider updated; passwordless auth   |
| ContosoUniversity    | MSMQ (System.Messaging)   | Azure Service Bus            | Managed Identity  | Notification send/receive migrated to SB      |
| ContosoUniversity    | Local File System Uploads | Azure Blob Storage           | Managed Identity  | Teaching material images stored in blob       |
| ContosoUniversity    | .NET Framework 4.8        | .NET 10 (net10.0)            | N/A               | SDK-style project conversion required         |
| ContosoUniversity    | On-premises IIS           | Azure Container Apps         | Managed Identity  | Containerized deployment via Docker           |

---

## Open Questions & Questionnaire

- [x] Q: What is the target .NET version? → A: .NET 10 (latest LTS), per dotnet-upgrade-guideline
- [x] Q: What is the target Azure deployment service? → A: Azure Container Apps (default)
- [x] Q: What authentication method for Azure services? → A: Managed Identity (default)
- [x] Q: Should integration tests be included? → A: No — skipped as not explicitly requested
- [x] Q: Should infrastructure (IaC) be provisioned? → A: No — not explicitly requested
