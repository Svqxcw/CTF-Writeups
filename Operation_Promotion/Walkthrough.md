### 🎯 Environment Overview & Task Objective

This section provides the active infrastructure details along with the primary scope of the assessment:

* **Attacker Machine:** `10.114.65.203` 
* **Target Machine (RecruitCorp):** `10.114.173.87` 
* **Objective:** Perform a solo penetration test against RecruitCorp's public-facing portal, fully compromise the target host, and capture all flags.
<img alt="Zrzut ekranu 2026-09-7 o 11 52 01" src="https://github.com/user-attachments/assets/05e12181-745e-4362-869b-f0ff2912ae8c" />

### 🕵️ Passive Reconnaissance & Manual Inspection

To start the assessment without triggering security alarms, we navigate directly to the target web application at `http://10.114.173.87` in the browser. 

Passive reconnaissance allows us to understand the application's layout, structure, and potential entry points without sending aggressive automated scans:

* **Navigating to the Target:** Accessing the main landing page reveals the **RecruitCorp - Careers Portal**.
* **Source Code Inspection:** Inspecting the underlying HTML source code (`Ctrl + U`) to look for hidden developer comments, API endpoints, or leaked credentials.
* **Element Analysis:** Checking visible interactive elements, such as job application forms or search inputs, for potential vulnerabilities (like XSS or SQLi).
* **Stealth Advantage:** Manual exploration ensures we learn how the site behaves normally before running noisy automated tools.
<img alt="Zrzut ekranu 2026-09-7 o 11 56 39" src="https://github.com/user-attachments/assets/040fd55f-7b09-41b8-b41f-4672006397f2" />

