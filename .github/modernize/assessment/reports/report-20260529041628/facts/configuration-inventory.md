# Configuration & Externalized Settings Inventory

Configuration is centralized primarily in `web.config`, with additional container/runtime settings in Dockerfile and environment-level host configuration. The application currently uses file-based settings for connection strings and auth gateway URLs with inline sensitive values.

## Configuration Sources

| Source | Type | Path/Location | Notes |
|---|---|---|---|
| web.config | Application/runtime config | `/web.config` | Connection strings, appSettings, authentication, authorization, machineKey |
| ZavaReportDashboard.csproj | Build config | `/ZavaReportDashboard.csproj` | Target framework and build configuration groups |
| Dockerfile | Container runtime config | `/Dockerfile` | Mono/xsp4 runtime setup, exposed port, startup command |

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies/Plugins |
|---|---|---|---|
| Debug | `Configuration=Debug` | Development build output to `bin\` | .NET Framework 4.8 build target |
| Release | `Configuration=Release` | Production build output to `bin\` | .NET Framework 4.8 build target |

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides |
|---|---|---|---|
| Default ASP.NET runtime | IIS/ASP.NET hosting load of web.config | `web.config` | Forms auth enabled, anonymous denied except Login/Logout |
| Container runtime | Docker `CMD` xsp4 execution | `Dockerfile`, `web.config` | Binds to port 8080 and serves app root |

## Properties Inventory

| Property Key | Default | Profiles | Source |
|---|---|---|---|
| connectionStrings:ZavaBankDb | `Server=sqlserver,1433;Database=ZavaBankDB;User Id=sa;...` | Default | web.config |
| appSettings:AuthGatewayLoginUrl | `http://localhost/auth/Login.aspx` | Default | web.config |
| appSettings:AuthGatewayLogoutUrl | `http://localhost/auth/Logout.aspx` | Default | web.config |
| system.web.compilation@debug | `true` | Default | web.config |
| system.web.compilation@targetFramework | `4.8` | Default | web.config |
| system.web.httpRuntime@targetFramework | `4.8` | Default | web.config |
| system.web.customErrors@mode | `Off` | Default | web.config |
| system.web.authentication/forms@timeout | `30` | Default | web.config |
| system.web.authentication/forms@name | `.ZAVAAUTH` | Default | web.config |
| system.web.authentication/forms@slidingExpiration | `true` | Default | web.config |

## Startup Parameters & Resource Requirements

| Service | JVM/Runtime Options | Memory | Instance Count |
|---|---|---|---|
| ZavaReportDashboard (xsp4 via mono) | `xsp4 --port 8080 --root /app` | Not explicitly configured | Not explicitly configured |

## Startup Dependency Chain

1. ZavaReportDashboard process starts (`xsp4`) and loads `web.config`.
2. Application requires configured SQL Server endpoint (`sqlserver:1433`) to fulfill dashboard queries.
3. Login/logout flows require reachable Auth Gateway URLs for full authentication lifecycle.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage (masked) |
|---|---|---|
| `connectionStrings:ZavaBankDb` credentials | Database credential | web.config (`[MASKED]`) |
| `system.web/machineKey` validation/decryption keys | Cryptographic keys | web.config (`[MASKED]`) |

### Secrets Provisioning Workflow

Secrets are currently file-provisioned directly in `web.config` and loaded at application startup by ASP.NET configuration APIs. No external secret manager integration or managed identity flow was detected; secret rotation and access control appear to depend on deployment-time file handling.

## Feature Flags

| Flag Name | Default | Controlled By |
|---|---|---|
| None detected | N/A | N/A |

## Framework & Runtime Versions

| Component | Version | Source |
|---|---|---|
| .NET Framework target | 4.8 | ZavaReportDashboard.csproj |
| ASP.NET Web Forms runtime | .NET Framework System.Web | csproj references |
| ADO.NET SqlClient | .NET Framework System.Data | csproj references / source usage |
| Container runtime | mono latest + xsp4 | Dockerfile |
