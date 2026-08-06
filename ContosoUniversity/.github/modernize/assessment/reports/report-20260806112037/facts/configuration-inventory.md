# Configuration & Externalized Settings Inventory

The project uses classic ASP.NET configuration files with a small set of externalized app settings and connection strings.

## Configuration Sources

| Source | Type | Path/Location | Notes |
|---|---|---|---|
| Web.config | XML config | `ContosoUniversity/Web.config` | Primary runtime settings and connection strings |
| Web.Debug.config | Transform | `ContosoUniversity/Web.Debug.config` | Debug transform placeholder |
| Web.Release.config | Transform | `ContosoUniversity/Web.Release.config` | Release transform placeholder |
| csproj properties | Build config | `ContosoUniversity/ContosoUniversity.csproj` | Build profile settings and IIS Express metadata |

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies/Plugins |
|---|---|---|---|
| Debug | `Configuration=Debug` | Local debugging, symbols, no optimization | Standard MSBuild targets |
| Release | `Configuration=Release` | Optimized production build | Standard MSBuild targets |

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides |
|---|---|---|---|
| default | IIS/App start defaults | `Web.config` | `DefaultConnection`, request limits, app settings |

## Properties Inventory

| Property Key | Default | Profiles | Source |
|---|---|---|---|
| `ConnectionStrings:DefaultConnection` | LocalDB connection string | default | `Web.config` |
| `webpages:Version` | `3.0.0.0` | default | `Web.config` |
| `webpages:Enabled` | `false` | default | `Web.config` |
| `ClientValidationEnabled` | `true` | default | `Web.config` |
| `UnobtrusiveJavaScriptEnabled` | `true` | default | `Web.config` |
| `NotificationQueuePath` | `.\Private$\ContosoUniversityNotifications` | default | `Web.config` |
| `httpRuntime:maxRequestLength` | `10240` | default | `Web.config` |
| `httpRuntime:executionTimeout` | `3600` | default | `Web.config` |
| `requestLimits:maxAllowedContentLength` | `10485760` | default | `Web.config` |

## Startup Parameters & Resource Requirements

| Service | JVM/Runtime Options | Memory | Instance Count |
|---|---|---|---|
| ContosoUniversity | .NET Framework runtime via IIS | Not specified in repo | Not specified in repo |

## Startup Dependency Chain

1. IIS hosts `MvcApplication`.
2. `Application_Start` registers filters/routes/bundles.
3. `InitializeDatabase` creates `SchoolContext` and executes `DbInitializer.Initialize`.
4. Controllers instantiate `NotificationService`, which checks/creates MSMQ queue.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage (masked) |
|---|---|---|
| `ConnectionStrings:DefaultConnection` | DB connection string | `Web.config` (`Integrated Security=True`, no password shown) |

### Secrets Provisioning Workflow

Configuration values are loaded from `Web.config` at runtime using `ConfigurationManager`. No external key vault, secret manager, or managed identity workflow was detected in the repository.

## Feature Flags

| Flag Name | Default | Controlled By |
|---|---|---|
| `webpages:Enabled` | false | `Web.config` |

## Framework & Runtime Versions

| Component | Version | Source |
|---|---|---|
| .NET Framework target | v4.8 | `ContosoUniversity.csproj` |
| ASP.NET MVC | 5.2.9 | `packages.config` |
| Entity Framework Core | 3.1.32 | `packages.config` |
| Microsoft.Data.SqlClient | 2.1.4 | `packages.config` |
| Newtonsoft.Json | 13.0.3 | `packages.config` |
