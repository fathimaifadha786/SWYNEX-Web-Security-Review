# SWYNEX Task 3 – Web Security Review

## Final Security Assessment Report

**Project:** Web Security Review  
**Application:** OWASP Juice Shop  
**Testing Environment:** Localhost – Docker Desktop  
**Testing Date:** November 2026  
**Testing Scope:** OWASP Top 10:2025 (A01–A10)  
**Testing Type:** Authorized Local Security Assessment

---

# 1. Introduction

For this task, I performed a web security review using the OWASP Juice Shop application in a local testing environment.

The main purpose was to learn about common web security issues, test the application, and understand how these issues can affect web applications.

I used Burp Suite Community Edition, Docker Desktop, browser developer tools, and PowerShell during this task.

All testing was performed only on my local training application.

# 2. Tools Used

- **OWASP Juice Shop:** Used as the vulnerable training application.
- **Docker Desktop:** Used to run the local application.
- **Burp Suite Community Edition:** Used to inspect and test HTTP requests and responses.
- **Browser Developer Tools:** Used to inspect frontend behaviour.
- **PowerShell:** Used to inspect files and package information.

# 3. OWASP Top 10 Security Review

## A01 – Broken Access Control

**Tests performed:**
- IDOR / Basket Access
- Admin Page Access
- API User Data Exposure
- SSRF

### Findings

**1. Unauthorized Basket Access (IDOR)**

- I changed the basket ID in a request.
- The application returned another user's basket information.
- The server responded with HTTP 200 OK.

**Severity:** Medium  
**Status:** Confirmed

**2. Unauthorized User Data Exposure**

- The `/api/Users` endpoint returned information about multiple users.
- The response included email addresses, roles, and other account-related information.

**Severity:** Medium  
**Status:** Confirmed

**3. Admin Page Access**

- I tried to access the administration page using a normal user account.
- The application returned HTTP 403 Forbidden.

**Status:** Not Vulnerable in Tested Scenario

**4. SSRF**

- I tested a localhost image URL.
- I did not find evidence of server-side request forgery.

**Status:** Not Confirmed

## A02 – Security Misconfiguration

**Tests performed:**
- HTTP Security Headers
- CORS Configuration
- Directory Listing
- Backup File Exposure
- Sensitive File Exposure

### Findings

**1. Directory Listing Enabled**

- The `/ftp/` directory showed a list of files and folders.
- These files were visible without authentication.

**Severity:** Medium  
**Status:** Confirmed

**2. Confidential Document Exposure**

- The `acquisitions.md` file was publicly accessible.
- It contained confidential acquisition-related information.

**Severity:** Medium  
**Status:** Confirmed

**3. Sensitive KeePass File Exposure**

- The `incident-support.kdbx` file could be downloaded.
- I did not open or analyse the file.

**Severity:** High (Preliminary)  
**Status:** Public Download Confirmed

### Other Observations

- Content-Security-Policy header was missing.
- Referrer-Policy header was missing.
- Modern Permissions-Policy header was missing.
- Wildcard CORS configuration was observed.
- Credentialed cross-origin access was not confirmed.
- Tested backup files returned HTTP 403 Forbidden.

These observations need further review. They do not all represent confirmed vulnerabilities.

## A03 – Software Supply Chain Failures

**Tests performed:**
- Package Manifest Exposure
- Dependency Lockfile Exposure
- Known Vulnerable Dependencies
- Outdated Dependencies

### Findings

- The tested `/package.json` path did not expose the package manifest.
- The tested `/package-lock.json` path did not expose the lockfile.
- Access to the backup lockfile was blocked.
- Frontend dependency searches were limited.
- Vulnerable and outdated dependency versions were not fully checked.

**Result:** No confirmed A03 vulnerability.

**Limitation:** A complete dependency vulnerability audit was not performed.

## A04 – Cryptographic Failures

**Tests performed:**
- Unencrypted Data Transmission
- Sensitive Data Exposure
- Password Storage Security
- Weak Cryptographic Algorithms
- Cryptographic Key Exposure

### Findings

**1. Unencrypted HTTP**

- The local application used HTTP.
- I could see a login request in Burp Suite over the local HTTP connection.

**Status:** Observed in Local Lab  
**Severity:** Informational / Configuration Observation

This does not prove that credentials are exposed over an external network.

**2. Sensitive Data Exposure**

- The tested `/rest/user/whoami` endpoint returned basic user information.
- No password or secret key was found in that response.

**Status:** No Issue Identified in Tested Endpoint

**3. Password Storage Security**

