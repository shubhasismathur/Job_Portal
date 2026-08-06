# Modernization Plan: Job Portal Cloud Modernization

**Project**: Job_Portal

---

## Technical Framework

- **Language**: C# / .NET Framework 4.7.2
- **Framework**: ASP.NET WebForms 4.7.2
- **Build Tool**: MSBuild, NuGet (packages.config)
- **Database**: MySQL (OnlineJobPortal) via raw ADO.NET (MySql.Data 9.2.0)
- **Key Dependencies**: BouncyCastle.Cryptography 2.4.0, iTextSharp 5.5.13.4, MySql.Data 9.2.0, Newtonsoft.Json 13.0.3

---

## Overview

> This migration modernizes the Job Portal ASP.NET WebForms application from .NET Framework 4.7.2 to .NET 10 (ASP.NET Core), moves its MySQL database to Azure Database for MySQL, stores files (company logos, resumes) in Azure Blob Storage, externalizes secrets to Azure Key Vault, adds console-based structured logging, and deploys the containerized application to Azure Container Apps using Managed Identity throughout. The application currently runs as a monolithic IIS-hosted WebForms app on Windows with a hardcoded localhost MySQL connection and plaintext credentials in Web.config. The new architecture will:
>
> - Eliminate the Windows/IIS dependency by migrating from ASP.NET WebForms to ASP.NET Core (.NET 10), enabling Linux container deployment
> - Replace the local MySQL instance with Azure Database for MySQL, removing hardcoded localhost and credential exposure
> - Move local file system storage (CompanyLogos, Resumes folders) to Azure Blob Storage, enabling cloud-scalable multi-instance deployment
> - Secure credentials and secrets using Azure Key Vault with Managed Identity, eliminating plaintext passwords in config files
> - Externalize non-secret application settings from Web.config to Azure App Configuration
> - Add structured console logging for cloud observability
> - Remediate known CVEs in NuGet dependencies
> - Deploy to Azure Container Apps via containerization
>
> The migration follows a phased approach: first upgrade the runtime (.NET 10), then transform services one by one, apply security remediation, and finally containerize and deploy.

---

## Migration Impact Summary

| Application  | Original Service            | New Azure Service                   | Authentication     | Comments                                              |
|--------------|-----------------------------|-------------------------------------|--------------------|-------------------------------------------------------|
| Job_Portal   | MySQL localhost              | Azure Database for MySQL            | Managed Identity   | Addresses ISSUE-010: hardcoded localhost connection   |
| Job_Portal   | Local filesystem (Resumes,  | Azure Blob Storage                  | Managed Identity   | Addresses ISSUE-005: local file system storage        |
|              | CompanyLogos)               |                                     |                    |                                                       |
| Job_Portal   | Web.config credentials      | Azure Key Vault (secrets)           | Managed Identity   | Addresses ISSUE-006: credentials in Web.config        |
| Job_Portal   | Web.config appsettings      | Azure App Configuration             | Managed Identity   | Externalize non-secret config from Web.config         |
| Job_Portal   | No logging                  | Console Logging (cloud-native)      | N/A                | Addresses ISSUE-015: no error logging framework       |
| Job_Portal   | .NET Framework 4.7.2 /      | .NET 10 / ASP.NET Core              | N/A                | Addresses ISSUE-004, ISSUE-012, ISSUE-013: WebForms   |
|              | ASP.NET WebForms / IIS      |                                     |                    | and IIS-specific platform blockers                    |
| Job_Portal   | IIS / Windows               | Azure Container Apps                | Managed Identity   | Cloud-native containerized deployment                 |

---

## Open Questions & Questionnaire

- [x] Q: Should the plan include environment/infrastructure provisioning? → A: No — no infrastructure provisioning; focus on code migration only (no infra config found in repo)
- [x] Q: Should the plan include integration testing? → A: No — skip integration testing entirely (no test infrastructure present, ISSUE-016 noted as optional)
- [x] Q: Should the plan include a security scan and CVE remediation task? → A: Yes — include security/CVE remediation (default)
- [x] Q: Which Azure deployment target should the plan use? → A: Azure Container Apps (default) — includes containerization
- [x] Q: Should the plan include containerization? → A: Skipped — deployment target (Azure Container Apps) already includes containerization
