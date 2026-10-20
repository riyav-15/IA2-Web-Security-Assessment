# Web Application Authentication and Access Control Security Assessment

**Course:** Honors Cyber Forensics & Laws (CFL 216H02C501)  
**Assessment:** Internal Assessment 2 (IA-2)  

### Team Members
- **Krishna Rathod** — `16010125801`
- **Riya Vikam** — `16010125802`
- **Khushi Satam** — `16010125829`

---

## 1. Project Overview

This project conducts a focused security assessment of web application authentication, access control mechanisms, and session management behaviors. By leveraging industry-standard web proxy tools and controlled lab environments, this assessment investigates how common implementation flaws—such as verbose authentication errors, unvalidated client parameters, and improper session handling—can compromise application security.

---

## 2. Objectives

- **Authentication Resilience:** Evaluate login interfaces for information disclosure vulnerabilities enabling automated username enumeration.
- **Access Control Verification:** Test server-side authorization boundaries against Insecure Direct Object Reference (IDOR) manipulation.
- **Session Lifecycle Analysis:** Audit session cookie flags and inspect session invalidation behavior upon user logout.
- **Remediation & Defense:** Formulate standard mitigation strategies based on secure coding standards and OWASP best practices.

---

## 3. Tools & Environment

- **Burp Suite Community Edition:** Interception proxy, HTTP history analysis, Repeater for manual request crafting, and Intruder for automated parameter analysis.
- **PortSwigger Web Security Academy:** Authorized, isolated web security sandbox environments.
- **Chromium / Firefox:** Web clients configured with local proxy routing and local PortSwigger CA certificates.

---

## 4. Assessment Practicals Summary

| Practical | Focus Area | Brief Description & Outcome | Evidence Folder |
| :--- | :--- | :--- | :--- |
| **Practical 1** | **Username Enumeration** | Used Burp Intruder to compare login responses and identify a valid username in an authorized PortSwigger lab. The lab was successfully completed. | [practical-1-username-enumeration/](screenshots/practical-1-username-enumeration/) |
| **Practical 2** | **Insecure Direct Object References (IDOR)** | Changed a user ID parameter in the authorized lab and accessed another user's account information, demonstrating an access-control weakness. The lab was successfully completed. | [screenshots/practical-2-idor/](screenshots/practical-2-idor/) |
| **Practical 3** | **Session Security Assessment** | Inspected cookie attributes, logout behaviour, and account responses. The results did not conclusively establish a session-invalidation vulnerability. | [screenshots/practical-3-session-security/](screenshots/practical-3-session-security/) |

> For comprehensive technical steps, request/response headers, impact evaluations, and remediation guidelines, refer to [methodology.md](methodology.md).

---

## 5. Repository Structure

```text
IA2-Web-Security-Assessment/
│
├── README.md                                  # Project overview, team details, and practical summary
├── methodology.md                             # In-depth technical methodology, procedures, and mitigations
├── .gitignore                                 # Sensible exclusions for OS files, IDE configs, and caches
│
├── screenshots/                               # Photographic evidence and test artifacts
│   ├── practical-1-username-enumeration/      # Intruder attack results, response diffs, lab completion
│   ├── practical-2-idor/                      # Parameter manipulation, API key extraction, lab completion
│   └── practical-3-session-security/          # Cookie attributes, logout headers, and Repeater tests
│
├── report/                                    # Final project report document (PDF / Word)
│   └── .gitkeep
│
└── presentation/                              # Project defense presentation slide deck (PPTX / PDF)
    └── .gitkeep
```

---

## 6. Documentation & Deliverables

- Detailed step-by-step procedures and technical analysis: [methodology.md](methodology.md)
- Primary deliverables (to be added prior to submission):
  - Academic Assessment Report: [`report/`](report/)
  - Project Slide Deck: [`presentation/`](presentation/)

---

## 7. Ethical Considerations

All penetration testing activities were conducted in strict accordance with academic guidelines and legal standards:
- **Authorized Testing Only:** Testing was strictly confined to designated PortSwigger Web Security Academy training labs.
- **No Production Targets:** No external, unauthorized, or live university systems were targeted or intercepted.
- **Data Protection:** No personal passwords, live session tokens, or sensitive user data are stored or exposed within this repository.

---

## 8. References

- [PortSwigger Web Security Academy](https://portswigger.net/web-security)
- [OWASP Top 10:2021 — A01: Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [OWASP Top 10:2021 — A07: Identification and Authentication Failures](https://owasp.org/Top10/A07_2021-Identification_and_Authentication_Failures/)
- [OWASP Web Security Testing Guide (WSTG v4.2)](https://owasp.org/www-project-web-security-testing-guide/)
