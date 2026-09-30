# NETWORKWALKS-B083---WEEK-4---WEB-APPLICATION-PENETRATION-TEST

## 📌 Project Overview

For Week 04, I conducted a **full black-box penetration test** against the web infrastructure of **Mediroza General Hospital**.

The assessment focused on identifying weaknesses that could allow unauthorized access to restricted areas, exposure of confidential patient information, recovery of protected files, and access to sensitive hospital data.

| | |
|---|---|
| Client | Mediroza General Hospital |
| Target | https://medirozahospital.com/ |
| Target IP | 199.188.201.16 |
| Assessment Type | Full black-box penetration test |
| Duration | 5 days |
| Batch | B083-NetworkWalks |
| Pentester | Balogun Esther |
| Date | 30 September 2026 |
| Authorization | Written permission granted |
| Scope | Target domain only |

 **Liability disclaimer.** All testing was performed under **written authorisation from Networkwalks**, against a target they provided for this exercise, and limited to the target domain only (no denial-of-service, no social engineering). Everything here is for education. Unauthorised testing of systems you do not own is illegal.

---

### Rules of Engagement

Testing was limited to the authorized target domain.

- No social engineering.
- No denial-of-service testing.
- No testing outside the agreed scope.

---

## 🎯 Objectives

The project was divided into four milestones:

- **M1 — Initial Access:** Attack the website and retrieve 3 confidential patient PDF lab reports.
- **M2 — Data Extraction:** Crack the encryption on all 3 retrieved files.
- **M3 — Attack (cracking):** Find staff salaries and shareholder details.
- **M4 — Pentest Report:** Write a professional penetration-testing report for the client.

---
### Milestone 1 -Initial Access
**Objective**
Attack the website and retrieve **three confidential patient PDF laboratory reports** from the authorized lab environment.

## Reconnaissance (footprinting) 
The following reconnaissance tools were used to gather information about the target:

| | |
|---|---|
| Tool | Purpose |
| WHOIS| Gathered domain registration and ownership information |
| WhatWeb | Identified technologies and web technologies used by the target |
| Wafw00f | Checked for the presence and type of Web Application Firewall|
| nslookup | Performed DNS queries to identify relevant DNS information |
| Nikto | Scanned the web server for potentially interesting files, configurations, and known issues |
| robots.txt | Reviewed the site's robots exclusion file for potentially exposed paths and resources |

![whois](f01-whois.png)
![nslookup](f02-nslookup.png)
![whatweb](f03-whatweb.png)
![wafw00f](f04-wafw00f.png)
![dnsrecon](f05-dnsrecon.png)

**Findings:** Namecheap shared hosting (IP redacted-in-summary), LiteSpeed + OpenResty/CDN, Mediroza CMS 1.4.2, **WAF**, **SPF `~all`** (softfail) + **DMARC `p=none`** (e-mail spoofing exposure), **DNSSEC unsigned**.

---
## Patient Portal and Authentication  Testing

I tested the Patient Portal identified during reconnaissance:

**https://medirozahospital.com/patient/login.php**

I made an authentication attempt. I was able to gain access to:

**https://medirozahospital.com/patient/portal.php**

The portal contained three password-protected/encrypted patient laboratory reports.

### Evidence

![Patient Portal SQL injection test and successful access](./03-patient-portal-sqli-access.png)

---

## Retrieved Patient Reports

After gaining access to the patient portal, I reached the **My lab reports** page.

The portal displayed:

1. **Pathology Report — S. Dlamini**  
   Lab Ref: **LR-2024-1187** | **2024-11-04** | PDF (encrypted)

2. **Pathology Report — P. Reddy**  
   Lab Ref: **LR-2024-1192** | **2024-11-05** | PDF (encrypted)

3. **Pathology Report — E. Thompson**  
   Lab Ref: **LR-2024-1205** | **2024-11-06** | PDF (encrypted)

Each report had a **Download** option.

---

##  M2 — Data Extraction

**Objective**
Crack the encryption/password protection on all three retrieved PDF files.

## Password Recovery Approach

After downloading the three encrypted patient reports, I first attempted password recovery with **John the Ripper**, but it did not produce a result.

I then used the **NetworkWalks Hash Calculator** to extract crackable hashes from the three encrypted PDFs.

### Patient PDF Hash Extraction

I processed:

- **patient_report_1.pdf**
- **patient_report_2.pdf**
- **patient_report_3.pdf**

The Hash Calculator generated crackable hashes in **pdf2john / hashcat-compatible format**.

### Password Cracking

After extracting the hashes, I used the **NetworkWalks Password Cracker** to recover the PDF passwords.

I tested the hashes against **multiple wordlists**.

The passwords recovered were:

| Patient PDF | Recovered Password |
|---|---|
| patient_report_1.pdf | ******|
| patient_report_2.pdf | ******** |
| patient_report_3.pdf | ******* |

### Evidence

![Patient PDF password cracking](./05-patient-pdf-password-cracking.png)

### Result

The passwords for all three encrypted patient PDFs were recovered, allowing the protected files to be opened.

---

## Milestone 3 -Attack(Cracking)
**Objective**
Find the staff salaries and shareholder details of the hospital

`robots.txt` advertises a hidden `/old/` directory; **directory listing is enabled**, exposing a full SQL database backup that anyone can download without logging in.

![robots](f06-robots.png)
![old listing](f07-old-listing.png)
![sql backup](f10-sql-backup.png)

The backup exposes every employee's personal data (names, national IDs, phones, **salaries**) and the hospital's **shareholder register** - satisfying Milestone 3. No exploitation required.

> **Why it matters:** never store a database backup inside the web root. This single misconfiguration is a full confidentiality breach.

---

## Risk summary

| Finding | Rating |
|---------|--------|
| Public database backup (staff PII, salaries, shareholders) | Critical |
| SQL injection - authentication bypass | Critical |
| Weak encryption passwords on patient files | High |
| Directory listing enabled | Medium |
| Username enumeration on patient login | Medium |
| E-mail spoofing exposure (SPF `~all` / DMARC `p=none`) | Medium |
| robots.txt discloses sensitive paths | Low |
| No WAF / no DNSSEC / exposed error log | Low |

## Author

**Balogun Esther** - Cybersecurity Professional | Batch B083
Networkwalks Cybersecurity & Ethical Hacking Internship | Week 04
