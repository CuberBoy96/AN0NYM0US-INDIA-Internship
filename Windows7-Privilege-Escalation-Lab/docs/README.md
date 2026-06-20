## 📖 Prologue: A Self-Imposed Challenge
I started out doing a normal payload delivery, just running an `.exe` on a Windows 7 victim machine and getting a basic reverse shell in Linux. But out of curiosity, I challenged myself to gain a higher-privilege shell without even interacting with the Windows 7 machine. 

I treated it like a real-life scenario where I had to play by a strict set of rules:
* **Zero GUI Interaction:** No physical access or manual execution of payloads on the target desktop.
* **Remote Enumeration Only:** Rely strictly on network scanning and remote service enumeration to find an entry point.
* **Start at the Bottom:** Assume the role of an attacker who has only managed to compromise the least-privileged user account.

With just that initial remote foothold, I was able to compromise the whole machine and gain full admin privileges in the end.

---

###
"I started out doing a normal payload delivery, just running an .exe on a Windows 7 victim machine and getting a basic reverse shell in Linux. But out of curiosity, I challenged myself to gain a higher-privilege shell without even interacting with the Windows 7 machine. I treated it like a real-life scenario where I could only scan, enumerate, and start with the least-privileged account. With just that, I was able to compromise the whole machine and gain admin privileges in the end."



## ✍️ Author

Abhay

- 🔴 Red Team Operations
- 🛡️ Detection Engineering
- 🧠 Threat Hunting
- ⚡ Offensive Security Research

---
📆 2026
