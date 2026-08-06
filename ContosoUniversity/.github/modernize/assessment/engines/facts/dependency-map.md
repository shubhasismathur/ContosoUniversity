# Dependency Map

ContosoUniversity is an ASP.NET MVC 5 web application targeting .NET Framework 4.8 with 46 declared NuGet dependencies across web, data access, messaging, security, and utility categories.

## Dependencies

```mermaid
flowchart LR
    App["ContosoUniversity\n(.NET Framework 4.8)"]

    subgraph Web["Web Frameworks"]
        ASPMVC["ASP.NET MVC 5.2.9"]
        Razor["ASP.NET Razor 3.2.9"]
        WebPages["ASP.NET WebPages 3.2.9"]
        Bootstrap["Bootstrap 5.3.3"]
        jQuery["jQuery 3.7.1"]
        jQueryVal["jQuery Validation 1.21.0"]
        jQueryUnobtrus["jQuery Unobtrusive Validation 4.0.0"]
        Optimization["Web.Optimization 1.1.3"]
        WebGrease["WebGrease 1.5.2"]
        Modernizr["Modernizr 2.6.2"]
        Antlr["Antlr 3.4.1"]
    end
    subgraph DB["Database / ORM"]
        EFCore["EF Core 3.1.32"]
        EFCoreRelational["EF Core Relational 3.1.32"]
        EFCoreSqlServer["EF Core SqlServer 3.1.32"]
        EFCoreAbstractions["EF Core Abstractions 3.1.32"]
        EFCoreTools["EF Core Tools 3.1.32"]
        SqlClient["Microsoft.Data.SqlClient 2.1.4"]
        SqlClientSNI["SqlClient SNI Runtime 2.1.1"]
    end
    subgraph Messaging["Messaging"]
        MSMQBuiltin["System.Messaging (MSMQ)\nbuilt-in .NET Framework"]
    end
    subgraph Sec["Security"]
        MSAL["Microsoft.Identity.Client 4.21.1"]
    end
    subgraph Extensions["Microsoft.Extensions"]
        DI["DI Abstractions 3.1.32"]
        DIImpl["DI 3.1.32"]
        Logging["Logging Abstractions 3.1.32"]
        LoggingImpl["Logging 3.1.32"]
        Config["Configuration 3.1.32"]
        ConfigAbstr["Configuration Abstractions 3.1.32"]
        ConfigBinder["Configuration Binder 3.1.32"]
        Caching["Caching Abstractions 3.1.32"]
        CachingMemory["Caching Memory 3.1.32"]
        Options["Options 3.1.32"]
        Primitives["Primitives 3.1.32"]
    end
    subgraph Util["Utilities"]
        Newtonsoft["Newtonsoft.Json 13.0.3"]
        WebInfra["Microsoft.Web.Infrastructure 2.0.1"]
        Roslyn["CodeDom DotNetCompilerPlatform 2.0.1"]
        NetStandard["NETStandard.Library 2.0.3"]
        Buffers["System.Buffers 4.5.1"]
        ImmutableCollections["System.Collections.Immutable 1.7.1"]
        ComponentAnnotations["System.ComponentModel.Annotations 4.7.0"]
        DiagSource["System.Diagnostics.DiagnosticSource 4.7.1"]
        Memory["System.Memory 4.5.4"]
        Numerics["System.Numerics.Vectors 4.5.0"]
        CompilerUnsafe["System.Runtime.CompilerServices.Unsafe 4.5.3"]
        TasksExtensions["System.Threading.Tasks.Extensions 4.5.4"]
        BclAsync["Microsoft.Bcl.AsyncInterfaces 1.1.1"]
        BclHashCode["Microsoft.Bcl.HashCode 1.1.1"]
    end

    App -->|"web framework"| Web
    App -->|"data access"| DB
    App -->|"messaging"| Messaging
    App -->|"security"| Sec
    App -->|"platform extensions"| Extensions
    App -->|"utilities"| Util
    EFCoreAbstractions -.->|"used by"| EFCore
    DI -.->|"used by"| EFCore
    Logging -.->|"used by"| EFCore
```

### Dependency Summary

| Category | Count | Key Libraries | Notes |
|---|---|---|---|
| Web Frameworks | 11 | ASP.NET MVC 5.2.9, Bootstrap 5.3.3, jQuery 3.7.1, Web.Optimization 1.1.3 | Legacy MVC stack; System.Web not compatible with .NET Core |
| Database / ORM | 7 | EF Core 3.1.32 (EOL), Microsoft.Data.SqlClient 2.1.4 | EF Core 3.1 is end-of-life since Dec 2022 |
| Messaging | 1 | System.Messaging (MSMQ) — built-in | Windows-only; not available on .NET Core; major migration blocker |
| Security | 1 | Microsoft.Identity.Client 4.21.1 | MSAL present but no auth is implemented |
| Microsoft.Extensions | 11 | DI, Logging, Configuration, Caching, Options 3.1.32 | All at EOL 3.1.32; must be upgraded |
| Utilities | 14 | Newtonsoft.Json 13.0.3, System.Memory 4.5.4, NETStandard.Library 2.0.3 | Several polyfill packages only needed on .NET Framework |

### Version & Compatibility Risks

The most significant risk is the **ASP.NET MVC 5 / System.Web dependency** — the entire web stack (System.Web.Mvc, System.Web.Optimization, System.Web.Routing) is incompatible with .NET Core and must be replaced with ASP.NET Core MVC. **Entity Framework Core 3.1** reached end-of-life in December 2022 and does not receive security patches; it must be upgraded to EF Core 8.x or later. All **Microsoft.Extensions.*** packages (DI, Logging, Configuration, Caching) are at version 3.1.32 — also EOL. **System.Messaging (MSMQ)** is a Windows-only built-in .NET Framework library with no .NET Core equivalent; migration to Azure Service Bus or another cloud messaging service is required. **Microsoft.Data.SqlClient 2.1.4** is several major versions behind the current 5.x release and has known CVE vulnerabilities. **Modernizr 2.6.2** is significantly outdated (last release 2013) and may contain security issues; consider removing it.

### Notable Observations

- **MSAL installed but unused**: `Microsoft.Identity.Client 4.21.1` is declared as a dependency but the application implements no authentication or authorization — all routes are anonymous. This library should either be activated or removed.
- **Polyfill packages can be removed after .NET migration**: Packages like `System.Buffers`, `System.Memory`, `System.Threading.Tasks.Extensions`, `Microsoft.Bcl.AsyncInterfaces`, and `System.Numerics.Vectors` are backports for .NET Framework compatibility and are built-in on .NET 5+. They can be removed after migration.
- **NETStandard.Library 2.0.3 as explicit dependency**: This is a meta-package that should not typically be referenced directly in .NET Framework projects; it indicates a non-standard setup.
- **No dedicated logging library declared**: The project uses custom `LoggingService` wrapping `System.Diagnostics.Debug.WriteLine` but no structured logging library (Serilog, NLog, Microsoft.Extensions.Logging providers) is declared.

## Test Dependencies

No test project or test-scoped dependencies were detected in the solution.

Total test-scope dependencies: 0

The solution contains no test project and no test framework references (xUnit, NUnit, MSTest, etc.). Adding a test project with unit and integration tests is strongly recommended before beginning the modernization effort.
