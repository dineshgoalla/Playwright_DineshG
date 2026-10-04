# Test Plan: CRM Portal Login & Session Management

---

## 1. Test Plan ID and Title

* **Document ID:** `TP-CRM-AUTH-001`
* **Title:** Test Plan for CRM Portal Login, Authentication, and Session Termination
* **Version:** 1.0
* **Author:** Senior QA Engineer (Test Automation Specialist)
* **Target Audience:** QA Engineers, Software Developers, Product Owner

---

## 2. Objective and References

### 2.1 Objective
The primary objective of this test plan is to define the testing strategy, test coverage, execution matrix, measurable entry/exit criteria, and automated regression approach for the authentication and session management workflows of the CRM Portal (`CRM.com`), specifically for `USER1` holding the `Administrator` role.

### 2.2 References
* **Requirements Specification:**
  * `REQ-01`: An active account with valid credentials can sign in.
  * `REQ-02`: An incorrect password does not authenticate the user.
  * `REQ-03`: Empty required fields prevent authentication.
  * `REQ-04`: Logout ends the authenticated session and prevents access to protected content.
* **Architecture / Design Documents:** Not provided (Proposed: standard OAuth/cookie-based session state).
* **Tooling Standard:** Playwright Test Framework with Google Chrome execution target.

---

## 3. In Scope and Out of Scope

### 3.1 In Scope
* Functional verification of authentication flows for `USER1` with `Administrator` role.
* Positive login flow with valid credentials (`REQ-01`).
* Negative login handling with incorrect passwords (`REQ-02`).
* Client-side and server-side validation on empty required fields (`REQ-03`).
* Explicit logout functionality and session invalidation (`REQ-04`).
* Protection of internal routes post-logout (e.g., verifying back-button and direct URL access redirects to login) (`REQ-04`).
* Automated end-to-end regression scripts using Playwright on Google Chrome.

### 3.2 Out of Scope
* Single Sign-On (SSO) and Social Authentication integrations.
* Multi-Factor Authentication (MFA) and One-Time Passwords (OTP).
* Self-service user registration and account onboarding.
* Password reset, forgot password, and account unlocking self-service workflows.
* Non-functional performance/load testing and penetration/vulnerability scanning.
* Administrative user management outside the authentication boundary.

---

## 4. Requirements and Planned Coverage

The following Requirement Traceability Matrix (RTM) maps the provided requirements to specific positive, negative, and edge test scenarios planned for manual validation and Playwright automation:

| Req ID | Scenario ID | Scenario Description | Test Type | Input Data / Preconditions | Expected Observable Result | Automation Candidate (Playwright / Chrome) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **REQ-01** | `TC-AUTH-001` | Sign in with valid `USER1` credentials | Positive Functional | Active `USER1` account, correct password | Successful authentication, redirection to Admin Dashboard, session cookie/token created | Yes (Smoke / Regression) |
| **REQ-01** | `TC-AUTH-002` | Verify persistent session state upon page reload | Functional / Session | Logged in as `USER1` | Page refreshes; user remains logged in without re-authenticating | Yes (Regression) |
| **REQ-02** | `TC-AUTH-003` | Attempt login with valid username and incorrect password | Negative Functional | Active `USER1` username, invalid password string | Authentication rejected; generic error message displayed; password field cleared; user remains on login page | Yes (Regression) |
| **REQ-02** | `TC-AUTH-004` | Multiple consecutive incorrect password submissions | Negative / Security Edge | Active `USER1` username, incorrect password (x3) | Authentication rejected; account status remains verifiable; no sensitive stack traces exposed | Yes (Regression) |
| **REQ-03** | `TC-AUTH-005` | Attempt login with empty username and valid password | Negative / Validation | Username: `""`, Password: valid | Form submission prevented; inline validation message displayed for username; no auth network call initiated | Yes (Regression) |
| **REQ-03** | `TC-AUTH-006` | Attempt login with valid username and empty password | Negative / Validation | Username: `USER1`, Password: `""` | Form submission prevented; inline validation message displayed for password; no auth network call initiated | Yes (Regression) |
| **REQ-03** | `TC-AUTH-007` | Attempt login with both fields empty | Negative / Validation | Username: `""`, Password: `""` | Form submission prevented; validation messages shown on both required fields | Yes (Regression) |
| **REQ-04** | `TC-AUTH-008` | User clicks Logout from active session | Positive / Session | Active session for `USER1` on dashboard | Session invalidated server-side; redirected to Login screen; session token revoked | Yes (Smoke / Regression) |
| **REQ-04** | `TC-AUTH-009` | Access protected route via browser Back button after logout | Security / Session Edge | Logged out successfully (`TC-AUTH-008`), browser Back clicked | Protected page is not rendered from cache; user is redirected back to Login screen with session expired state | Yes (Regression) |
| **REQ-04** | `TC-AUTH-010` | Direct deep-link URL navigation to Admin route post-logout | Security / Session Edge | Logged out state; direct GET navigation to `/admin/dashboard` | Access denied; user redirected to `/login` with prompt to authenticate | Yes (Regression) |

