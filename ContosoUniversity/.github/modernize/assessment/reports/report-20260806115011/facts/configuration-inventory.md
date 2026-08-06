# Configuration & Externalized Settings Inventory

ContosoUniversity has a single configuration source (`Web.config`) with no environment-specific overrides, no secrets management tooling, and no feature flags — all settings are hardcoded in the XML configuration file checked into source control.

## Configuration Sources

| Source | Type | Path/Location | Notes |
|---|---|---|---|
| Web.config | XML Application Config | `ContosoUniversity/Web.config` | Primary (and only) configuration file; contains connection strings, appSettings, assembly binding redirects |
| Properties/AssemblyInfo.cs | Assembly metadata | `ContosoUniversity/Properties/AssemblyInfo.cs` | Assembly version, company, product info |
| BundleConfig.cs | Code-based config | `App_Start/BundleConfig.cs` | Script and style bundle definitions |
| RouteConfig.cs | Code-based config | `App_Start/RouteConfig.cs` | MVC routing table |
| FilterConfig.cs | Code-based config | `App_Start/FilterConfig.cs` | Global MVC filter registrations |

No `appsettings.json`, `appsettings.{Environment}.json`, `launchSettings.json`, Docker Compose, Kubernetes ConfigMaps, Spring Cloud Config server, Azure App Configuration, or HashiCorp Vault references are present.

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies / Plugins |
|---|---|---|---|
| Debug | Default in Visual Studio / MSBuild | Development build; includes debug symbols, no optimisation | `DebugSymbols=true`, `DebugType=full`, `Optimize=false`, constants: `DEBUG;TRACE` |
| Release | Manual (`/p:Configuration=Release` or VS publish) | Production packaging; PDB-only symbols, optimised | `DebugSymbols=true`, `DebugType=pdbonly`, `Optimize=true`, constants: `TRACE` |

No Maven/Gradle profiles, conditional MSBuild targets, or webpack/vite configurations are present.

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides |
|---|---|---|---|
| Default (single profile) | N/A — no profile system configured | `Web.config` only | No profile-specific overrides |

The application has no environment-specific configuration files (no `Web.Debug.config` / `Web.Release.config` transforms are configured beyond assembly binding redirects). All environment-specific values (e.g., the database connection string) must be manually edited in `Web.config` before each deployment. This is a significant operational risk for cloud deployment, where environment-specific configuration should be injected via environment variables or a configuration service.

## Properties Inventory

### Connection Strings

| Property Key | Default Value | Environment Override | Notes |
|---|---|---|---|
| DefaultConnection | `Data Source=(LocalDb)\MSSQLLocalDB;Initial Catalog=ContosoUniversityNoAuthEFCore;Integrated Security=True;MultipleActiveResultSets=True` | Not configured | Hardcoded LocalDB path; must be replaced for any non-developer environment |

### Application Settings (appSettings)

| Property Key | Default Value | Type | Notes |
|---|---|---|---|
| `webpages:Version` | `3.0.0.0` | string | ASP.NET WebPages version binding |
| `webpages:Enabled` | `false` | bool | Disables WebMatrix WebPages routing |
| `ClientValidationEnabled` | `true` | bool | Enables MVC client-side validation |
| `UnobtrusiveJavaScriptEnabled` | `true` | bool | Enables jQuery unobtrusive validation |
| `NotificationQueuePath` | `.\Private$\ContosoUniversityNotifications` | string | MSMQ private queue path; hardcoded Windows path; not cloud-portable |

### HTTP Runtime Settings (`system.web`)

| Setting | Value | Notes |
|---|---|---|
| `compilation debug` | `true` | Should be `false` in production |
| `targetFramework` | `4.8` | .NET Framework 4.8 |
| `maxRequestLength` | `10240` KB (10 MB) | Maximum upload size for HTTP requests |
| `executionTimeout` | `3600` seconds | 1-hour request timeout — unusually long; review for security |

### IIS / Web Server Settings (`system.webServer`)

| Setting | Value | Notes |
|---|---|---|
| `maxAllowedContentLength` | `10485760` bytes (10 MB) | IIS request size limit; matches `maxRequestLength` |

## Startup Parameters & Resource Requirements

| Item | Value | Notes |
|---|---|---|
| Runtime | .NET Framework 4.8 | Requires Windows; CLR 4.0 |
| Web Server | IIS / IIS Express (dev) | Requires Windows; not compatible with Kestrel without migration |
| JVM/CLR heap | Not explicitly configured | Default CLR garbage collector; no explicit heap tuning |
| Process model | IIS application pool | Single worker process; no horizontal scaling configuration |
| Request limits | maxRequestLength: 10 MB; executionTimeout: 3600 s | Defined in Web.config |