- I could not inspect the database storage.

**Status:** Not Assessed

**4. Weak Cryptographic Algorithms**

- Frontend searches did not confirm the use of weak cryptographic algorithms.

**Status:** Not Confirmed

**5. Cryptographic Key Exposure**

- No hardcoded key was found in the frontend files I searched.

**Status:** Not Confirmed

## A05 – Injection

**Tests performed:**
- SQL Injection
- NoSQL Injection
- Cross-Site Scripting (XSS)
- Command Injection
- Input Validation

### Findings

**1. SQL Injection – Product Search**

- I tested a modified search input.
- The application returned HTTP 500 Internal Server Error.
- The response showed SQL query details and a SQLite syntax error.

**Severity:** High (Preliminary)  
**Status:** SQL Injection Indicator Confirmed

**2. SQL Injection – Login**

- A single quote in the login email caused a SQL error.
- A modified login input returned HTTP 200 OK.
- An Admin email was visible in the response.
- A complete authenticated session was not verified.

**Severity:** High (Potential)  
**Status:** Strong SQL Injection Indicator; Authentication Bypass Not Fully Confirmed

**3. NoSQL Injection**

- I found potentially risky dynamic query construction in the reviewed source files.
- Tested inputs were blocked or sanitized.
- Successful query manipulation was not demonstrated.

**Status:** Not Confirmed

**4. Reflected XSS**

- Tested search inputs were not reflected as executable HTML.

**Status:** Not Found in Tested Endpoint

**5. Stored XSS**

- User-controlled HTML appeared in the review page's DOM.
- JavaScript execution was not confirmed.

**Status:** Potential Stored XSS – Execution Not Confirmed

**6. Command Injection**

- No direct command execution evidence was found in the reviewed files.
- Runtime testing was not completed.

**Status:** Not Assessed

**7. Input Validation**

- Empty review input was prevented.
- Full input validation was not tested.

**Status:** Partially Assessed

## A06 – Insecure Design

**Tests performed:**
- Business Logic Flaws
- Workflow Bypass
- Rate Limiting
- Quantity and Price Manipulation
- Abuse of Application Features

### Findings

- Normal basket operations worked as expected.
- Duplicate basket item requests returned database constraint errors.
- Checkout required an address and delivery selection.
- I stopped testing before payment to avoid any real payment activity.
- Five manual product searches did not trigger a visible rate-limit response.
- The tested basket creation request did not contain a client-supplied price field.
- Invalid coupon codes were rejected.
- Coupon length validation worked.

**Result:** No confirmed A06 vulnerability.

**Limitation:** Payment workflow and server-side pricing controls were not fully tested.

## A07 – Authentication Failures

**Tests performed:**
- Weak Password Policy
- Brute-Force Protection
- Account Lockout
- Authentication Bypass
- Session Management
- Password Reset Security

### Findings

**1. Weak Password Policy**

- The application accepted a five-character password.
- No additional password complexity requirement was observed.

**Severity:** Low–Medium  
**Status:** Confirmed

**2. Brute-Force Protection**

- I tried six manual failed login attempts.
- The same invalid credentials message appeared each time.
- No visible CAPTCHA or account lockout was observed.
- Backend rate limiting was not verified.

**Status:** Not Confirmed

**3. Account Lockout**

- No account lockout was observed during the limited test.

**Status:** Not Confirmed

**4. Authentication Bypass**

- An unauthenticated request to `/rest/user/whoami` returned an empty user object.
- No unauthorized account access was demonstrated.

**Status:** Not Confirmed

**5. Session Management**

- A previously issued JWT was still accepted after logout.
- The old session returned the user's profile information.

**Severity:** Medium  
**Status:** Confirmed in Tested Scenario

**6. Password Reset Security**

- An incorrect security answer was rejected.
- The correct answer allowed the password reset.

**Status:** No Issue Confirmed in Tested Scenario

## A08 – Software or Data Integrity Failures

**Tests performed:**
- Software Integrity
- Insecure Deserialization
- Data Integrity
- File Upload Integrity
- Update and Dependency Integrity

### Findings

**1. Software Integrity**

- External JavaScript files were included without SRI integrity attributes.

**Status:** Security Hardening Observation

**2. Insecure Deserialization**

- I could not identify a suitable endpoint for this test.

**Status:** Not Assessed

**3. Data Integrity**

- I could not identify a suitable product data modification endpoint in the observed requests.

**Status:** Not Assessed

**4. File Upload Integrity**

- The tested feedback form did not have a file upload field.
- No suitable upload feature was identified.

