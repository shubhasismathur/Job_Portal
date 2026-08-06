# Security Assessment Report

**Generated:** 2026-08-06T11:25:02.0000000Z

## Summary

| Metric | Count |
|--------|-------|
| Total Findings | 3 |
| CVE Vulnerabilities | 0 |
| CWE Vulnerabilities | 3 |
| Total Rules Assessed | 59 |
| Rules Passed | 56 |

### By Severity

| Severity | Count |
|----------|-------|
| mandatory | 0 |
| optional | 3 |
| potential | 0 |

## CVE Findings (Dependency Vulnerabilities)

No CVE findings met the configured severity threshold (`critical`).

## CWE Findings (Code-Level Vulnerabilities)

### CWE-259: Use of Hard-coded Password
- **Category:** Credentials & Secrets
- **Severity:** optional
- **Story Points:** 5
- **Files:** Web.config:9

`Web.config` defines a database connection string inline with a password segment (`connectionString=...;user id=root;******;database=...`). This is a hard-coded password pattern in application configuration.

### CWE-798: Use of Hard-coded Credentials
- **Category:** Credentials & Secrets
- **Severity:** optional
- **Story Points:** 5
- **Files:** Web.config:9

Database credentials are embedded directly in `Web.config` connection string configuration, indicating static credential storage in source-controlled configuration.

### CWE-79: Improper Neutralization of Input During Web Page Generation
- **Category:** Injection Attacks
- **Severity:** optional
- **Story Points:** 8
- **Files:** JobSeeker/Register.aspx.cs:60, JobSeeker/Profile.aspx.cs:42

User-controlled data is written to HTML output without explicit encoding. In `JobSeeker/Register.aspx.cs` line 60, username input is embedded directly into `lblMsg.Text` with HTML markup. In `JobSeeker/Profile.aspx.cs` line 42 onward, profile fields loaded from database are directly assigned to label text controls. This creates stored/reflected XSS risk if malicious script payloads are persisted in user fields.
