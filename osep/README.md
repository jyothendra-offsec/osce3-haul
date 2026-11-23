# OSEP -- PEN-300 Preparation Notes

Personal study notes for authorized **OffSec PEN-300 / OSEP** lab work, condensed
into an index with the standard enumeration and reference commands I reach for most.

> **Scope:** Everything here is for authorized lab and exam practice only. Keep all
> testing inside the written scope of the course, exam, or engagement, and clean up
> afterwards.

---

## Quick Reference -- Defensive Review

- **Scope and safety:** Confirm permitted targets, testing window, allowed techniques,
  and cleanup requirements before lab work.
- **Evidence:** Record timestamps, host/process context, network observations, and the
  outputs needed for a reproducible report.
- **Detection themes to revise:** Office apps launching script or command interpreters;
  unexpected outbound connections; scripts run with no file trail; unusual cross-process
  memory/thread/APC activity; unexpected local-admin membership changes; web-server
  processes touching unusual files or spawning shells.
- **Response checklist:** Preserve logs and evidence, trace process ancestry and network
  destinations, compare accounts/binaries against baseline, restore snapshots, document
  findings and cleanup.

---

## Contents

**Scanning** -- external (Nmap), internal (Portscan)

**Initial & Further Access** -- Phishing (SendEmail, Swaks), Microsoft Word macros, JScript,
HTA, staged payloads, shellcode runners, PowerShell stagers, process injection / hollowing

**Payloads** -- C#, C/C++ (EXE/DLL), PowerShell, WebShells (ASPX / JSP / PHP)

**Upgrading Shells** -- process hollowing, native injection (NtMapInjection, NtQueueApc),
shellcode hollower / injector

**Tunneling** -- Ligolo-ng (simple, double, reverse, local port-forward, file transfer),
Metasploit (autoroute, socks), sshuttle

**AV Bypassing** -- AppLocker bypass, CLM bypass, shellcode runner

**Privilege Escalation (Windows)** -- enumeration, token impersonation (SeImpersonate,
PrintSpoofer, SpoolSample, SigmaPotato, FullPowers), AlwaysInstallElevated, service binary
hijacking

**Defense Evasion** -- enumerate defenses, AppLocker/CLM/AMSI/Defender/PPL, UAC bypass,
firewall, RDP pass-the-hash

**Credentials** -- Mimikatz, LSA protection bypass, MiniDump, hash cracking (NTLM /
Net-NTLMv2), pass-the-hash, relay attacks, spraying (Hydra, NetExec)

**Linux** -- initial access, reverse shells, payloads, privesc (SUID, cronjobs, setuid),
lateral movement (SSH keys/persistence/hijacking/agent forwarding, Ansible, Artifactory),
Kerberos (keytab, cached tickets, Impacket)

**MSSQL** -- authentication, enumeration, relay, linked servers, privesc, code execution
(PowerUpSQL, SQLRecon, MSSQLClient, CLR, mssqlpwner)

**AD Enumeration** -- BloodHound / SharpHound, LDAP / ldapsearch, domain enum from Linux and
Windows, ASREPRoast, Kerberoast, Overpass-the-Hash, delegation (unconstrained / constrained /
RBCD)

**AD Attacking** -- unconstrained delegation, DCSync, Golden Tickets, LAPS

**Utilities** -- payload encoders, PowerView, shellcode encryptors, RDP

---

## Standard Reference Commands

The sections below are the general scanning and enumeration commands from the notes. The
detailed payload, injection, and evasion code is kept out of this repo (see the note at the
bottom).

### External Scanning -- Nmap

```bash
# Advanced enumeration -- full system/version detection
nmap -A [IP/domain] -oN [machine_name].txt

# Fast all-ports scan (SYN + UDP)
sudo nmap -p- -sS -sU --min-rate=1000 --max-retries=1 -T4 [IP/domain]

# Fast TCP scan, skip host discovery
nmap -p- -T4 -n -Pn [IP/domain] -oN [machine_name]_ports.txt

# Greppable full-port scan, high rate
sudo nmap --min-rate 5000 -p- -vvv -Pn -n -oG openPorts.txt [IP]

# Top ports only
nmap [IP/domain] --top-ports [N]
```

### Windows -- Local Enumeration

```powershell
# Enumerate everything in a user's folder
Get-ChildItem -Path C:\Users\ -Include *.* -File -Recurse -ErrorAction SilentlyContinue

# Password-manager databases
Get-ChildItem -Path C:\ -Include *.kdbx -File -Recurse -ErrorAction SilentlyContinue

# Backup / temp / old files
Get-ChildItem -Path C:\Users -Include *.bak,*.old,*.tmp -File -Recurse -ErrorAction SilentlyContinue

# Executable / script files
Get-ChildItem -Path C:\Users -Include *.exe,*.bat,*.ps1 -File -Recurse -ErrorAction SilentlyContinue
```

Good spots to check: web roots (`C:\inetpub\wwwroot`), and `Temp` / `Tasks` / `Config`
folders for hardcoded strings in config files.

### Active Directory -- Enumeration

```bash
# Nmap LDAP scripts (no brute)
nmap -n -sV --script "ldap* and not brute" <target_ip>

# Basic LDAP (anonymous)
ldapsearch -x -H ldap://<target_ip>
ldapsearch -x -h <DOMAIN_IP> -s base
ldapsearch -x -H ldap://<target_ip> -s base namingcontexts

# Authenticated -- domain users
ldapsearch -x -h <DOMAIN_IP> -D "<USER>@domain.com" -w <PASS> \
  -b "DC=domain,DC=local" "(objectClass=user)" sAMAccountName

# Authenticated -- computers
ldapsearch -x -h <DOMAIN_IP> -D "<USER>@domain.com" -w <PASS> \
  -b "DC=domain,DC=local" "(objectClass=computer)" name
```

Useful LDAP search terms for review: `(ms-MCS-AdmPwd=*)` (LAPS passwords),
`(description=*password*)`, `(ms-MCS-AdmPwdExpirationTime=*)`.

Graph collection: **SharpHound** / **BloodHound** for mapping group membership, delegation,
and attack paths.

### Linux -- Local Enumeration & Privilege Checks

```bash
# What can I run as sudo?
sudo -l

# Find SUID binaries
find / -perm -4000 -type f 2>/dev/null
ls -l $(find / -perm -4000 -type f 2>/dev/null)   # then cross-check GTFOBins

# Hidden config / secret files worth checking
# .bashrc  .bash_profile  .mysql_history  .ssh  .enc

# Crack /etc/passwd + /etc/shadow (when readable)
unshadow /etc/passwd /etc/shadow > unshadowed.txt
john --rules --wordlist=/usr/share/wordlists/rockyou.txt unshadowed.txt
hashcat -m 1800 hashes.txt /usr/share/wordlists/rockyou.txt
```

---

## Tooling Stack (referenced throughout)

Nmap | PowerView | Metasploit / Meterpreter | Impacket | BloodHound / SharpHound |
ldapsearch | Ligolo-ng | sshuttle | PrintSpoofer | SpoolSample | SigmaPotato | FullPowers |
Mimikatz | Rubeus | PowerUpSQL | SQLRecon | MSSQLClient | mssqlpwner | NetExec | Hydra |
John the Ripper | Hashcat | GTFOBins

---

## Note on the detailed technique code

This README is the **index and standard-reference** layer of my OSEP notes. The full
operational code for the offensive sections -- shellcode runners, process injection and
hollowing, AV/AMSI/CLM/AppLocker bypass, loaders, reverse shells, and credential-dumping
one-liners -- is kept in my offline lab notes rather than in this repository.
