# IT-Support-Intern   
IT Support Intern — Learning Log &amp; Task Documentation


 <img width="218" height="75" alt="image" src="https://github.com/user-attachments/assets/f4bc83ff-2a81-406c-bdbc-4b8c5367dd9d" />


# IT Support Intern — Learning Log & Task Documentation

**Owner:** (Olawoyin Joseph Pamielrin)

**Role:** IT Support Intern

**Date:** *March — November*

---

## Overview

This document records tasks, solutions, tools, and procedures I have learned and performed as an IT Support intern. It is written as an actionable log so other team members or future-you can reproduce the steps, troubleshoot, and onboard faster.

Included: step-by-step task descriptions, commands and automation code, troubleshooting flows, and image placeholders you can replace with actual screenshots.

---

## Table of contents

1. Daily routine & ticket workflow
2. Common tools installed (Chrome, AnyDesk, Zoom, Outlook Classic)
3. Printing issues — diagnosis & fixes
4. Network / connectivity basics
5. Remote support (AnyDesk) checklist
6. Automation scripts (PowerShell / winget / Chocolatey)
7. Useful logs & locations
8. Sample ticket entries
9. Screenshots & how to capture them
10. Appendix: further reading & notes

---

## 1) Daily routine & ticket workflow

* Check ticket queue first thing: prioritize P1 (no authentication / company-wide outages), then P2 (single user unable to work), P3 (requests, installations).
* Triage steps:

  1. Read ticket (summary & attachments).
  2. Reproduce (if possible) or collect environment details: OS, version, network (wired/wifi), error messages, time.
  3. Attempt quick fixes; escalate if blocked.

**Ticket example:**


<img width="1919" height="901" alt="image" src="https://github.com/user-attachments/assets/bf378c73-1a01-4e2b-9d26-6dec91e01785" />
<img width="996" height="754" alt="image" src="https://github.com/user-attachments/assets/dce6a610-2015-48ca-9d11-16db1c6a3af0" />


---

## 2) Common tools installed

Below are step-by-step notes and the commands/scripts I used to install apps on Windows machines. Replace image placeholders with your screenshots.

### Google Chrome — manual

