# Assessment Overview

This document provides navigation links to all supplementary analysis documents generated during the assessment of the **Job_Portal** application — a .NET Framework 4.7.2 ASP.NET WebForms online recruitment platform.

## Supplementary Documents

| Document | Description |
|---|---|
| [Architecture Diagram](./architecture-diagram.md) | Two-layer architecture visualization: application-level flowchart (technology stack, data flow, external integrations) and component-level relationships (page interactions, data access patterns). |
| [Dependency Map](./dependency-map.md) | Visual map of all 22 declared NuGet dependencies grouped by functional category (database drivers, PDF generation, serialization, cryptography, utilities), with compatibility risk analysis and notable observations. |
| [API & Service Communication Contracts](./api-service-contracts.md) | Catalog of all 24 server-rendered page endpoints (HTTP GET/POST), communication patterns, security posture analysis, and a sequence diagram of the core request flows. |
| [Data Architecture & Persistence Layer](./data-architecture.md) | Database configuration, inferred entity model (User, Jobs, AppliedCandidates, Contact, Country), ER diagram, inline data access patterns, and PII/sensitive data classification. |
| [Configuration & Externalized Settings Inventory](./configuration-inventory.md) | Full inventory of Web.config settings, build profiles (Debug/Release), secrets handling (plaintext credentials), startup dependency chain, and framework version catalog. |
| [Core Business Workflows](./business-workflows.md) | End-to-end documentation of 7 primary business workflows (registration, login, job browsing, job application, resume building, job posting, dashboard), business rules, validation constraints, and authorization logic. |

## Key Findings Summary

| Area | Finding |
|---|---|
| Platform | ASP.NET WebForms on .NET Framework 4.7.2 — not portable to .NET Core/Linux without a full rewrite |
| Security | Plaintext password storage; credentials in Web.config; no HTTPS enforcement; session-based auth with no secure flags |
| Architecture | Monolith with no service layer, no DI, no tests; all business logic in page code-behind files |
| Data | Raw ADO.NET with no ORM; single MySQL instance hardcoded to localhost |
| Dependencies | Unused MySqlConnector package adding ~13 transitive deps; iTextSharp carries AGPL license risk |
| Observability | No health checks, no logging framework, no metrics, no tracing |

For full issue details and remediation recommendations, see the main [report.json](../report.json).
