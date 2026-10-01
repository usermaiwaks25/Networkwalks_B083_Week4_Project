# Networkwalks_B083_Week4_Project
[README.md](https://github.com/user-attachments/files/32937247/README.md)
# Mediroza General Hospital — Penetration Testing Report

> **Networkwalks | Batch B083 | Week 4**

A controlled black-box penetration testing assessment of the Mediroza General Hospital web application, conducted as part of the Networkwalks Week 4 project.

**Target:** `https://medirozahospital.com`  
**Assessment Type:** Full Black-Box Penetration Test  
**Duration:** 5 Days  
**Tester:** Ibrahim Usman Maiwake

> ⚠️ **Authorization Notice**
>
> This assessment was performed in a controlled educational environment with written authorization. Testing was limited to the target domain. Social engineering and denial-of-service testing were excluded.

---

## 📌 Overview

The assessment followed the Week 4 milestones:

1. Obtain the three confidential patient laboratory reports.
2. Recover the contents of the protected PDF files.
3. Investigate the recovered material for further critical data exposure.

The assessment identified weaknesses involving authentication, user enumeration, SQL injection, weak PDF passwords, metadata information disclosure, directory indexing, and exposure of a database backup.

---

## 🧰 Tools Used

| Tool | Purpose |
|---|---|
| **Nmap** | Reconnaissance and service enumeration |
| **nslookup** | DNS reconnaissance |
| **Burp Suite** | Web/application analysis |
| **THC Hydra** | Username/password testing |
| **SecLists** | Username wordlist |
| **RockYou** | Password dictionary |
| **Online HashCrack** | Password/hash recovery |
| **Networkwalks PasswordCracker** | Password recovery |
| **qpdf** | PDF decryption |
| **exiftool** | PDF metadata analysis |

---

## 🔎 Assessment Flow

```text
Reconnaissance
      ↓
Authentication Analysis
      ↓
Username Enumeration
      ↓
SQL Injection / Authentication Weakness
      ↓
Access to Patient Portal
      ↓
Retrieve 3 Protected PDF Reports
      ↓
Recover PDF Passwords
      ↓
Decrypt Reports
      ↓
Analyze PDF Metadata
      ↓
Discover /old/ Directory
      ↓
Directory Indexing
      ↓
Download SQL Database Backup
      ↓
Identify Sensitive Staff & Shareholder Data
```

---

# 1. Reconnaissance

Initial reconnaissance was performed against the target domain to understand the exposed web infrastructure and identify areas for further testing.

### Evidence

![Reconnaissance evidence](images/evidence-1.png)

---

# 2. Authentication & Username Enumeration

The patient portal returned different responses depending on whether the supplied username existed.

Examples observed during the assessment included:

- `Username not found`
- `Incorrect password`

This difference in responses could be used to distinguish valid usernames from invalid ones.

Using **THC Hydra** with a SecLists username wordlist, the username:

```text
admin
```

was successfully identified.

### Evidence

![Username enumeration evidence](images/evidence-2.png)

---

# 3. SQL Injection / Authentication Weakness

Further testing of the login functionality produced a `mysqli_query()` warning containing a SQL syntax error.

This behaviour indicated that user-controlled input was reaching a database query without adequate protection.

The assessment subsequently demonstrated successful authentication compromise against the identified account using a password dictionary.

### Evidence

![SQL injection evidence](images/evidence-3.png)

---

# 4. Access to Patient Laboratory Reports

After authentication, three password-protected patient laboratory reports were obtained:

- **S. Dlamini**
- **P. Reddy**
- **E. Thompson**

The reports contained sensitive laboratory information.

### Evidence

![Patient reports evidence](images/evidence-4.png)

---

# 5. Weak PDF Password Protection

The three PDF reports were protected with weak passwords:

| Patient | Recovered Password |
|---|---|
| S. Dlamini | `123456` |
| P. Reddy | `password` |
| E. Thompson | `!@#$%^&` |

The recovered passwords were used to decrypt the documents with `qpdf`.

The decrypted reports contained laboratory results, including:

- S. Dlamini — high White Cell Count
- P. Reddy — high Total Cholesterol and LDL
- E. Thompson — low Haemoglobin and Vitamin D

### Evidence

![PDF password recovery evidence](images/evidence-5.png)

---

# 6. PDF Metadata Information Disclosure

Metadata analysis was performed against the decrypted PDF documents using **exiftool**.

The E. Thompson report exposed information including:

```text
Creator: j.malik
```

The metadata also contained a developer comment referring to a database backup that had been moved to an `/old` directory before a site migration.

This provided an operational clue for further investigation.

### Evidence

![PDF metadata evidence](images/evidence-6.png)

---

# 7. Directory Indexing

The `/old/` directory was accessible from the web application and directory indexing was enabled.

This allowed the contents of the directory to be listed.

### Evidence

![Directory indexing evidence](images/evidence-7.png)

---

# 8. Exposed Database Backup

Directory listing revealed the following SQL backup:

```text
mediroza_db_backup_2019.sql
```

The backup was downloadable from the web-accessible directory.

Analysis of the backup exposed sensitive organizational information, including hospital staff salary information and shareholder details.

### Evidence

![Database backup evidence](images/evidence-8.png)

---

# 9. Sensitive Data Exposure

The database backup demonstrated that sensitive business information could be obtained from a file that should not have been publicly accessible.

This finding significantly increased the impact of the earlier server-configuration weakness.

### Evidence

![Sensitive data evidence](images/evidence-9.png)

---

# ⚠️ Key Findings

| Finding | Severity |
|---|---|
| Exposed Database Backup | **Critical** |
| SQL Injection / Authentication Weakness | **Critical** |
| Weak PDF Passwords | **High** |
| Username Enumeration | **Medium** |
| Directory Indexing | **Medium** |
| Metadata Information Leakage | **Low** |

> Severity labels above reflect the assessment documented in the project report.

---

# 🛠️ Recommendations

### 1. Remediate SQL Injection

Use parameterized queries/prepared statements for database interactions and validate user input on the server side.

### 2. Prevent Username Enumeration

Use a generic authentication response such as:

```text
Invalid username or password
```

Avoid revealing whether a username exists.

### 3. Implement Brute-Force Protection

Add appropriate rate limiting, progressive delays, and account protection controls.

### 4. Disable Directory Indexing

Disable directory browsing on the web server, especially for directories containing archived or operational files.

### 5. Remove Public Database Backups

Database backups should never be stored inside a publicly accessible web root.

Move backups to protected storage with appropriate access controls.

### 6. Strengthen PDF Protection

Use strong, unique passwords and appropriate document access controls for sensitive patient reports.

### 7. Remove Sensitive Metadata

Scrub author, creator, developer comments, and other unnecessary metadata before documents are published.

### 8. Review Potentially Exposed Credentials and Secrets

Where credentials or secrets may have been exposed during the assessment, review and rotate them as appropriate.

---

# 📊 Security Impact

The assessment demonstrated how several individually important weaknesses could form a chain:

```text
Verbose Authentication Errors
          ↓
Username Enumeration
          ↓
Authentication Weakness
          ↓
Patient Portal Access
          ↓
Sensitive Patient Reports
          ↓
Weak PDF Passwords
          ↓
Metadata Disclosure
          ↓
/old/ Directory Discovery
          ↓
Database Backup Exposure
          ↓
Sensitive Organizational Data
```

---

# 📁 Repository Structure

```text
mediroza-pentest/
│
├── README.md
│
└── images/
    ├── evidence-1.png
    ├── evidence-2.png
    ├── evidence-3.png
    ├── evidence-4.png
    ├── evidence-5.png
    ├── evidence-6.png
    ├── evidence-7.png
    ├── evidence-8.png
    └── evidence-9.png
```

---

# 📄 Full Report

The complete penetration testing report contains the detailed methodology, findings, evidence, risk ratings, and remediation recommendations.

**Assessment:** Mediroza General Hospital  
**Project:** Networkwalks Batch B083 — Week 4  
**Tester:** Ibrahim Usman Maiwake

---

## ⚖️ Disclaimer

This repository documents an authorized educational penetration testing exercise. The techniques and evidence presented here are intended for authorized security testing, learning, and defensive security purposes only.

Do not reproduce these techniques against systems without explicit authorization.
