
<img alt="Zrzut ekranu 2026-09-6 o 16 01 33" src="https://github.com/user-attachments/assets/d23818ff-2343-45ad-9841-8ce3811c91ee" />

### 📡 Initial Reconnaissance

Started off with some passive recon by throwing the target IP straight into the browser, but no luck here — the page just refused to connect.
<img alt="Zrzut ekranu 2026-09-6 o 16 02 24" src="https://github.com/user-attachments/assets/b7944c2e-e358-41a7-8528-0b9039b6adac" />

### 🔍 Network Scanning & Service Discovery

I jumped straight into active scanning with Nmap to see what services were actually listening on the target.

```bash
nmap -sC -sV //machine_ip
```
<img alt="Zrzut ekranu 2026-09-6 o 16 06 15" src="https://github.com/user-attachments/assets/040ee6a5-cdac-42b0-9752-6d6a41ba6194" />

### 🌐 Accessing the Web Application

With port **5050** identified, I headed back to the browser and specified the custom port in the URL.

```text
http://10.112.173.195:5050
```
<img alt="Zrzut ekranu 2026-09-6 o 16 08 48" src="https://github.com/user-attachments/assets/91f61b3f-587d-4f3b-8acc-3555cc27e19b" />

### 📂 Web Recon & Directory Brute-Forcing

Once inside the CorpNet portal, I started with some quick manual inspection — checking for hidden forms, potential IDOR vulnerabilities, page source code, and developer comments, but found nothing useful. 

To uncover hidden endpoints, I kicked off directory enumeration using Gobuster with a standard SecLists wordlist[cite: 1]:

```bash
gobuster dir -u [http://10.112.173.195:5050](http://10.112.173.195:5050) -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-small.txt -x html,js,env,php
```
<img alt="Zrzut ekranu 2026-09-6 o 16 10 17" src="https://github.com/user-attachments/assets/b0ddb92a-3f53-44ab-a683-c0be46e39aa4" />

### 🔑 Accessing the Operator Sign-In Page

After Gobuster discovered the `/internal` path, I immediately navigated to this endpoint in the browser.

```text
http://10.112.173.195:5050/internal
```
<img alt="Zrzut ekranu 2026-09-6 o 16 11 20" src="https://github.com/user-attachments/assets/a6632405-df74-4f6e-b2ab-24c16cd0d1cf" />

### 🧪 Testing for Authentication Bypass (SQL Injection)

Finally, we hit an interactive login form to test for security flaws and potential authentication bypasses[cite: 1]. 

I immediately started testing the login fields for SQL Injection (SQLi) vulnerabilities[cite: 1].

```
Admin' OR 1=1--
```
<img alt="Zrzut ekranu 2026-09-6 o 16 12 15" src="https://github.com/user-attachments/assets/bf2f392e-dd86-471d-9c27-e4632fbe0e89" />




