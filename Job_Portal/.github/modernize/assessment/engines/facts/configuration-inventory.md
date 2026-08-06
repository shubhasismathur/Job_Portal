# Configuration & Externalized Settings Inventory

The Job Portal application has a minimal configuration landscape: a single `Web.config` file is the sole configuration source, with `Web.Debug.config` and `Web.Release.config` providing build-time XDT transform overlays. No external config servers, secrets managers, environment variable files, or feature flag systems are present.

## Configuration Sources

| Source | Type | Path/Location | Notes |
|---|---|---|---|
| Web.config | XML Application Config | Job_Portal/Web.config | Primary config: connection strings, AppSettings, compiler settings, assembly bindings |
| Web.Debug.config | XML XDT Transform | Job_Portal/Web.Debug.config | Applied during Debug build — currently empty (no overrides defined) |
| Web.Release.config | XML XDT Transform | Job_Portal/Web.Release.config | Applied during Release build — removes `debug="true"` from compilation element |
| packages.config | NuGet Package Manifest | Job_Portal/packages.config | Declares all NuGet package references and target frameworks |
| AssemblyInfo.cs | Assembly Metadata | Job_Portal/Properties/AssemblyInfo.cs | Assembly version, title, copyright metadata |

No environment variable files, Docker Compose environment sections, Kubernetes ConfigMaps/Secrets, Spring Cloud Config, Azure App Configuration, HashiCorp Vault, or any other external configuration source is present.

## Build Profiles

| Profile | Activation | Purpose | Key Changes |
|---|---|---|---|
| Debug | Default in Visual Studio / `msbuild /p:Configuration=Debug` | Local development and debugging | `compilation debug="true"` retained; Web.Debug.config transform applied (currently a no-op) |
| Release | `msbuild /p:Configuration=Release` | Production deployment package | `debug="true"` attribute removed from `<compilation>` by Web.Release.config XDT transform |

No additional build profiles (e.g., Staging, QA) are defined. The `Web.Release.config` transform does not override the connection string — the same hardcoded localhost connection string is used in all builds.

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides |
|---|---|---|---|
| (single, no profiles) | N/A | Web.config | No runtime profile system exists |

The application does not use ASP.NET environment profiles (`ASPNETCORE_ENVIRONMENT` is not applicable to .NET Framework). There is no `appsettings.Development.json` / `appsettings.Production.json` equivalent. All configuration is static and read from `Web.config` at startup.

## Properties Inventory

### Job_Portal (Web.config)

| Property Key | Value | Source | Notes |
|---|---|---|---|
| connectionStrings["cs"] — server | 127.0.0.1 | Web.config | MySQL host — hardcoded localhost; not environment-variable driven |
| connectionStrings["cs"] — user id | root | Web.config | Hardcoded root user — privileged database account |
| connectionStrings["cs"] — password | [MASKED] | Web.config | Plaintext credential in config file |
| connectionStrings["cs"] — database | OnlineJobPortal | Web.config | Target schema name |
| connectionStrings["cs"] — providerName | MySql.Data.MySqlClient | Web.config | ADO.NET provider factory name |
| appSettings["ValidationSettings:UnobtrusiveValidationMode"] | None | Web.config | Disables ASP.NET WebForms unobtrusive validation JS |
| system.web/compilation[@debug] | true | Web.config | Debug compilation enabled — removed by Release XDT transform |
| system.web/compilation[@targetFramework] | 4.7.2 | Web.config | .NET Framework target version |
| system.web/httpRuntime[@targetFramework] | 4.7.2 | Web.config | HTTP pipeline target version |
| runtime/assemblyBinding — System.Memory | redirect 0.0.0.0-4.0.1.2 → 4.0.1.2 | Web.config | NuGet-generated binding redirect |
| runtime/assemblyBinding — System.Runtime.CompilerServices.Unsafe | redirect 0.0.0.0-6.0.0.0 → 6.0.0.0 | Web.config | NuGet-generated binding redirect |

## Startup Parameters & Resource Requirements

