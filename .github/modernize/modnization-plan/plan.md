# Modernization Plan: ContosoUniversity Azure Modernization

**Project**: ContosoUniversity

---

## Technical Framework

- **Language**: C# / .NET Framework 4.8
- **Framework**: ASP.NET MVC 5 with Entity Framework Core 3.1
- **Build Tool**: MSBuild solution with NuGet `packages.config`
- **Database**: SQL Server LocalDB / SQL Server via `DefaultConnection`
- **Key Dependencies**: Microsoft.EntityFrameworkCore.SqlServer, Microsoft.Data.SqlClient, System.Messaging, Newtonsoft.Json

---

## Overview

> This migration modernizes the ContosoUniversity web application from its current
> on-premises, Windows-centric runtime to Azure-aligned services. The application
> currently depends on local SQL Server connectivity, MSMQ-based notifications,
> and local file system storage for teaching material uploads. The new
> architecture will:
>
> - Move relational data storage to Azure SQL Database for managed operations
>   and cloud-hosted persistence
> - Replace MSMQ notifications with Azure Service Bus so messaging no longer
>   depends on Windows-only infrastructure
> - Move uploaded teaching material assets to Azure Blob Storage and deploy the
>   application to Azure Container Apps for cloud-native hosting
>
> The migration follows a phased approach that captures a baseline first, then
> updates application dependencies sequentially, verifies the migrated behavior,
> and completes security review before deployment.

---

## Migration Impact Summary

| App | Original | Azure | Auth | Notes |
|-----|----------|-------|------|-------|
| CU | SQL Server | Azure SQL DB | Managed identity | Replace LocalDB |
| CU | MSMQ | Azure Service Bus | Managed identity | Keep admin alerts |
| CU | Local uploads | Azure Blob | Managed identity | Keep file flows |
| CU | Windows hosting | Container Apps | Entra/MI | Cloud deployment |

---

## Open Questions & Questionnaire

- [x] Q: Should the plan include environment/infrastructure provisioning? → A: No — no infrastructure provisioning or repo-defined IaC was found, so the plan focuses on code migration and deployment work.
- [x] Q: Should the plan include integration testing to verify migrated services? → A: Yes — Mock mode, because no provisioned Azure environment or `infra/` configuration was provided.
- [x] Q: Should the plan include a security scan and CVE remediation task? → A: Yes — include security/CVE remediation.
- [x] Q: Which Azure deployment target should the plan use? → A: Azure Container Apps (default), matching the repository modernization context.
