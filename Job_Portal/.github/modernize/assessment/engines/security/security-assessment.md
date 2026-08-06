# Security Assessment Report

**Generated:** 2026-08-06T12:05:19.0000000Z

## Summary

| Metric | Count |
|--------|-------|
| Total Findings | 7 |
| CVE Vulnerabilities | 0 |
| CWE Vulnerabilities | 7 |
| Total Rules Assessed | 59 |
| Rules Passed | 52 |

### By Severity

| Severity | Count |
|----------|-------|
| mandatory | 0 |
| optional | 4 |
| potential | 3 |

## CVE Findings (Dependency Vulnerabilities)

No CVE vulnerabilities found at or above the `high` severity threshold for the declared package versions.

## CWE Findings (Code-Level Vulnerabilities)

### CWE-772: Missing Release of Resource after Effective Lifetime
- **Category:** Code Quality
- **Severity:** potential
- **Story Points:** 3
- **Files:** Job_Portal/JobSeeker/Login.aspx.cs, Job_Portal/JobSeeker/ResumeBuild.aspx.cs

MySqlDataReader objects are obtained via ExecuteReader() but are not wrapped in 'using' statements or explicitly closed in all code paths. In Login.aspx.cs line 55, the reader is not in a using block — if an exception occurs between Open() and the end of the method, the reader is not disposed. In ResumeBuild.aspx.cs line 49, the reader obtained from ExecuteReader() is not in a using block inside LoadUserDetails().

### CWE-1057: Data Access Operations Outside of Expected Data Manager Component
- **Category:** Code Quality
- **Severity:** potential
- **Story Points:** 5
- **Files:** Job_Portal/JobSeeker/Login.aspx.cs, Job_Portal/JobSeeker/Register.aspx.cs, Job_Portal/JobSeeker/Job_Details.aspx.cs, Job_Portal/JobProvider/NewJob.aspx.cs, Job_Portal/JobProvider/Dashboard.aspx.cs

All database operations are performed directly inside ASP.NET WebForms code-behind files (presentation layer) rather than in a dedicated data manager or repository component. Every .aspx.cs file instantiates MySqlConnection and MySqlCommand inline, bypassing any centralized data management layer. This architectural pattern violates separation of concerns and makes secure, consistent data access enforcement impossible.

### CWE-259: Use of Hard-coded Password
- **Category:** Credentials & Secrets
- **Severity:** optional
- **Story Points:** 5
- **Files:** Job_Portal/Web.config

Web.config line 9 contains a hard-coded MySQL password in the connection string: connectionString="server=127.0.0.1;user id=root;******;database=OnlineJobPortal;". The password is embedded in a source-controlled configuration file and is the same across all environments (no environment-specific override in Web.Release.config).

### CWE-732: Incorrect Permission Assignment for Critical Resource
- **Category:** Credentials & Secrets
- **Severity:** optional
- **Story Points:** 5
- **Files:** Job_Portal/Web.config

The MySQL connection string uses 'user id=root' — the MySQL root account with unrestricted server-level permissions. This violates the principle of least privilege: the application has full DDL and administrative rights on the database server, not just the DML permissions needed for application operation. If the application is compromised, an attacker would have full control over the entire MySQL server.

### CWE-778: Insufficient Logging
- **Category:** Credentials & Secrets
- **Severity:** potential
- **Story Points:** 3
- **Files:** Job_Portal/JobSeeker/Login.aspx.cs, Job_Portal/JobSeeker/Register.aspx.cs, Job_Portal/JobProvider/NewJob.aspx.cs

No logging framework (EventLog, NLog, Serilog, ILogger, or ASP.NET Trace) is used anywhere in the application. Security-critical events — failed login attempts (Login.aspx.cs catch block), registration failures, file upload errors, database errors — are only displayed to the end-user via lblMsg.Text and are never logged to any persistent log store. There is no audit trail for authentication events, application errors, or data modification operations.

### CWE-798: Use of Hard-coded Credentials
- **Category:** Credentials & Secrets
- **Severity:** optional
- **Story Points:** 5
- **Files:** Job_Portal/Web.config

Web.config contains hard-coded database credentials (username 'root' and password) in the <connectionStrings> section. This file is source-controlled, meaning credentials are exposed to anyone with repository access. The Web.Release.config transform does not override the connection string, so production deployments use the same developer credentials.

### CWE-79: Improper Neutralization of Input During Web Page Generation ('Cross-site Scripting')
- **Category:** Injection Attacks
- **Severity:** optional
- **Story Points:** 8
- **Files:** Job_Portal/JobSeeker/Register.aspx.cs, Job_Portal/JobSeeker/Job_Details.aspx.cs, Job_Portal/JobProvider/NewJob.aspx.cs

Multiple reflected and stored XSS vulnerabilities exist. (1) Register.aspx.cs line 60: user-supplied username is embedded unencoded into an HTML string assigned to lblMsg.Text — `$"<b>{txtUserName.Text.Trim()}</b> username already exists."`. ASP.NET Label.Text renders raw HTML, allowing script injection. (2) Job_Details.aspx.cs lines 55, 73, 83: database-stored job fields (Description, Qualification, Experience) are assigned directly to Label and Literal controls without HTML encoding — a provider who stored malicious script in a job posting would trigger XSS for all visitors viewing that job. (3) NewJob.aspx.cs line 78: exception message appended to an HTML div string in lblMsg.Text.
