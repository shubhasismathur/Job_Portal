# Configuration & Externalized Settings Inventory

The project uses a small set of XML-based configuration files with a single primary runtime profile and no external configuration server. Sensitive settings are embedded in `Web.config` and consumed directly by the application at runtime.

## Configuration Sources

| Source | Type | Path/Location | Notes |
|---|---|---|---|
| Web.config | Application runtime config | `Job_Portal/Web.config` | Connection strings, appSettings, compiler and runtime binding redirects |
| Web.Debug.config | Transform config | `Job_Portal/Web.Debug.config` | Debug transform placeholder |
| Web.Release.config | Transform config | `Job_Portal/Web.Release.config` | Release transform placeholder |
| Job_Portal.csproj | Build config | `Job_Portal/Job_Portal.csproj` | Build settings, imports, references, target framework |
| packages.config | Dependency config | `Job_Portal/packages.config` | NuGet package versions |

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies/Plugins |
|---|---|---|---|
| Debug | Visual Studio/MSBuild configuration | Development build and debugging | .NET Framework 4.7.2 project settings |
| Release | Visual Studio/MSBuild configuration | Production-oriented build output | Uses Web.Release.config transform hooks |

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides |
|---|---|---|---|
| Default | IIS/ASP.NET host startup | `Web.config` | Connection string `cs`, target framework 4.7.2 |
| Debug transform | Build transform selection | `Web.Debug.config` + `Web.config` | Environment-specific transform capability |
| Release transform | Build transform selection | `Web.Release.config` + `Web.config` | Environment-specific transform capability |

## Properties Inventory

| Property Key | Default | Profiles | Source |
|---|---|---|---|
| connectionStrings:cs | `server=127.0.0.1;user id=root;******;database=OnlineJobPortal;` | Default | Web.config |
| ValidationSettings:UnobtrusiveValidationMode | `None` | Default | Web.config |
| compilation:debug | `true` | Default (often overridden by transform) | Web.config |
| compilation:targetFramework | `4.7.2` | Default | Web.config |
| httpRuntime:targetFramework | `4.7.2` | Default | Web.config |
| System.Memory bindingRedirect | `4.0.1.2` | Default | Web.config |
| System.Runtime.CompilerServices.Unsafe bindingRedirect | `6.0.0.0` | Default | Web.config |

## Startup Parameters & Resource Requirements

| Service | JVM/Runtime Options | Memory | Instance Count |
|---|---|---|---|
| Job_Portal | ASP.NET on .NET Framework under IIS (no explicit startup args in repo) | Not specified in repo | Not specified in repo |

## Startup Dependency Chain

1. IIS/ASP.NET host starts Job_Portal application domain.
2. Application loads `Web.config` and runtime binding redirects.
3. First request requiring data access opens MySQL connection using `connectionStrings:cs`.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage (masked) |
|---|---|---|
| `connectionStrings:cs` password segment | Database credential | `Web.config` inline value `[MASKED]` |

### Secrets Provisioning Workflow

No external secret manager integration was identified. Credentials appear to be stored directly in configuration and loaded at runtime by `ConfigurationManager.ConnectionStrings`. Deployment-time provisioning is therefore likely manual or environment transform based.

## Feature Flags

| Flag Name | Default | Controlled By |
|---|---|---|
| None detected | N/A | N/A |

## Framework & Runtime Versions

| Component | Version | Source |
|---|---|---|
| .NET Framework target | 4.7.2 | `Job_Portal.csproj`, `Web.config` |
| ASP.NET Web Forms project type | Legacy Web Application | `Job_Portal.csproj` ProjectTypeGuids |
| MySql.Data | 9.2.0 | `packages.config` |
| MySqlConnector | 2.4.0 | `packages.config` |
| Newtonsoft.Json | 13.0.3 | `packages.config` |
| Microsoft.CodeDom.Providers.DotNetCompilerPlatform | 2.0.1 | `packages.config` |
