

# 🛡️ Web Investigation - CyberDefenders Writeup
<img width="400" height="237" alt="image" src="https://github.com/user-attachments/assets/f4711160-f970-4583-a6f8-b990e3126570" />


## 📌 Incident Overview
This repository documents a forensic network investigation into a web server compromise at **BookWorld**, an online bookstore. The investigation was initiated following an anomalous spike in database queries and server resource utilization. Utilizing **Wireshark** for packet analysis, this writeup dissects the end-to-end attack lifecycle—from initial web application reconnaissance and SQL injection exploitation to administrative credential compromise and malicious webshell deployment.
---

## 1. Executive Summary
- **Incident Type:** Web Application Attack / Automated SQL Injection & Webshell Deployment
- **Target Application:** BookWorld Online Bookstore (`bookworldstore.com`)[cite: 1]
- **Severity:** Critical
- **Analyst:** (Ahmed Raafat)
- **Overview:** Comprehensive analysis of captured HTTP traffic logs to investigate a web server compromise running on Apache/2.4.52 and Ubuntu[cite: 1]. The investigation uncovered automated exploitation via `sqlmap` (v1.8.3#stable)[cite: 1], database schema enumeration, directory discovery, administrative credential compromise, and the deployment of a malicious PHP webshell.

---

## 2. Attack Chain (Kill Chain Overview)

1. **Reconnaissance & Initial Access:** The threat actor probed the web application's search functionality (`/search.php?search=`) using automated scanning tools to identify vulnerability vectors[cite: 1].
<img width="1889" height="898" alt="Screenshot 2026-10-27 143744" src="https://github.com/user-attachments/assets/50538882-ac0b-49a6-a073-6cd836ffea91" />

2. **Exploitation (SQL Injection):** Execution of advanced SQL injection techniques, including Boolean-based (`AND 1=1`, `AND 1=2`), Time-based (`PG_SLEEP(5)`), Error-based (`EXTRACTVALUE`, `XMLType`), and UNION-based injections (`UNION ALL SELECT`)[cite: 1]. <img width="1239" height="329" alt="Screenshot 2026-10-27 143952" src="https://github.com/user-attachments/assets/ad29589a-b518-40ae-bef1-ca8091357da4" />

3. **Database Enumeration:** Extraction of internal database structures and sensitive schema data, specifically targeting customer information tables (`customers`). <img width="1241" height="317" alt="Screenshot 2026-10-27 150448" src="https://github.com/user-attachments/assets/9ed7e1a3-6c8a-49d8-bd1b-bb18e4c9ec53" />

4. **Credential Compromise & Post-Exploitation:** Accessing hidden administrative directories and authenticating with compromised credentials, culminating in the upload of a malicious PHP webshell (`NVri2vhp.php`) for persistent remote code execution.
<img width="1846" height="858" alt="Screenshot 2026-10-27 151607" src="https://github.com/user-attachments/assets/ac1e9718-2211-473b-9450-cbf014ac379a" />

---


## 3. Indicators of Compromise (IOCs) Table

| Type | Indicator / Value | Description |
| :--- | :--- | :--- |
| **Target Endpoint** | `http://bookworldstore.com/search.php?search=` | Vulnerable search parameter utilized for SQLi[cite: 1] |
| **Tool User-Agent** | `sqlmap/1.8.3#stable` | Automated exploitation framework signature[cite: 1] |
| **Attack Signatures** | `PG_SLEEP(5)`, `EXTRACTVALUE`, `UNION ALL SELECT` | Malicious SQL injection payloads observed in traffic[cite: 1] |
| **Compromised Table**| `customers` | Database table containing sensitive website user records |
| **Hidden Directory** | `/admin/` | Unauthorized administrative path discovered and accessed |
| **Malicious File** | `NVri2vhp.php` | Uploaded PHP webshell for remote code execution |

---

## 4. Detection & Mitigation Recommendations
- **Parameterized Queries:** Enforce strict prepared statements and parameterized queries across all database interactions to completely eliminate SQL injection vectors.
- **WAF Rule Tuning:** Implement and update Web Application Firewall (WAF) rules to detect and block automated tools like `sqlmap` and common payload signatures (`EXTRACTVALUE`, `PG_SLEEP`)[cite: 1].
- **File Upload Hardening:** Restrict directory execution permissions in user upload directories and validate MIME types rigorously.
- **Access Control & Monitoring:** Enforce multi-factor authentication (MFA) for administrative panels and monitor unusual spikes in database error responses (HTTP 500) which often signify error-based exploitation attempts[cite: 1].