| Service | Runtime Options | Memory | CPU | Instance Count |
|---|---|---|---|---|
| Job_Portal (IIS Worker Process) | Standard ASP.NET application pool defaults | IIS default (varies by server) | IIS default | 1 (no load balancer or multi-instance config) |
| MySQL | MySQL server defaults | Not configured in application | Not configured | 1 (localhost only) |

No JVM/CLR heap tuning, Docker container resource limits, Kubernetes resource requests/limits, or autoscaling configuration is present. The application assumes a single-instance IIS deployment.

## Startup Dependency Chain

```
IIS Application Pool starts
  └─► ASP.NET loads Web.config
        └─► Connection string read into ConfigurationManager.ConnectionStrings["cs"]
              └─► MySQL (127.0.0.1:3306) must be reachable at runtime
                    (no readiness probe, no retry logic — first DB query fails with unhandled exception if MySQL is down)
```

- **No startup wait mechanism**: There is no `dockerize`, health-check retry, or connection test on startup. Database connection failures surface as unhandled exceptions on the first page load that touches the database.
- **No health check endpoint**: The application exposes no `/health` or `/readyz` probe path.
- **IIS application pool recycling**: Handled by IIS with default recycle settings (no custom configuration found).

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage Method | Masked Value |
|---|---|---|---|
| connectionStrings["cs"].password | MySQL root password | Plaintext in Web.config | [MASKED] |
| connectionStrings["cs"].user id | MySQL username | Plaintext in Web.config | root |

### Secrets Provisioning Workflow

**Current state — no secrets management workflow exists**:

1. The MySQL password is stored in plaintext directly in `Web.config` (source-controlled).
2. There is no environment variable injection, no secrets manager, and no DPAPI encryption of config sections.
3. The `Web.Release.config` XDT transform does not inject or override the connection string — the same developer/test credentials are used in all environments.
4. No CI/CD pipeline secrets injection (GitHub Actions secrets, Azure Key Vault references, AWS Secrets Manager) is configured.

**Recommended remediation**:
- Encrypt the `<connectionStrings>` section using ASP.NET Protected Configuration (`aspnet_regiis -pe`) as a minimum step.
- For cloud migration, replace with environment variable injection or a secrets manager integration (Azure Key Vault, AWS Secrets Manager).

## Feature Flags

No feature flag framework is present. There are no `@ConditionalOnProperty` equivalents, no LaunchDarkly/Unleash integration, no `.NET FeatureManagement` usage, and no custom toggle configuration.

| Flag Name | Default | Controlled By |
|---|---|---|
| (none detected) | — | — |

The only conditional behavior is the `debug` compilation attribute toggled by the Release XDT transform, which is a build-time switch, not a runtime feature flag.

## Framework & Runtime Versions

| Component | Version | Source |
|---|---|---|
| Target Framework | .NET Framework 4.7.2 | Web.config `targetFramework`, Job_Portal.csproj |
| Web Framework | ASP.NET WebForms (System.Web) | In-box with .NET Framework 4.7.2 |
| C# Language | default (Roslyn via CodeDom) | compilerOptions `/langversion:default` in Web.config |
| Roslyn Compiler Provider | Microsoft.CodeDom.Providers.DotNetCompilerPlatform 2.0.1 | packages.config |
| MySQL Driver (primary) | MySql.Data 9.2.0 | packages.config |
| MySQL Driver (unused) | MySqlConnector 2.4.0 | packages.config |
| JSON Serialization | Newtonsoft.Json 13.0.3 | packages.config |
| PDF Generation | iTextSharp 5.5.13.4 | packages.config |
| Cryptography | BouncyCastle.Cryptography 2.4.0 | packages.config (transitive) |
| Build Tool | MSBuild (Visual Studio / msbuild CLI) | Job_Portal.csproj (ToolsVersion="15.0") |
| Assembly GUID | 3fb45854-15a3-47cf-93ec-e4d051abf3c6 | AssemblyInfo.cs |
| Assembly Copyright | Copyright © 2025 | AssemblyInfo.cs |