**Status:** Not Assessed

**5. Update and Dependency Integrity**

- I inspected the local package manifest and lockfile.
- The lockfile contained 1,350 integrity entries.
- The actual software update verification process was not tested.

**Status:** Not Confirmed

## A09 – Security Logging & Alerting Failures

**Tests performed:**
- Failed Login Logging
- Successful Login Logging
- Unauthorized Access Logging
- Error Logging
- Security Alerting

### Findings

- Failed login attempts returned HTTP 401 Unauthorized.
- Successful login was observed in the browser.
- The available Docker Logs did not show clear login-related security events.
- Historical logs for the earlier unauthorized access test were unavailable.
- Database-related error information appeared in Docker Logs.
- No security alert or notification was observed in the available logs.

**Result:** Logging and alerting weaknesses could not be confirmed.

**Status:** Limited Assessment – Further Verification Required

## A10 – Mishandling of Exceptional Conditions

**Tests performed:**
- Error Handling
- Invalid Input Handling
- Missing Input Handling
- Boundary Value Handling
- Database Error Handling
- Service Failure Handling

### Findings

**1. Error Handling**

- SQL error responses exposed query details during A05 testing.

**Status:** Technical Error Disclosure Observed

**2. Invalid Input Handling**

- Invalid product search input returned a normal no-results message.
- The application did not visibly crash.

**Status:** No Issue Observed

**3. Missing Input Handling**

- The Submit button remained disabled when the review field was empty.

**Status:** No Issue Observed

**4. Boundary Value Handling**

- A 100-character search input returned a normal no-results message.

**Status:** No Issue Observed in Tested Scenario

**5. Database Error Handling**

- Duplicate basket item requests returned HTTP 500.
- A Sequelize database constraint error was exposed.

**Status:** Error Handling Weakness Observed

**6. Service Failure Handling**

- Docker Logs showed warnings because an optional AI service was unavailable.
- The main application continued running.

**Status:** No Issue Observed in Tested Scenario

# 4. Key Security Findings

The following findings need attention:

| No. | Finding | Severity | OWASP Category |
|---|---|---|---|
| 1 | SQL Injection Indicators | High (Preliminary) | A05 |
| 2 | Sensitive KeePass File Exposure | High (Preliminary) | A02 |
| 3 | Unauthorized User Data Exposure | Medium | A01 |
| 4 | IDOR – Unauthorized Basket Access | Medium | A01 |
| 5 | Session Not Invalidated After Logout | Medium | A07 |
| 6 | Confidential Document Exposure | Medium | A02 |
| 7 | Directory Listing Enabled | Medium | A02 |
| 8 | Weak Password Policy | Low–Medium | A07 |
| 9 | Technical Error Information Disclosure | Observation | A05 / A10 |

These severity ratings are preliminary and may need further review.

# 5. Recommendations

Based on the testing results, I recommend the following improvements:

1. Use parameterized SQL queries to prevent SQL injection.
2. Check user permissions on the server before allowing access to user-specific data.
3. Restrict access to sensitive files and disable public directory listing.
4. Remove confidential documents and sensitive database files from public folders.
5. Invalidate sessions and revoke tokens after logout.
6. Require stronger passwords.
7. Add login rate limiting and account protection.
8. Avoid showing SQL queries, database errors, and stack traces to users.
9. Add security logging and alerting for failed logins and suspicious activity.
10. Use HTTPS in deployed environments.
11. Add suitable security headers, including Content-Security-Policy.
12. Regularly check and update software dependencies.
13. Validate user input on the server and handle errors safely.

# 6. Testing Limitations

This assessment was performed using a local OWASP Juice Shop application.

The following limitations applied:

- Not every endpoint or feature was tested.
- Some database and dependency checks could not be completed.
- Some tests were limited to browser and Burp Suite observations.
- Historical logs were unavailable for some earlier tests.
- Some potential vulnerabilities could not be confirmed.
- Payment operations were intentionally not completed.
- No external production systems were tested.
- Severity ratings are preliminary.

# 7. Conclusion

I completed a web security review of the OWASP Juice Shop application using the OWASP Top 10 categories.

During this task, I identified several security weaknesses, including unauthorized basket access, user data exposure, directory listing, SQL injection indicators, weak password policy, and session management issues.

I also learned how to inspect HTTP requests and responses, use Burp Suite, check application logs, and document security findings.

Some tests did not identify vulnerabilities, while others need further testing.

This task helped me improve my practical knowledge of web application security and understand the importance of secure coding, proper access control, authentication, and error handling.

**End of Report**
