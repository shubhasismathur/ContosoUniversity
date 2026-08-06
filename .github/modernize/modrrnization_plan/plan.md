# Modernization Plan: ContosoUniversity Azure Migration

**Project**: ContosoUniversity

---

## Technical Framework

- **Language**: C# / .NET Framework 4.8
- **Framework**: ASP.NET MVC 5.2.9
- **Build Tool**: MSBuild / NuGet
- **Database**: SQL Server (LocalDB in development) via Entity Framework Core 3.1.32
- **Key Dependencies**: Microsoft.EntityFrameworkCore 3.1.32, Microsoft.AspNet.Mvc 5.2.9, System.Messaging (MSMQ), Newtonsoft.Json 13.0.3, Microsoft.Identity.Client 4.21.1

---

## Overview

> This migration modernizes the ContosoUniversity ASP.NET MVC 4.8 application to run on Azure. The application currently runs on .NET Framework 4.8 with an ASP.NET MVC 5 front-end, uses SQL Server (LocalDB) for data persistence via Entity Framework Core, and relies on MSMQ (System.Messaging) for the notification queue service. The new architecture will:
>
> - Upgrade the runtime from .NET Framework 4.8 to .NET 10 (latest LTS), converting to an SDK-style project to unlock modern Azure SDK support and cloud-native deployment.
> - Migrate the SQL Server database connection to Azure SQL Database using Managed Identity (passwordless authentication), eliminating hard-coded connection strings.
> - Migrate the MSMQ-based notification service to Azure Service Bus with Managed Identity, providing a cloud-native, scalable messaging solution.
> - Remediate all known CVEs in project dependencies to ensure a secure baseline before deployment.
>
> The migration follows a sequential approach: runtime upgrade first, then service migrations to Azure, then security remediation.

---

## Migration Impact Summary

| Application          | Original Service         | New Azure Service         | Authentication     | Comments                          |
|----------------------|--------------------------|---------------------------|--------------------|-----------------------------------|
| ContosoUniversity    | SQL Server (LocalDB)     | Azure SQL Database        | Managed Identity   | EF Core data access layer         |
| ContosoUniversity    | MSMQ (System.Messaging)  | Azure Service Bus         | Managed Identity   | NotificationService queue         |
| ContosoUniversity    | .NET Framework 4.8       | .NET 10 (net10.0)         | N/A                | SDK-style project conversion      |

---

## Open Questions & Questionnaire

- [x] Q: What is the target .NET version? → A: .NET 10 (latest LTS), per dotnet-upgrade-guideline.md since .NET Framework 4.8 is EOL for mainstream support path and requires SDK-style conversion for Azure SDK compatibility.
- [x] Q: What authentication method should be used for Azure services? → A: Managed Identity (DefaultAzureCredential) for all Azure services (Azure SQL Database, Azure Service Bus).
- [x] Q: Is integration testing requested? → A: No explicit request — skipping integration test task.
- [x] Q: Is infrastructure provisioning requested? → A: No explicit request — skipping infrastructure task.
- [x] Q: What is the deployment target? → A: No explicit deployment request — skipping deployment task.
