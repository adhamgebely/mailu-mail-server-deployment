# 📧 Mailu Mail Server Deployment – Company Implementation

> Production-like corporate mail server deployment using Mailu, Docker, and Roundcube with real-world troubleshooting and email delivery validation.

---

## 🧠 Overview

This project was developed as part of a real-world deployment for Ghaymah

The objective was not only to install the platform, but also to operate it in a production-like environment while handling real-world challenges such as DNS authentication, spam filtering, Gmail rejection, and service troubleshooting.

---

## 🏢 Project Context

* **Company:** Ghaymah
* **Project Type:** Private Corporate Mail Server
* **User Management:** Admin-controlled (no public registration)
* **Target Users:** Internal company employees
* **Goal:** Provide a secure and reliable internal email communication system

---

## ⚙️ Technology Stack

* **Mail Server:** Mailu
* **Containerization:** Docker & Docker Compose
* **Webmail:** Roundcube
* **Protocols:** SMTP, IMAP
* **Security & Authentication:**

  * SPF
  * DKIM
  * DMARC
  * Reverse DNS (PTR)

---

## 🌐 Environment (Sanitized)

| Component   | Value                                         |
| ----------- | --------------------------------------------- |
| Domain      | example.com                                   |
| Mail Host   | mail.example.com                              |
| Server IP   | 192.0.2.10                                    |
| Admin Email | [admin@example.com](mailto:admin@example.com) |

> Note: `192.0.2.0/24` is a reserved documentation IP range (RFC 5737).

---

## 🧩 System Architecture

```text
User → Webmail → SMTP → Mailu Server → IMAP → Inbox
                         ↓
                    External Mail (Gmail)
```

---

## 🚀 Implementation Steps

### 1. Mailu Setup Wizard

Mailu was configured using the official setup wizard:

`https://setup.mailu.io`

The wizard generated:

* `.env`
* `docker-compose.yml`

### 2. Initial Configuration

* Mail storage path configured
* Main mail domain defined
* Postmaster/admin account created
* TLS enabled using Let's Encrypt
* Rate limits configured
* Admin UI enabled

### 3. Feature Selection

* Webmail: Roundcube
* Antivirus (ClamAV): Enabled
* Oletools: Enabled
* Tika: Disabled

### 4. Network & Exposure

* Public hostname configured
* Internal DNS resolver enabled
* Docker network subnet defined
* HTTPS exposed on port `8443` instead of `443` due to an existing nginx service on the server

### 5. Deployment

```bash
docker compose up -d
docker compose ps
```

### 6. Admin Configuration

Performed through Mailu Admin UI:

* Added the mail domain
* Created employee user accounts
* Generated DKIM keys
* Retrieved DMARC policy values
* Verified Admin UI and Webmail access

---

## 📬 Email Testing Results

### ✔️ Internal Email Delivery

Internal email delivery was successfully verified by sending, receiving, and replying between users hosted on the same Mailu server.

### ✔️ External Email Delivery

External email delivery to Gmail was initially unsuccessful. After troubleshooting and correcting DNS authentication records, external delivery worked successfully.

### Issues Identified

* Missing SPF record
* DKIM not generated yet
* DMARC not published
* Reverse DNS (PTR) mismatch
* New domain / IP reputation

### Resolution

* Generated DKIM keys from Mailu Admin UI
* Published SPF, DKIM, and DMARC records
* Fixed reverse DNS (PTR)
* Re-tested email delivery after DNS propagation

### Final Result

* Gmail accepted emails successfully
* The mail server became compliant with common email authentication requirements
* Both internal and external email delivery were validated

---

## 🌐 DNS Configuration (Critical)

Correct DNS configuration was essential for successful external email delivery.

### Required DNS Records (Sanitized)

```text
example.com                MX   10 mail.example.com
example.com                TXT  "v=spf1 ip4:192.0.2.10 mx -all"
mail._domainkey.example.com TXT "v=DKIM1; p=PUBLIC_KEY_FROM_MAILU"
_dmarc.example.com         TXT  "v=DMARC1; p=none; rua=mailto:postmaster@example.com"
```

### Reverse DNS (PTR)

Configured at the hosting provider:

```text
192.0.2.10 → mail.example.com
```

---

## 🛠️ Commands Used During Deployment

### Docker & Services

```bash
docker --version
docker compose version
docker compose up -d
docker compose ps
docker compose restart
docker compose down
docker restart mailu-front-1
docker logs -f mailu-front-1
```

### Health Check

```bash
docker exec mailu-front-1 curl -k https://localhost/healthz
```

Expected output:

```text
OK
```

### DNS Verification

```bash
dig A mail.example.com +short
dig MX example.com +short
dig TXT example.com +short
dig TXT mail._domainkey.example.com +short
dig TXT _dmarc.example.com +short
dig -x 192.0.2.10 +short
nslookup example.com
nslookup mail.example.com
nslookup -type=MX example.com
nslookup -type=TXT example.com
nslookup 192.0.2.10
```

---

## 🔍 Troubleshooting

### Web Interface Not Loading

This issue was related to the front (nginx) container.

**Fix:**

```bash
docker restart mailu-front-1
docker logs -f mailu-front-1
docker exec mailu-front-1 curl -k https://localhost/healthz
```

### Gmail Rejection / Spam Issues

Root causes included:

* Missing SPF
* Missing DKIM
* Missing DMARC
* Incorrect PTR
* Domain/IP reputation

### Diagnostic Tools Used

* Mail Tester
* MXToolbox
* DNSChecker
* Google Admin Toolbox (CheckMX)

---

## 📚 Key Learnings

* Deploying a mail server is not enough; deliverability and trust are equally important
* DNS authentication records are critical for successful external mail delivery
* Internal email success does not automatically guarantee external deliverability
* Reverse DNS alignment matters for provider trust
* Real-world infrastructure troubleshooting requires both system and networking knowledge

---


## 🔐 Disclaimer

This repository does not expose any production domains, IP addresses, or credentials.

All values have been anonymized for documentation and portfolio purposes only.

---

## 👨‍💻 Author

**Adham Gebely**