---

## 5. Test Approach, Levels, and Types

### 5.1 Test Levels
1. **Integration / API Level:** Validate authentication endpoint contracts, response status codes (200 OK vs 401 Unauthorized), and secure cookie/token headers (`HttpOnly`, `SameSite`, `Secure`).
2. **System / End-to-End (E2E) Level:** Full end-to-end browser execution through Google Chrome verifying DOM states, navigation, redirects, and visual feedback.

### 5.2 Test Types
* **Smoke Testing:** Minimal acceptance check (`TC-AUTH-001` and `TC-AUTH-008`) verifying the login interface loads and authenticates successfully on build deployment.
* **Positive Functional Testing:** Verifying standard administrative user paths and correct dashboard landing views.
* **Negative & Boundary Testing:** Verifying error feedback for malformed, blank, and invalid credential inputs without exposing application stack traces.
* **Session & Security Testing:** Verifying post-logout cookie invalidation, local/session storage cleanup, and route guards against backward history navigation.
* **Automated Regression Suite:** Automated execution using Playwright to ensure continuous stability upon code changes.

### 5.3 Automation Strategy (Playwright)
* **Architecture:** Page Object Model (POM) separating locator selectors from test assertions.
* **Execution Target:** Google Chrome (`channel: 'chrome'`).
* **Test Isolation:** Independent browser context per test to prevent session leakage between test cases.

---

## 6. Environment, Tools, Access, and Test Data

### 6.1 Test Environment
* **Target Application URL:** `https://CRM.com` (Proposed staging/QA endpoint: `https://qa.crm.com` or `https://staging.crm.com`).
* **Execution Browser:** Google Chrome (Latest stable release) via Playwright Chromium channel.
* **Operating System:** Windows / Cross-platform CI runners.

### 6.2 Tooling Stack
* **Test Automation Framework:** Playwright (`@playwright/test`).
* **Language & Runtime:** JavaScript / TypeScript on Node.js LTS.
* **Assertion Engine:** Playwright built-in expect matchers (web-first assertions).
* **Test Reporting:** Playwright HTML Reporter, trace viewer, and video recordings on failure.

### 6.3 Test Data & Access Management
* **Target Role:** `Administrator` account (`USER1`).
* **Credentials Management:**
  * Secrets will **not** be committed into source code.
  * Credentials will be injected via environment variables:
    * `CRM_BASE_URL`: Base URL for the CRM instance.
    * `CRM_ADMIN_USER`: Username / Email for `USER1`.
    * `CRM_ADMIN_PASS`: Password for `USER1`.
* **Data State Requirements:** `USER1` must be seeded in active status with confirmed administrative permissions and no password-reset flag pending.

---

## 7. Entry and Exit Criteria

### 7.1 Proposed Entry Criteria (Gate to Begin Test Execution)
1. **Environment Readiness:** Target CRM build deployed to QA/Staging environment and responding with HTTP 200 on login endpoint.
2. **Account Provisioning:** `USER1` verified active in database with valid credentials and Administrator role permissions.
3. **Configuration & Secrets:** Environment variables and network access (VPN/firewall rules) confirmed on execution runners.
4. **Smoke Gate:** Smoke test suite (`TC-AUTH-001`) executed and passed successfully.

### 7.2 Proposed Exit Criteria (Gate for Release / Sign-off)
1. **Execution Completeness:** 100% of planned test cases (`TC-AUTH-001` through `TC-AUTH-010`) executed on Google Chrome.
2. **Pass Rate Threshold:** 
   * Minimum **100% pass rate** on critical-path functional scenarios (`REQ-01`, `REQ-04`).
   * Minimum **95% overall pass rate** across all combined edge/validation scenarios.
3. **Defect Thresholds:**
   * **0 open Severity 1 (Blocker)** defects.
   * **0 open Severity 2 (Critical)** defects.
   * Any open Severity 3 (Major) or Severity 4 (Minor) defects documented with confirmed workarounds and accepted by the Product Owner.
4. **Automation Artifacts:** Playwright HTML report and execution logs generated and archived.

---

## 8. Roles, Responsibilities, Estimates, and Schedule

### 8.1 RACI Matrix
| Role | Person / Group | Responsibilities |
| :--- | :--- | :--- |
| **Senior QA Automation Engineer** | QA Team | Author test plan, design test cases, build Playwright automation scripts, execute test runs, triage defects |
| **Software Developers** | Dev Team | Review test plan, deploy QA builds, provide fix turnarounds for reported bugs |
| **Product Owner** | Business / Product | Approve test plan, sign off on defect deferrals and final release acceptance |
| **DevOps / Infrastructure** | Infra Team | Maintain QA staging environments, database seed scripts, and CI runners |

