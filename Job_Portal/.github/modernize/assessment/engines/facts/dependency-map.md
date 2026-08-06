# Dependency Map

This dependency map covers declared NuGet dependencies for Job_Portal from `packages.config` and project references in `Job_Portal.csproj` (24 external packages detected).

## Dependencies

```mermaid
flowchart LR
    App["Job_Portal"]

    subgraph Web["Web Frameworks"]
        AspNet["ASP.NET Web Forms on .NET Framework 4.7.2"]
        CodeDom["Microsoft.CodeDom.Providers.DotNetCompilerPlatform 2.0.1"]
    end

    subgraph DB["Database and ORM"]
        MySqlData["MySql.Data 9.2.0"]
        MySqlConnector["MySqlConnector 2.4.0"]
    end

    subgraph Security["Security"]
        BouncyCastle["BouncyCastle.Cryptography 2.4.0"]
    end

    subgraph Logging["Logging"]
        LogAbs["Microsoft.Extensions.Logging.Abstractions 8.0.2"]
        DiagSrc["System.Diagnostics.DiagnosticSource 8.0.1"]
    end

    subgraph Util["Utilities"]
        Json["Newtonsoft.Json 13.0.3"]
        Protobuf["Google.Protobuf 3.26.1"]
        IText["iTextSharp 5.5.13.4"]
        Lz4["K4os.Compression.LZ4 1.3.8"]
        Lz4Stream["K4os.Compression.LZ4.Streams 1.3.8"]
        XxHash["K4os.Hash.xxHash 1.0.8"]
        Zstd["ZstdSharp.Port 0.8.0"]
        AsyncIntf["Microsoft.Bcl.AsyncInterfaces 8.0.0"]
        DiAbs["Microsoft.Extensions.DependencyInjection.Abstractions 8.0.2"]
        Buffers["System.Buffers 4.5.1"]
        CfgMgr["System.Configuration.ConfigurationManager 8.0.0"]
        Pipelines["System.IO.Pipelines 5.0.2"]
        Memory["System.Memory 4.5.5"]
        Vectors["System.Numerics.Vectors 4.5.0"]
        Unsafe["System.Runtime.CompilerServices.Unsafe 6.0.0"]
        TaskExt["System.Threading.Tasks.Extensions 4.5.4"]
    end

    App -->|"web"| Web
    App -->|"persistence"| DB
    App -->|"security"| Security
    App -->|"logging"| Logging
    App -->|"utilities"| Util
```

### Dependency Summary

| Category | Count | Key Libraries | Notes |
|---|---:|---|---|
| Web Frameworks | 2 | ASP.NET Web Forms, DotNetCompilerPlatform | Legacy .NET Framework web stack |
| Database and ORM | 2 | MySql.Data, MySqlConnector | Two MySQL client libraries coexist |
| Security | 1 | BouncyCastle.Cryptography | Cryptographic primitives |
| Logging | 2 | Microsoft.Extensions.Logging.Abstractions, DiagnosticSource | Abstractions only, no dedicated sink package |
| Utilities | 17 | Newtonsoft.Json, iTextSharp, Google.Protobuf, System.* | Mixed modern package versions on net472 |

### Version & Compatibility Risks

The project targets .NET Framework 4.7.2, which constrains modernization paths and increases compatibility friction with newer 8.x dependency packages. The coexistence of both `MySql.Data` and `MySqlConnector` can introduce runtime behavior differences and duplicate transitive trees.

### Notable Observations

- `iTextSharp 5.5.x` is a legacy line and frequently flagged for modernization/licensing review.
- Multiple compression packages (`LZ4`, `xxHash`, `ZstdSharp`) are present without obvious central abstraction.
- Dependencies are managed via `packages.config` instead of SDK-style `<PackageReference>`.
- No explicit test-scoped dependencies were detected in build/package files.

## Test Dependencies

| Framework | Version | Notes |
|---|---|---|
| None detected | N/A | No test package declarations found in this project |

Total test-scope dependencies: 0
No test dependencies were detected from `packages.config` or project metadata.
