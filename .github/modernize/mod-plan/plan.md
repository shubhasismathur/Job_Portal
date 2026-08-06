# Modernization Plan: Job_Portal Azure Cloud Modernization

**Project**: Job_Portal

---

## Technical Framework

- **Language**: .NET Framework 4.7.2, C#
- **Framework**: ASP.NET WebForms 4.7.2
- **Build Tool**: MSBuild, NuGet
- **Database**: MySQL (OnlineJobPortal, accessed via raw ADO.NET / MySql.Data 9.2.0)
- **Key Dependencies**: iTextSharp 5.5.13.4 (PDF), MySql.Data 9.2.0, MySqlConnector 2.4.0, Newtonsoft.Json 13.0.3, BouncyCastle.Cryptography 2.4.0

---

## Overview

This migration modernizes the Job_Portal ASP.NET WebForms application (.NET Framework 4.7.2) to a cloud-native ASP.NET Core application running on Azure. The application currently runs exclusively on Windows/IIS, stores files on the local file system, holds database credentials in Web.config, and connects to a MySQL database hardcoded to localhost.

The new architecture will:

- Run cross-platform in Linux containers on Azure Container Apps, addressing the WebForms/IIS-only platform blocker
- Store company logos and candidate resumes in Azure Blob Storage instead of the local file system, enabling scalability across multiple instances
- Use Azure SQL Database with Managed Identity authentication, replacing the MySQL localhost dependency and eliminating hard-coded credentials
- Externalize secrets (database credentials, connection strings) to Azure Key Vault via Managed Identity
- Emit structured console logs compatible with cloud log aggregation pipelines
- Scan and remediate known CVEs in all NuGet package dependencies before deployment

The migration follows a sequential upgrade-first approach: the .NET runtime and project format are upgraded to .NET 10 and ASP.NET Core first, followed by Azure service integrations, then security hardening, and finally deployment to Azure Container Apps.

---

## Migration Impact Summary

| Application  | Original Service            | New Azure Service             | Authentication     | Comments                                        |
|--------------|-----------------------------|-------------------------------|--------------------|-------------------------------------------------|
| Job_Portal   | ASP.NET WebForms (.NET 4.7.2)| ASP.NET Core (.NET 10)        | N/A                | Addresses ISSUE-004, ISSUE-012, ISSUE-013       |
| Job_Portal   | Local file system (Resumes, CompanyLogos) | Azure Blob Storage | Managed Identity   | Addresses ISSUE-005                             |
| Job_Portal   | MySQL (localhost, raw ADO.NET) | Azure SQL Database           | Managed Identity   | Addresses ISSUE-006, ISSUE-010                  |
| Job_Portal   | Credentials in Web.config   | Azure Key Vault Secrets       | Managed Identity   | Addresses ISSUE-006                             |
| Job_Portal   | appsettings / Web.config    | Azure App Configuration       | Managed Identity   | Addresses ISSUE-013                             |
| Job_Portal   | No structured logging       | Console logging (cloud-native)| N/A                | Addresses ISSUE-015                             |
| Job_Portal   | No deployment pipeline      | Azure Container Apps          | Managed Identity   | Default deployment target                       |

---

## Open Questions & Questionnaire

- [x] Q: What is the target .NET version? → A: .NET 10 (latest LTS), required due to mandatory blocker ISSUE-004 (ASP.NET WebForms not portable to .NET Core/Linux without a full rewrite)
- [x] Q: What is the target Azure deployment service? → A: Azure Container Apps (default)
- [x] Q: Should integration tests be included? → A: No integration testing explicitly requested; skipping integration test task
- [x] Q: What authentication method should be used for Azure services? → A: Managed Identity (default)
- [x] Q: Should infrastructure (IaC) be provisioned? → A: No explicit infrastructure request; skipping infrastructure task