1. Open Microsoft Edge/IE.
2. Go to [https://www.google.com/chrome](https://www.google.com/chrome) (or use internal repository).
3. Download installer and run as Administrator.
4. Accept defaults and finish.

**Screenshot:**

<img width="676" height="322" alt="image" src="https://github.com/user-attachments/assets/5e13f793-1a55-4fb0-92e3-559e884e47b7" />


### Google Chrome — automated (winget / PowerShell)

```powershell
# Run in an elevated PowerShell (Admin)
# Install via winget
winget install --id Google.Chrome -e --accept-package-agreements --accept-source-agreements
```

### AnyDesk — manual

1. Download AnyDesk installer from [https://anydesk.com/en](https://anydesk.com/en).
2. Run the installer and select "Install AnyDesk for unattended access" if requested.
3. Configure unattended access with a strong password if this is a persistent support machine.

**Screenshot placeholder:**

<img width="665" height="326" alt="image" src="https://github.com/user-attachments/assets/2aa926dc-dbcb-49b5-87ac-72ebb5ac8938" />


### AnyDesk — automated (winget)

```powershell
winget install --id AnyDesk.AnyDesk -e --accept-package-agreements --accept-source-agreements
```

### Zoom — manual

1. Download Zoom Client for Meetings.
2. Run the installer; sign in if required.

**Screenshot placeholder:**

<img width="569" height="172" alt="image" src="https://github.com/user-attachments/assets/56f8f152-a632-4ffd-aa68-ff7c92d27802" />


### Zoom — automated

```powershell
winget install --id Zoom.Zoom -e --accept-package-agreements --accept-source-agreements
```

### Outlook Classic / Microsoft Office configuration

Outlook Classic often comes as part of Microsoft 365 or Office365 installs. For single machine configuration: add mail profile -> POP/IMAP/Exchange settings as provided by the mail team.

**Add account:** File → Add Account → enter email → choose Exchange/IMAP(pop) → follow prompts.
<img width="456" height="324" alt="image" src="https://github.com/user-attachments/assets/ea8134ef-8b77-41a5-96b0-f650da727abc" />
<img width="276" height="328" alt="image" src="https://github.com/user-attachments/assets/6de4ad74-49b2-4876-a34a-f1ab48084669" />




## 3) Printing issues — common diagnosis & fixes

**Common symptoms:** "Printer offline", print jobs stuck in queue, driver errors, spooler keeps crashing.

### Step-by-step diagnosis

1. Check physical: is the printer powered on? network cable connected? Wi‑Fi indicator?
2. From the workstation: ping the printer IP (if network printer):

```powershell
ping 192.168.1.50 -n 4
```

3. Check Windows print spooler service:

```powershell
Get-Service -Name Spooler
# to restart:
Restart-Service -Name Spooler -Force
```

4. Clear print queue (if jobs stuck):

```powershell
# Stop spooler
Stop-Service -Name Spooler -Force
# Remove files from spooler queue
Remove-Item -Path "C:\Windows\System32\spool\PRINTERS\*" -Recurse -Force
# Start spooler
Start-Service -Name Spooler
```

5. Reinstall or update printer driver: visit vendor site (HP, Epson, Brother) and install latest driver recommended for the OS.

6. Check Event Viewer for error codes: `Event Viewer -> Windows Logs -> System`.

**Example ticket resolution:**

* Issue: Jobs stuck; spooler crashing.
* Fix: Restart spooler and clear queue, installed updated driver, test print OK.

**Screenshot placeholders:**

<img width="1462" height="531" alt="image" src="https://github.com/user-attachments/assets/b2b22ab1-dbf9-4d46-91ce-5c7eaa56e7e1" />


---

## 4) Network / connectivity basics

**Quick checks:**

* `ipconfig /all` — check DHCP vs static, gateway, DNS.
* `nslookup <host>` — DNS resolution test.
* `tracert ses.com.ng` — path diagnosis.

**Useful commands:**

```powershell
# Network Troubleshooting Script with Explanations

# 1️⃣ Release current IP address
# This command drops your current DHCP-assigned IP configuration.
# Use it when you want to clear or reset your IP address before requesting a new one.
ipconfig /release

# 2️⃣ Renew IP address
# After releasing, this command requests a fresh IP configuration from the DHCP server.
# It helps fix issues where your system fails to obtain or keep a valid IP address.
ipconfig /renew

# 3️⃣ Flush DNS cache
# This clears all cached DNS records stored on your system.
# It’s useful when you’ve changed DNS servers or a website’s IP address has changed,
# and your system is still using old cached DNS data.
ipconfig /flushdns

# 4️⃣ Check DNS resolution
# This command queries DNS to find the IP address of a domain name.
# It’s used to verify if DNS resolution is working properly.
nslookup example.com

# 5️⃣ Trace route to destination
# This shows the path your packets

```


---

## 5) Remote support (AnyDesk) checklist

* Ask user to run AnyDesk and provide their ID.
* Confirm identity (employee ID / email) before connecting.
* When connected: capture logs, explain what you'll do, use "Request permission" for sensitive actions.

**AnyDesk session notes:**

```
Session ID: 123 456 789
Start: 2025-01-15 09:05
Actions: updated printer driver, cleared spooler, configured default printer
End: 2025-01-15 09:22
```
<img width="665" height="326" alt="image" src="https://github.com/user-attachments/assets/bbea8f2d-48e7-4cf1-9147-a03749a7e6ef" />

---

## 6) Automation scripts — a small toolkit

Below are short scripts you can use to automate installation and basic checks on Windows.

### A) PowerShell: Basic audit & install script (winget)

```powershell

winget source update
# Install common apps (Chrome, AnyDesk, Zoom)
$apps = @("Google.Chrome","AnyDesk.AnyDesk","Zoom.Zoom")
foreach ($app in $apps) {
    Write-Host "Installing $app"
    winget install --id $app -e --accept-package-agreements --accept-source-agreements
}
Write-Host "All requested apps attempted."
```

### B) PowerShell: Printer queue cleaner

```powershell
# Save as Clear-PrintQueue.ps1
Stop-Service -Name Spooler -Force
Remove-Item -Path "C:\Windows\System32\spool\PRINTERS\*" -Recurse -Force
Start-Service -Name Spooler
Write-Host "Print queue cleared and spooler restarted."
```

### C) Bash (Linux) — some basic commands learnt during my linux administration course
Basic Script to automate task

```bash
#!/bin/bash
# quick network check
echo "IP:"
ip addr show
echo "DNS resolution for google.com:"
nslookup google.com
```

---

## 7) Useful logs & locations

* Windows Event Viewer: `Event Viewer -> Windows Logs -> System` and `Application`.
* Printer logs: vendor-specific (e.g., C:\Windows\System32\spool\PRINTERS).
* Outlook logs: Enable logging in Outlook (File -> Options -> Advanced -> Enable troubleshooting logging).
* AnyDesk logs: usually found in `%appdata%\AnyDesk`.

---


**Ticket:** Outlook not seeing the outlook file 

```
Symptoms: Outlook not seeing file at the file path
Actions: close the outlook and check the location of where the .pst file is sitting at and always check the one drive might be siiting there
if found cut from the location and paste back to the correct Document location.
Outcome: Open the outlook and the file will remain available 
```
<img width="398" height="173" alt="image" src="https://github.com/user-attachments/assets/c76e1da4-105f-49a3-87ac-9aadbb5d94e4" />

---

## 9) Screenshots & how to capture them

* Windows: use Snipping Tool (Win+Shift+S) or Print Screen.
* Save images into a `images/` folder and replace the placeholders in this document.

**Example folder structure for the project:**

```
IT-Support-Intern/
├─ README.md  (this doc)
├─ images/
│  ├─ chrome_install_page.png
│  ├─ anydesk_install.png
│  ├─ zoom_install.png
│  ├─ outlook_add_account.png
│  └─ spooler_restart.png
└─ scripts/
   ├─ Setup-ITIntern.ps1
   └─ Clear-PrintQueue.ps1
```

---

## 10) Appendix: further reading & tips

* Vendor support pages (HP, Brother, Epson) for drivers and firmware.
* Microsoft docs for Outlook and Office Deployment Tool.
* Security: never store admin credentials in plain text; use company-approved vault.

---

## Quick checklist for closing a ticket

* Confirm user acceptance and test with user present.
* Document steps in the ticket, include screenshots and commands used.
* Close ticket with summary and root cause if found.

---


