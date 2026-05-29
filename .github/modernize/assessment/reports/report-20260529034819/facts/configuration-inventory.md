# Configuration & Externalized Settings Inventory

This repository has a small but security-sensitive configuration surface centered on `web.config`, the project file, and the Dockerfile. Most runtime behavior is controlled through inline XML settings rather than environment-specific configuration files.

## Configuration Sources

| Source | Type | Path/Location | Notes |
| --- | --- | --- | --- |
| ASP.NET application config | XML configuration | `/tmp/workspace/crgarcia12/mnm-ZavaReportDashboard/web.config` | Primary runtime source for connection strings, app settings, auth, and machine keys |
| MSBuild project file | Build configuration | `/tmp/workspace/crgarcia12/mnm-ZavaReportDashboard/ZavaReportDashboard.csproj` | Declares target framework, references, content, and build outputs |
| NuGet package manifest | Package configuration | `/tmp/workspace/crgarcia12/mnm-ZavaReportDashboard/packages.config` | Present but empty |
| Container image build | Dockerfile | `/tmp/workspace/crgarcia12/mnm-ZavaReportDashboard/Dockerfile` | Defines Mono-based runtime, compile step, and exposed port |

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies/Plugins |
| --- | --- | --- | --- |
| Debug | Default when Configuration is unset | Local or development-style build output to `bin/` | Uses .NET Framework 4.8 references declared in the project |
| Release | Manual `/p:Configuration=Release` | Production-style build output to `bin/` | Uses same framework references; no profile-specific packages detected |

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides |
| --- | --- | --- | --- |
| Default | ASP.NET runtime default | `web.config` | Connection string, auth gateway URLs, forms auth, machine key |

## Properties Inventory

| Property Key | Default | Profiles | Source |
| --- | --- | --- | --- |
| connectionStrings:ZavaBankDb | `Server=sqlserver,1433;Database=ZavaBankDB;User Id=sa;******;TrustServerCertificate=true;` | Default | `web.config` |
| appSettings:AuthGatewayLoginUrl | `http://localhost/auth/Login.aspx` | Default | `web.config` |
| appSettings:AuthGatewayLogoutUrl | `http://localhost/auth/Logout.aspx` | Default | `web.config` |
| system.web/compilation@debug | `true` | Default | `web.config` |
| system.web/compilation@targetFramework | `4.8` | Default | `web.config` |
| system.web/httpRuntime@targetFramework | `4.8` | Default | `web.config` |
| system.web/customErrors@mode | `Off` | Default | `web.config` |
| system.web/authentication@mode | `Forms` | Default | `web.config` |
| forms@loginUrl | `~/Login.aspx` | Default | `web.config` |
| forms@timeout | `30` | Default | `web.config` |
| forms@name | `.ZAVAAUTH` | Default | `web.config` |
| machineKey@validationKey | `[MASKED]` | Default | `web.config` |
| machineKey@decryptionKey | `[MASKED]` | Default | `web.config` |

## Startup Parameters & Resource Requirements

| Service | JVM/Runtime Options | Memory | Instance Count |
| --- | --- | --- | --- |
| Zava Report Dashboard | `xsp4 --port 8080 --address 0.0.0.0 --nonstop` | Not specified | 1 container process implied by Dockerfile |

## Startup Dependency Chain

1. SQL Server must be reachable at the configured host before the reporting page can execute queries successfully.
2. The external authentication gateway must be reachable before the login and logout pages can complete their redirect-based auth flow.
3. The Web Forms application starts under Mono XSP4 after source compilation during image build; no explicit readiness or health-check mechanism is configured.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage (masked) |
| --- | --- | --- |
| `connectionStrings:ZavaBankDb` | SQL Server credentialed connection string | Stored inline in `web.config` with password masked in this document |
| `machineKey:validationKey` | ASP.NET validation secret | Stored inline in `web.config` and masked in this document |
| `machineKey:decryptionKey` | ASP.NET decryption secret | Stored inline in `web.config` and masked in this document |

### Secrets Provisioning Workflow

No external secret store or provisioning workflow is configured in the repository. Sensitive values are checked into `web.config`, so deployment appears to rely on static configuration files rather than environment injection, managed identities, or vault-based secret resolution.

## Feature Flags

| Flag Name | Default | Controlled By |
| --- | --- | --- |
| None detected | N/A | No feature-flag framework or conditional config found |

## Framework & Runtime Versions

| Component | Version | Source |
| --- | --- | --- |
| .NET Framework target | 4.8 | `ZavaReportDashboard.csproj` and `web.config` |
| ASP.NET Web Forms | .NET Framework-provided | `System.Web` reference in `ZavaReportDashboard.csproj` |
| Mono base image | 6.12 | `Dockerfile` |
| XSP runtime | mono-xsp4 package | `Dockerfile` |
| MSBuild schema | ToolsVersion 4.0 project format | `ZavaReportDashboard.csproj` |
