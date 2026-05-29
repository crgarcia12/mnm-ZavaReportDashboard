# Architecture Diagram

This document summarizes the dashboard architecture and the main runtime relationships found in the repository. The application is a small ASP.NET Web Forms site that renders reports directly from SQL queries and relies on an external authentication gateway.

## Application Architecture

```mermaid
flowchart TD
    subgraph Client["Client Layer"]
        Browser["Web Browser"]
    end
    subgraph App["Application Layer - ASP.NET Web Forms on .NET Framework 4.8"]
        Master["Site.Master layout"]
        LoginPage["Login.aspx"]
        Dashboard["Default.aspx reporting page"]
        LogoutPage["Logout.aspx"]
    end
    subgraph Data["Data Layer"]
        ADO["ADO.NET SqlConnection and SqlCommand"]
        SQL[("SQL Server ZavaBankDB")]
    end
    subgraph External["External Services"]
        Auth["ZavaAuthGateway"]
    end

    Browser -->|"requests pages"| Master
    Master -->|"routes unauthenticated users"| LoginPage
    Browser -->|"loads dashboard"| Dashboard
    Dashboard -->|"executes report queries"| ADO
    ADO -->|"SQL queries"| SQL
    LoginPage -->|"redirects for sign-in"| Auth
    LogoutPage -->|"redirects for sign-out"| Auth
```

### Technology Stack Summary

| Layer | Technology | Version | Purpose |
| --- | --- | --- | --- |
| Presentation | ASP.NET Web Forms | .NET Framework 4.8 | Renders login, logout, and reporting pages |
| UI Composition | Site.Master, UpdatePanel, GridView, Literal | Framework-provided | Shared shell and partial page updates for reports |
| Data Access | ADO.NET via SqlConnection, SqlCommand, SqlDataAdapter | Framework-provided | Executes direct SQL queries and binds results to controls |
| Data Storage | Microsoft SQL Server | Not pinned in repo | Stores loan application, payment, and transaction reporting data |
| Hosting | Mono XSP4 container | 6.12 base image | Containerized runtime for the Web Forms site |

### Data Storage & External Services

The application connects to a single SQL Server database identified as ZavaBankDB and reads reporting data from LoanApplications, LoanPayments, and Transactions. Authentication is delegated to a separate ZavaAuthGateway application by redirecting users to configured login and logout URLs.

### Key Architectural Decisions

- Uses server-rendered Web Forms pages rather than a separate API plus SPA architecture.
- Accesses reporting data through inline SQL in page code-behind instead of a repository or ORM layer.
- Delegates authentication to an external gateway while enforcing Forms Authentication locally for page access.

## Component Relationships

```mermaid
flowchart LR
    subgraph Presentation["Presentation"]
        SiteMaster["SiteMaster"]
        Login["Login page"]
        DefaultPage["Default page"]
        Logout["Logout page"]
    end
    subgraph Business["Business Logic"]
        AuthFlow["Authentication redirect flow"]
        ReportFlow["Report orchestration"]
        Validation["Date range validation"]
    end
    subgraph DataAccess["Data Access"]
        LoanSummary["BuildLoanSummary query"]
        Delinquency["BindDelinquency query"]
        DailyVolume["BindDailyVolume query"]
    end
    subgraph Infra["Infrastructure"]
        FormsAuth["Forms Authentication"]
        AuthGateway["External auth gateway"]
        SqlDb["SQL Server database"]
    end

    SiteMaster -->|"shows auth state"| FormsAuth
    Login -->|"starts sign-in"| AuthFlow
    AuthFlow -->|"redirects"| AuthGateway
    Logout -->|"signs out and redirects"| FormsAuth
    Logout -->|"forwards sign-out"| AuthGateway
    DefaultPage -->|"validates input"| Validation
    DefaultPage -->|"runs reports"| ReportFlow
    ReportFlow -->|"summary"| LoanSummary
    ReportFlow -->|"delinquency"| Delinquency
    ReportFlow -->|"daily volume"| DailyVolume
    LoanSummary -->|"reads"| SqlDb
    Delinquency -->|"reads"| SqlDb
    DailyVolume -->|"reads"| SqlDb
    FormsAuth -.->|"guards page access"| DefaultPage
```

### Component Inventory

| Component | Layer | Type | Responsibility |
| --- | --- | --- | --- |
| SiteMaster | Presentation | Master page | Renders shared navigation and user status |
| Login page | Presentation | Web Forms page | Redirects unauthenticated users to the external auth gateway |
| Default page | Presentation | Web Forms page | Accepts date filters and renders reporting widgets |
| Logout page | Presentation | Web Forms page | Signs the user out locally and redirects to gateway logout |
| Authentication redirect flow | Business Logic | Page orchestration | Builds login and logout redirect targets |
| Report orchestration | Business Logic | Page method group | Refreshes summary, delinquency, and volume reports |
| Date range validation | Business Logic | Validation rule | Rejects invalid or reversed date ranges |
| BuildLoanSummary query | Data Access | ADO.NET query routine | Aggregates applications by status |
| BindDelinquency query | Data Access | ADO.NET query routine | Lists overdue unpaid loan payments |
| BindDailyVolume query | Data Access | ADO.NET query routine | Aggregates transaction volume by day |
| Forms Authentication | Infrastructure | Security mechanism | Tracks authenticated users and guards protected pages |
| External auth gateway | Infrastructure | External web app | Performs centralized sign-in and sign-out |
| SQL Server database | Infrastructure | Database | Stores reporting source data |
