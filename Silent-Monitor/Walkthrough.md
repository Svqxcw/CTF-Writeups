
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



