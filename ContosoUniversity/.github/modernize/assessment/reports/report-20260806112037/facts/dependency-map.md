# Dependency Map

This dependency map summarizes the declared NuGet dependencies for ContosoUniversity and groups them by functional role.

## Dependencies

```mermaid
flowchart LR
    App["ContosoUniversity"]

    subgraph Web["Web Frameworks"]
        Mvc["Microsoft.AspNet.Mvc 5.2.9"]
        Razor["Microsoft.AspNet.Razor 3.2.9"]
        WebPages["Microsoft.AspNet.WebPages 3.2.9"]
        WebOpt["Microsoft.AspNet.Web.Optimization 1.1.3"]
    end
    subgraph DB["Database / ORM"]
        EFCore["Microsoft.EntityFrameworkCore 3.1.32"]
        EFSql["Microsoft.EntityFrameworkCore.SqlServer 3.1.32"]
        SqlClient["Microsoft.Data.SqlClient 2.1.4"]
    end
    subgraph Cache["Caching"]
        MemCache["Microsoft.Extensions.Caching.Memory 3.1.32"]
    end
    subgraph Log["Logging"]
        ExtLog["Microsoft.Extensions.Logging 3.1.32"]
    end
    subgraph Sec["Security"]
        Msal["Microsoft.Identity.Client 4.21.1"]
    end
    subgraph Util["Utilities"]
        Json["Newtonsoft.Json 13.0.3"]
        JQuery["jQuery 3.7.1"]
        Bootstrap["bootstrap 5.3.3"]
        WebGrease["WebGrease 1.5.2"]
    end

    App -->|"web"| Web
    App -->|"persistence"| DB
    App -->|"caching"| Cache
    App -->|"logging"| Log
    App -->|"security"| Sec
    App -->|"utilities"| Util
```

### Dependency Summary

| Category | Count | Key Libraries | Notes |
|---|---:|---|---|
| Web Frameworks | 4 | Microsoft.AspNet.Mvc, Razor, WebPages | Classic ASP.NET MVC stack |
| Database / ORM | 3 | EF Core, EF Core SqlServer, SqlClient | EF Core 3.1 with SQL Server provider |
| Caching | 1 | Microsoft.Extensions.Caching.Memory | In-process cache abstraction available |
| Logging | 1 | Microsoft.Extensions.Logging | Logging primitives included |
| Security | 1 | Microsoft.Identity.Client | Identity client library present |
| Utilities | 4 | Newtonsoft.Json, jQuery, bootstrap, WebGrease | UI/runtime utilities |

### Version & Compatibility Risks

The application targets .NET Framework 4.8 and relies on several older libraries (for example EF Core 3.1 and ASP.NET MVC 5.x) that are out of mainstream innovation paths and may require upgrades for long-term support and modernization.

### Notable Observations

- Mixes classic ASP.NET MVC packages with EF Core 3.1 data access libraries.
- Uses packages.config dependency management rather than SDK-style `PackageReference`.
- Includes front-end packages directly in NuGet package graph.
- SQL connectivity relies on `Microsoft.Data.SqlClient` 2.x runtime binaries.

## Test Dependencies

| Framework | Version | Notes |
|---|---|---|
| N/A | N/A | No test-scoped dependency declarations detected in `packages.config` |

Total test-scope dependencies: 0

No test dependencies detected.
