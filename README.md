# Web-Application-Penetration-Test-EduCal-Calendar-Platform-
 A sanitised project of an independent, authorised security assessment of an EdTech calendar application. All client-identifying information, live endpoints, credentials, and personal data have been redacted. This repository documents methodology, findings, and remediation verification.

**Role:** Independent Security Assessor (solo engagement)
**Type:** Grey/white-box web application & API penetration test
**Authorisation:** Conducted under a signed Rules of Engagement (ROE)
**Framework:** OWASP Top 10 (2021)

---

## Summary

EduCal (a pseudonym) is a web-based calendar platform used by an education provider to manage class schedules and student information. I performed an end-to-end security assessment covering the frontend, the API layer, and the cloud backend.

The assessment identified **13 findings, including 5 Critical**. The most significant issue was a complete absence of authentication across all API endpoints, allowing any unauthenticated party to read the entire user database (including plaintext passwords), create accounts, escalate privileges, and delete data. Additional findings included insecure credential storage, an over-permissive CORS policy exposing data to any website, and the exposure of student personal data in breach of applicable data-protection obligations.

All Critical findings were reported, remediated by the development team, and **verified as resolved through re-testing**.

---

## Engagement Lifecycle

1. **Scoping & Authorisation** — Defined scope and boundaries; executed a signed Rules of Engagement.
2. **Reconnaissance & Static Analysis** — Reviewed the application source, ran static analysis to surface code-level issues using KALI-LINUX.
3. **Manual Testing & Exploitation** — Exercised the API and authentication layer against the OWASP Top 10; confirmed vulnerabilities with working proof-of-concept requests.
4. **Reporting** — Produced a technical report with severity ratings, reproduction steps, business impact, and remediation guidance.
5. **Remediation Support** — Advised the developer on fixes (authentication middleware, password hashing, CORS hardening, authorisation checks).
6. **Verification Re-testing** — Re-ran every Critical exploit after fixes to confirm resolution.

---

## Key Findings (Sanitised)

| ID | Finding | Severity | Status |
|----|---------|----------|--------|
| 01 | Missing Subresource Integrity on external CDN | Medium | | ✅ Verified fixed |
| 02 | Unsafe format string in error logging | Low | | ✅ Verified fixed |
| 03 | Unauthenticated access to full user database | **Critical** | ✅ Verified fixed |
| 04 | Unauthenticated access to calendar data incl. student PII | **Critical** | ✅ Verified fixed |
| 05 | Unauthenticated account creation | **Critical** | ✅ Verified fixed |
| 06 | Unauthenticated user & event deletion | **Critical** | ✅ Verified fixed |
| 07 | Plaintext password storage | **Critical** | ✅ Verified fixed |
| 08 | Password returned in login response | High | ✅ Verified fixed |
| 09 | Timing-based username enumeration / no rate limiting | Low | Reported |
| 10 | CORS wildcard exposing API to any origin | High | ✅ Verified fixed |
| 11 | Privilege escalation via client-controlled role | **Critical** | ✅ Verified fixed |
| 12 | Account takeover via record overwrite | **Critical** | ✅ Verified fixed |
| 13 | Outdated build dependency (known advisory) | Low | | ✅ Verified fixed |

Detailed sanitised writeups for selected findings are in [`/findings`](./findings).

---

## Highlighted Finding: Unauthenticated Database Access

**Category:** A01 Broken Access Control / A02 Cryptographic Failures
**Severity:** Critical

The API served the complete user database in response to an unauthenticated request. No token, session, or credential was required. The response included usernames, plaintext passwords, roles, and identifiers for every account.

**Proof of concept (sanitised):**
```
GET https://api.example.com/backend?route=/users
(no authentication headers)

→ 200 OK
[ { "username": "User A", "password": "[REDACTED]", "role": "USER", ... }, ... ]
```

**Impact:** Any party aware of the API endpoint could retrieve every user credential in a single request. Combined with plaintext storage, this constituted full account compromise for all users, and exposed student personal data.

**Remediation:** Authentication middleware enforcing a valid token on all endpoints; passwords migrated to bcrypt hashing; password field removed from all API responses.

**Verification:** After remediation, the same request returned `401 Unauthorized`.

---

## Data-Protection (PDPA) Dimension

Beyond the technical issues, several findings had direct data-protection implications under Singapore's Personal Data Protection Act (PDPA):

- Student personal data (names, schedules) was accessible without authentication, contravening the requirement to protect personal data with reasonable security arrangements.
- Credentials were stored without hashing.

I documented these findings with their regulatory context, including the obligation to protect personal data regardless of organisation size and the existence of breach notification requirements. This connected technical vulnerabilities to compliance risk

---

## Methodology & Tools

**Approach:** Risk-based testing, prioritising the highest-impact attack surface (the unauthenticated API layer) first, then working through the OWASP Top 10 systematically.

**Tools used:**
- **Postman** — primary tool for API exploitation and re-testing
- **Semgrep** — static code analysis
- **Browser DevTools** (Network / Console) — endpoint discovery and cross-origin (CORS) testing
- **npm audit** — dependency / supply-chain analysis
- **Kali Linux** — testing environment
- **Obsidian** — findings documentation

---

## OWASP Top 10 (2021) Coverage

| Category | Coverage |
|----------|----------|
| A01 Broken Access Control | Tested |
| A02 Cryptographic Failures | Tested |
| A03 Injection | Partially tested (XSS tested, not exploitable) |
| A04 Insecure Design | Tested |
| A05 Security Misconfiguration | Tested |
| A06 Vulnerable & Outdated Components | Tested |
| A07 Identification & Authentication Failures | Tested |
| A08 Software & Data Integrity Failures | Tested |
| A09 Security Logging & Monitoring Failures | Confirmed present |
| A10 Server-Side Request Forgery | Not applicable (no server-side URL fetch surface) |

---

## Scope & Limitations

This was a **scoped assessment**, not a guarantee that the application or its wider infrastructure is free of all vulnerabilities. The following were explicitly out of scope and recommended as follow-up:

- Systematic input-validation / injection fuzzing across all fields
- Formal threat modelling
- Full AWS infrastructure review (IAM least-privilege, storage exposure)
- JWT implementation hardening review

---

## Note on This Repository

This is a **sanitised portfolio project**. All client names, live endpoints, credentials, tokens, identifiers, and personal data have been removed or replaced with placeholders. It is shared with the client's permission for portfolio use. No real data is contained in this repository or its commit history.

