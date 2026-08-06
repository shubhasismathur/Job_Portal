# Dependency Map

The Job Portal project is a .NET Framework 4.7.2 ASP.NET WebForms application with 22 declared NuGet dependencies listed in `packages.config`. There are no test-scoped dependencies detected.

## Dependencies

```mermaid
flowchart LR
    App["Job_Portal\n.NET Framework 4.7.2"]

    subgraph Web["Web Frameworks"]
        Roslyn["CodeDom.Providers.DotNetCompilerPlatform v2.0.1"]
    end

    subgraph DB["Database / ORM"]
        MySqlData["MySql.Data v9.2.0"]
        MySqlConn["MySqlConnector v2.4.0"]
    end

    subgraph PDF["Document Generation"]
        iText["iTextSharp v5.5.13.4"]
    end

    subgraph Serial["Serialization"]
        Newtonsoft["Newtonsoft.Json v13.0.3"]
        Protobuf["Google.Protobuf v3.26.1"]
    end

    subgraph Crypto["Cryptography"]
        Bouncy["BouncyCastle.Cryptography v2.4.0"]
    end

    subgraph Util["Utilities and Runtime"]
        BclAsync["Microsoft.Bcl.AsyncInterfaces v8.0.0"]
        DIAbstracts["Microsoft.Extensions.DependencyInjection.Abstractions v8.0.2"]
        LogAbstracts["Microsoft.Extensions.Logging.Abstractions v8.0.2"]
        ConfigMgr["System.Configuration.ConfigurationManager v8.0.0"]
        DiagSource["System.Diagnostics.DiagnosticSource v8.0.1"]
        Buffers["System.Buffers v4.5.1"]
        Memory["System.Memory v4.5.5"]
        Numerics["System.Numerics.Vectors v4.5.0"]
        Unsafe["System.Runtime.CompilerServices.Unsafe v6.0.0"]
        TasksExt["System.Threading.Tasks.Extensions v4.5.4"]
        Pipelines["System.IO.Pipelines v5.0.2"]
        LZ4["K4os.Compression.LZ4 v1.3.8"]
        LZ4S["K4os.Compression.LZ4.Streams v1.3.8"]
        xxHash["K4os.Hash.xxHash v1.0.8"]
        Zstd["ZstdSharp.Port v0.8.0"]
    end

    App -->|"web compiler"| Web
    App -->|"persistence"| DB
    App -->|"PDF generation"| PDF
    App -->|"serialization"| Serial
    App -->|"cryptography"| Crypto
    App -->|"utilities"| Util
    MySqlConn -.->|"uses"| LZ4
    MySqlConn -.->|"uses"| LZ4S
    MySqlConn -.->|"uses"| xxHash
    MySqlConn -.->|"uses"| Zstd
    MySqlConn -.->|"uses"| Pipelines
    MySqlConn -.->|"uses"| Memory
    MySqlConn -.->|"uses"| Buffers
```

### Dependency Summary

| Category | Count | Key Libraries | Notes |
|---|---|---|---|
| Web Frameworks | 1 | Microsoft.CodeDom.Providers.DotNetCompilerPlatform 2.0.1 | Roslyn compiler for WebForms; ASP.NET WebForms itself is in-box |
| Database / ORM | 2 | MySql.Data 9.2.0, MySqlConnector 2.4.0 | Two competing MySQL drivers — both present, only MySql.Data appears used in code |
| Document Generation | 1 | iTextSharp 5.5.13.4 | AGPL-licensed legacy PDF library; no longer actively maintained |
| Serialization | 2 | Newtonsoft.Json 13.0.3, Google.Protobuf 3.26.1 | JSON used for dashboard charts; Protobuf pulled in as transitive dep of MySqlConnector |
| Cryptography | 1 | BouncyCastle.Cryptography 2.4.0 | Pulled in as transitive dependency of MySqlConnector |
| Utilities / Runtime | 15 | System.Memory, System.Buffers, K4os.Compression.LZ4, ZstdSharp.Port, etc. | Mostly transitive dependencies of MySqlConnector's async/compression pipeline |

### Version & Compatibility Risks

**Microsoft.CodeDom.Providers.DotNetCompilerPlatform 2.0.1** is an old release (2018) used only by the .NET Framework WebForms compilation pipeline; it is irrelevant in any modern ASP.NET Core migration. **iTextSharp 5.5.13.4** is the final community release of the AGPL v5 branch — it is no longer actively maintained by iText Group and carries a viral AGPL license. The successor iText 7 requires a commercial license for non-AGPL use. Both **MySql.Data 9.2.0** and **MySqlConnector 2.4.0** are present but only `MySql.Data` is referenced in code; the `MySqlConnector` package and its entire dependency tree (LZ4, xxHash, ZstdSharp, Pipelines, BouncyCastle, Protobuf) appear to be unused bloat. The `System.Configuration.ConfigurationManager 8.0.0` package targets .NET Standard/Core back-compat shim and is unnecessary on .NET Framework 4.7.2 where this API is already built in.

### Notable Observations

- **Duplicate MySQL drivers**: Both `MySql.Data` (Oracle's official connector) and `MySqlConnector` (open-source async connector) are declared. Only `MySql.Data` is used in application code. `MySqlConnector` and its ~13 transitive dependencies should be removed to reduce the dependency surface.
- **Bloated transitive dependency tree from MySqlConnector**: The compression (LZ4, ZstdSharp), hashing (xxHash), cryptography (BouncyCastle), and networking (System.IO.Pipelines) packages are all pulled in transitively by the unused `MySqlConnector`. Removing it would eliminate approximately 15 of the 22 declared packages.
- **AGPL license risk (iTextSharp)**: iTextSharp 5.x is AGPL-3.0 licensed. Any distribution of the application must comply with AGPL terms (source disclosure) unless a commercial iText license is obtained. This requires legal review before any commercial deployment.
- **No ORM layer**: The project uses raw ADO.NET SQL strings throughout. There is no Entity Framework or Dapper usage. This increases maintenance burden and makes migration to EF Core or another ORM a prerequisite for modernization.

## Test Dependencies

No test-scoped dependencies were detected. There are no test projects, unit test files, or test frameworks (xUnit, NUnit, MSTest) present in the repository.

Total test-scope dependencies: **0**

No automated test infrastructure exists. This is a significant risk for any modernization effort — refactoring code without a test harness makes regressions very difficult to detect. Adding unit and integration tests should be a first step before attempting any migration.
