# Active Directory Attack & Defense Lab

A full **attack-and-defend** walkthrough of an Active Directory domain, built from scratch in an isolated lab. It chains the standard AD kill chain up to **Domain Admin** — then switches sides and maps every step to the Windows event that detects it.

> **Environment & authorisation.** All testing was performed against `adlab.local`, a self-built, **deliberately vulnerable** Active Directory domain running on an **isolated virtual network** (VirtualBox internal network, no internet access), with Kali Linux as the attacker. No real systems, accounts, or data are involved. Every credential shown exists only inside this lab.

## 📄 Read the report

**[Active_Directory_Attack_Defense_Lab.pdf](./Active_Directory_Attack_Defense_Lab.pdf)** — full report with the attack narrative, CVSS-rated findings, MITRE ATT&CK mapping, log evidence, and remediation (renders in your browser).

## What this project demonstrates

- Building a realistic AD lab (Server Core Domain Controller + domain-joined Windows client) and attacking it end to end
- **AS-REP roasting** and **Kerberoasting** — extracting and cracking Kerberos credentials offline
- Lateral movement via **credential reuse**, then privilege escalation to **Domain Admin**
- Attack-path analysis with **BloodHound**
- **Blue-team detection** — mapping each attack step to the Windows Security Event ID that records it, the half of the engagement most write-ups skip
- Professional reporting: a risk register with **CVSS v3.1**, **MITRE ATT&CK** mapping, and prioritised remediation

## Attack chain (summary)

1. **Initial access** — foothold as a standard domain user (`jdoe`), enumeration over SMB
2. **AS-REP roasting** — `tlee` (Kerberos pre-auth disabled) → hash cracked offline
3. **Kerberoasting** — `svc_sql` service account (has an SPN) → hash cracked offline
4. **Lateral movement** — the recovered password is reused as the workstation's local administrator
5. **Privilege escalation** — the same password is also the domain administrator's → full control of `adlab.local`

## Findings (risk register)

| ID | Finding | CVSS | Severity |
|---|---|---|---|
| F-01 | Password reuse (local admin = domain admin) | 9.0 | Critical |
| F-02 | Kerberoastable service account with a weak password | 8.1 | High |
| F-03 | AS-REP roastable account (pre-auth disabled) | 7.5 | High |
| F-04 | Over-privileged user (Account Operators) | 6.5 | Medium |
| F-05 | Weak, guessable account passwords | 5.9 | Medium |

## Detection mapping (blue team)

| Attack step | MITRE ATT&CK | Event ID | Key indicator |
|---|---|---|---|
| AS-REP roasting | T1558.004 | 4768 | Pre-authentication type **0** |
| Kerberoasting | T1558.003 | 4769 | Encryption type **0x17 (RC4)** |
| Lateral movement | T1021 / T1047 | 4624 | Logon type **3** from the attacker |
| Privilege escalation | T1098 | 4728 | Member added to **Domain Admins** |

## Tools

BloodHound · NetExec (nxc) · Impacket · hashcat · Sysmon · Windows Event Logs · Kali Linux

## What I'd do differently next

- Replace the password-reuse escalation with a real **ACL / delegation path** discovered through BloodHound
- Write a **Sigma rule** for one of the detections to close the loop between the offensive and defensive halves