No Docker, Kubernetes, Azure App Service, or cloud-specific resource configuration is present.

## Startup Dependency Chain

```
Application Pool Start
  → Global.asax Application_Start()
      → AreaRegistration.RegisterAllAreas()
      → FilterConfig.RegisterGlobalFilters()
      → RouteConfig.RegisterRoutes()
      → BundleConfig.RegisterBundles()
      → InitializeDatabase()
          → ConfigurationManager.ConnectionStrings["DefaultConnection"]
          → new DbContextOptionsBuilder<SchoolContext>().UseSqlServer(connectionString)
          → DbInitializer.Initialize(context)
              → context.Database.EnsureCreated()   ← blocks on SQL Server availability
              → Seed data insertion (if empty)
  → Application ready (first request served)
```

**Critical dependency**: SQL Server must be available before the application can finish starting. There are no retry mechanisms, timeout overrides, readiness probes, or health-check endpoints. If the database is unavailable, `EnsureCreated()` will throw an unhandled exception during startup. For cloud/container deployments, a startup retry or readiness probe must be added.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Current Storage | Risk |
|---|---|---|---|
| `DefaultConnection` connection string | Database connection | Plain text in `Web.config` — committed to source control | HIGH — credentials (if using SQL auth) or server name exposed in repo |
| `NotificationQueuePath` | MSMQ queue path | Plain text in `Web.config` | LOW — non-secret path but environment-specific |

> **Note**: The current connection string uses Windows Integrated Security (no username/password visible), but the server name and database name are exposed in source control. In cloud environments using SQL authentication, a plaintext password in `Web.config` would be a critical secret leak.

### Secrets Provisioning Workflow

**Current state**: No secrets management workflow exists. All configuration, including the database connection string, is stored in plain text in `Web.config` and committed to source control. There is no Azure Key Vault, HashiCorp Vault, AWS Secrets Manager, environment variable injection, or any other secrets management mechanism.

**Recommended workflow for cloud migration**:
1. Remove all secrets from `Web.config` / source control.
2. Store the database connection string in Azure Key Vault or as an Azure App Service application setting (environment variable).
3. Use a system-assigned managed identity on the App Service with `get`/`list` permissions on the Key Vault secret.
4. Reference secrets via `@Microsoft.KeyVault(SecretUri=...)` in App Service configuration or use `Azure.Security.KeyVault.Secrets` SDK at runtime.
5. For the MSMQ replacement (Azure Service Bus), store the connection string / SAS token in Key Vault.

## Feature Flags

No feature flag framework is configured. The `FilterConfig.cs` contains a commented-out `AuthorizeAttribute` that hints at an in-progress feature, but no feature toggle mechanism (LaunchDarkly, `Microsoft.FeatureManagement`, `IFeatureManager`, etc.) is present.

| Flag / Toggle | Current State | Notes |
|---|---|---|
| Global authorization (commented out) | Disabled (commented out) | `// filters.Add(new AuthorizeAttribute())` in FilterConfig.cs |

## Framework & Runtime Versions

| Component | Version | Source |
|---|---|---|
| .NET Framework | 4.8 | `ContosoUniversity.csproj` → `TargetFrameworkVersion` |
| ASP.NET MVC | 5.2.9 | `packages.config` → `Microsoft.AspNet.Mvc` |
| ASP.NET Razor | 3.2.9 | `packages.config` → `Microsoft.AspNet.Razor` |
| ASP.NET WebPages | 3.2.9 | `packages.config` → `Microsoft.AspNet.WebPages` |
| Entity Framework Core | 3.1.32 | `packages.config` → `Microsoft.EntityFrameworkCore` |
| EF Core SQL Server | 3.1.32 | `packages.config` → `Microsoft.EntityFrameworkCore.SqlServer` |
| Microsoft.Data.SqlClient | 2.1.4 | `packages.config` → `Microsoft.Data.SqlClient` |
| System.Messaging (MSMQ) | Built-in (NET 4.8) | .NET Framework built-in |
| Newtonsoft.Json | 13.0.3 | `packages.config` → `Newtonsoft.Json` |
| Bootstrap | 5.3.3 | `packages.config` → `bootstrap` |
| jQuery | 3.7.1 | `packages.config` → `jQuery` |
| jQuery Validation | 1.21.0 | `packages.config` → `jQuery.Validation` |
| Microsoft.Identity.Client (MSAL) | 4.21.1 | `packages.config` → `Microsoft.Identity.Client` |
| System.Web.Optimization | 1.1.3 | `packages.config` → `Microsoft.AspNet.Web.Optimization` |
| Build tool | MSBuild (Visual Studio 2017+) | `ContosoUniversity.csproj` ToolsVersion="15.0" |
| Package manager | NuGet (packages.config format) | Legacy format; requires migration to PackageReference |
