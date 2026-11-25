# Penetration Testing Report: Mediroza General Hospital

[![Risk Rating](https://img.shields.io/badge/Overall%20Risk-Critical-red.svg)](#summary-of-findings)
[![Assessment Type](https://img.shields.io/badge/Type-Black--Box%20Web%20App-blue.svg)](#engagement-overview)
[![Target](https://img.shields.io/badge/Target-medirozahospital.com-lightgrey.svg)](#engagement-overview)
[![Date](https://img.shields.io/badge/Date-Nov%2022--24%2C%202026-informational.svg)](#engagement-overview)

## 📋 Overview

This repository contains the black-box penetration testing report and supporting documentation for **Mediroza General Hospital** (`https://medirozahospital.com`). Conducted over a 3-day window (22–24 November 2026) under written client authorization against synthetic/test data, this assessment evaluated the external attack surface of the hospital's patient and staff web portals.

The full formal report is available in [`Mediroza_General_Hospital_Pentest_Report.pdf`](./Mediroza_General_Hospital_Pentest_Report%20(3).pdf).

---

## 🎯 Engagement Overview

| Parameter | Details |
| :--- | :--- |
| **Client / Target** | Mediroza General Hospital (`https://medirozahospital.com`) |
| **Assessment Type** | Black-box, External, Unauthenticated |
| **Timeline** | November 22 – November 24, 2026 (3 Days) |
| **Author** | `@Lwa3er` |
| **Authorization** | Written client authorization (test/training data environment) |
| **Rules of Engagement** | Restricted to target domain; no DoS; no social engineering; no out-of-scope infrastructure testing |

---

## 🔗 Attack Chain & Milestones

During the assessment, a chain of six vulnerabilities was identified that allowed an unauthenticated external attacker to escalate from initial reconnaissance to a full breach of Protected Health Information (PHI), staff HR records, and corporate shareholder data:

```text
[F1: Directory Listing] 
       │  (Mapped /patient/ endpoints: login.php, download.php, portal.php)
       ▼
[F2: SQL Injection in Login] ──► Milestone 1 (M1): Admin Portal Access
       │  (Bypassed auth via `admin'--`)
       ▼
[F3: Broken Access Control]
       │  (Downloaded all encrypted patient PDF lab reports)
       ▼
[F4: Weak PDF Passwords]     ──► Milestone 2 (M2): PHI Decryption
       │  (Cracked passwords offline via John the Ripper + rockyou.txt)
       ▼
[F5: PDF Metadata Disclosure]
       │  (ExifTool revealed comment: "DB backup moved to /old...")
       ▼
[F6: Exposed Legacy DB Backup] ──► Milestone 3 (M3): Full HR & Shareholder Breach
          (Unauthenticated access to `mediroza_hr` SQL dump at `/old`)
```

### Key Milestones Achieved
* **M1 — Initial Access (Authentication Bypass):** Exploited SQL injection in the patient login form (`admin'--`) to gain administrative session access without valid credentials.
* **M2 — Credential Recovery (Weak Document Protection):** Extracted hashes from three retrieved patient pathology PDFs and cracked all passwords offline within minutes using `John the Ripper`:
  * `patient_report_1.pdf`: `123456`
  * `patient_report_2.pdf`: `Password`
  * `patient_report_3.pdf`: `!@#$%^&`
* **M3 — Sensitive Data Exposure (Legacy Backup):** Inspected PDF metadata using `ExifTool`, uncovering an internal comment pointing to `/old`. Navigating to the legacy directory exposed a complete SQL database backup (`mediroza_hr`) containing staff salaries, national IDs, contact details, and corporate shareholder equity records.

---

## 🚨 Summary of Findings

| ID | Finding | Severity | OWASP Category | Summary |
| :--- | :--- | :--- | :--- | :--- |
| **F2** | **SQL Injection in Patient Login** | **CRITICAL** | A03:2021 – Injection | Unsanitized username input allows full authentication bypass into an administrative portal session. |
| **F6** | **Exposed Legacy Database Backup** | **CRITICAL** | A05:2021 – Security Misconfiguration | Unauthenticated public access to a historical SQL dump (`mediroza_hr`) containing HR salaries, PII, and shareholder records. |
| **F3** | **Unauthorised Access to Patient Lab Reports** | **HIGH** | A01:2021 – Broken Access Control | The file-retrieval mechanism (`download.php`) fails to verify record ownership, allowing bulk download of all patient reports. |
| **F4** | **Weak PDF Encryption / Passwords** | **HIGH** | A02:2021 – Cryptographic Failures | Patient PDFs are protected with trivial dictionary passwords crackable offline in seconds. |
| **F5** | **Sensitive Data in PDF Metadata** | **MEDIUM** | A05:2021 – Security Misconfiguration | Internal authoring details and migration comments (`/old` path) are embedded in patient-facing PDFs. |
| **F1** | **Directory Listing Enabled** | **LOW** | A05:2021 – Security Misconfiguration | Web server exposes directory indexes on `/patient/` and `/staff/`, revealing internal PHP endpoints and `error_log`. |

---

## 🛠️ Tooling & Methodology

* **Reconnaissance & Manual Testing:** `curl`, browser developer tools (endpoint mapping, SQLi verification, and access control testing).
* **Metadata Analysis:** `ExifTool` (extracting document properties and hidden authoring comments).
* **Password Auditing:** `pdf2john` / `John the Ripper` with `rockyou.txt` wordlist (offline recovery of weak PDF encryption keys).

---

## 🛡️ Remediation Roadmap

1. **Immediate (Critical Priority):**
   * **Remove `/old` Backup Directory (F6):** Purge the legacy database export from the public web root immediately, audit the server for other migration artifacts, and rotate any exposed credentials or sensitive identifiers.
   * **Patch SQL Injection (F2):** Refactor all authentication and database queries to use parameterized queries / prepared statements.
2. **Short-Term (High Priority):**
   * **Enforce Object-Level Authorization (F3):** Validate session ownership on every `download.php` request before serving patient records.
   * **Strengthen Document Security (F4):** Replace static, weak PDF passwords with strong, randomly generated per-document credentials or rely exclusively on authenticated portal delivery.
3. **Hardening & Hygiene (Medium / Low Priority):**
   * **Sanitize Document Metadata (F5):** Strip internal comments, creator tags, and author metadata in the lab reporting generation pipeline prior to distribution.
   * **Disable Directory Indexing (F1):** Configure the LiteSpeed web server to disable directory listings globally (`Options -Indexes`) and restrict access to `error_log` files.

---

## ⚖️ Disclaimer

*This penetration test was conducted strictly within the boundaries of a signed authorization agreement against a designated training/test environment containing synthetic data. No real patient, employee, or corporate records were accessed or impacted.*