### 8.2 Proposed Estimates & Schedule (Effort Hours)
* **Test Plan Review & Finalization:** 4 hours
* **Playwright POM & Script Development (10 scenarios):** 16 hours
* **Manual & Automated Test Execution / Verification:** 8 hours
* **Defect Retesting & Verification:** 6 hours
* **Test Summary Report & Sign-off:** 2 hours
* **Total Estimated Effort:** 36 hours (~4.5 working days)

---

## 9. Defect Management and Reporting

### 9.1 Defect Severity Classification
* **Severity 1 (Blocker):** Unable to sign in with valid credentials (`REQ-01`); application crash; data loss.
* **Severity 2 (Critical):** Logout fails to terminate session (`REQ-04`); authenticated routes accessible post-logout; security bypass.
* **Severity 3 (Major):** Empty field validation fails to prevent submission (`REQ-03`); inaccurate error messaging for incorrect password (`REQ-02`).
* **Severity 4 (Minor / Cosmetic):** UI alignment glitches, typographical errors in warning banners, styling defects.

### 9.2 Defect Triage & Lifecycle
* **Reporting Standard:** Every defect report must contain:
  1. Summary & Requirement ID (`REQ-01` to `REQ-04`).
  2. Exact steps to reproduce and environment details.
  3. Actual result vs. Expected observable result.
  4. Playwright trace / screenshot / video attachment.
* **Triage Cadence:** Daily defect review meeting with QA, Dev Lead, and Product Owner during active execution.

---

## 10. Risks, Dependencies, Assumptions, and Open Questions

### 10.1 Risks and Mitigations
| Risk Description | Severity | Likelihood | Mitigation Strategy |
| :--- | :--- | :--- | :--- |
| **Dynamic UI Selectors:** Frequent frontend updates causing locator instability in Playwright | High | Medium | Use robust, user-facing locators (`getByRole`, `getByLabel`, `getByTestId`) rather than fragile CSS/XPath paths. |
| **Environment Instability:** Downtime on staging CRM endpoint causing false-positive test failures | High | Medium | Implement automated health-check probe in Playwright `globalSetup` before triggering test suite. |
| **Session Cache Retention:** Browser caching retaining protected page states post-logout | Medium | High | Test explicitly with cache-disabled headers and verify route redirection under fresh contexts. |

### 10.2 Dependencies
* Availability and uptime of the CRM Portal staging instance.
* Stable database seeding mechanism to reset `USER1` state before test cycles.

### 10.3 Assumptions
* Standard cookie or bearer token based authentication model.
* Google Chrome is the primary client browser for CRM Portal administrative operations.
* No CAPTCHA or bot-detection mechanism is active on the designated QA environment for `USER1`.

### 10.4 Open Questions
* Are there account-lockout policies (e.g., account lock after 5 failed password attempts) that need specific assertion in `TC-AUTH-004`?
* What are the exact localized validation text strings for empty fields and invalid passwords?

---

## 11. Suspension and Resumption Criteria

### 11.1 Suspension Criteria
Testing execution will be formally suspended if any of the following occur:
1. Target CRM Portal environment is inaccessible or returning 5xx Server Errors on the login landing page for > 30 minutes.
2. `USER1` credentials fail to authenticate due to environment-level configuration or database seeding issues (blocking `REQ-01`).
3. Core smoke test fails with a Severity 1 defect that blocks access to the administrative module.

### 11.2 Resumption Criteria
Testing will resume when:
1. DevOps/Dev confirms deployment of a stable patch addressing the blocking issue.
2. Successful verification of the smoke test suite in the target environment.
3. Relevant test cases for the fixed defect and affected regression areas are scheduled for execution.

---

## 12. Test Deliverables and Approval

### 12.1 Planned Deliverables
* **Test Plan Document:** This document (`TP-CRM-AUTH-001`).
* **Automated Test Suite:** Playwright test scripts covering scenarios `TC-AUTH-001` through `TC-AUTH-010`.
* **Execution Logs & Reports:** Playwright HTML report, failure traces, and execution run logs.
* **Defect Reports:** Documented defect tickets in the issue tracker.
* **Test Summary Report (TSR):** Final QA sign-off document detailing actual execution metrics against the exit criteria.

*(Note: In accordance with standard QA governance, this document defines planned tests; no execution results or release readiness are claimed prior to formal test execution.)*

### 12.2 Approval & Sign-Off Block
| Stakeholder Role | Name / Title | Signature / Status | Date |
| :--- | :--- | :--- | :--- |
| **Senior QA Engineer** | Senior QA Lead | Submitted for Review | Proposed |
| **Development Lead** | Lead Software Engineer | Pending Review | Proposed |
| **Product Owner** | CRM Product Owner | Approved | Approved |
