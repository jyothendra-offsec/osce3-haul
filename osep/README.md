# OSEP — PEN-300

Study index for **OffSec Experienced Penetration Tester (OSEP)**, the certification tied to the **PEN-300: Evasion Techniques and Breaching Defenses** course.

OSEP is about getting a foothold and moving through a hardened environment when the easy paths are closed: defences are on, antivirus is watching, application control is enforced, and the network is segmented. The exam is a 48-hour hands-on assessment followed by a report.

## Syllabus domains

These are the public PEN-300 topic areas. I use them as a checklist and link out to my own notes as I write them.

| # | Domain | What it covers (high level) |
| --- | --- | --- |
| 1 | Operating system and programming theory | The background needed for the rest: Windows internals, PE basics, .NET, PowerShell, C# tooling |
| 2 | Client-side code execution | Phishing payloads delivered through Office documents and similar vectors |
| 3 | Process injection and migration | Moving code execution between processes |
| 4 | Antivirus evasion | Understanding how AV detects, and why signatured tooling gets caught |
| 5 | Application whitelisting | How allow-listing works and where it's commonly misconfigured |
| 6 | Bypassing network filters | Egress and proxy-aware communication |
| 7 | Linux post-exploitation | Enumeration and escalation on Linux hosts |
| 8 | Kiosk breakouts | Escaping restricted desktop/kiosk environments |
| 9 | Windows credentials | Credential theft, caching, and reuse concepts |
| 10 | Windows lateral movement | Moving between hosts with recovered access |
| 11 | Linux lateral movement | The same, in Linux-heavy estates |
| 12 | Microsoft SQL attacks | Abusing MSSQL trust and links |
| 13 | Active Directory exploitation | Kerberos, delegation, ACL abuse, forest/domain trust paths |
| 14 | Combining the pieces | Chaining the above into a full assumed-breach path |

> Treat each row as "can I explain the concept, and have I done it in the lab?" — not "do I have a copy-paste command." The exam rewards understanding why a technique works so you can adapt when the obvious version is blocked.

## Prep checklist

- [ ] Build a comfortable C# / PowerShell tooling workflow before relying on public tools
- [ ] Understand the detection mechanism behind each evasion topic, not just the bypass
- [ ] Work every course exercise and the extra-mile challenges
- [ ] Complete all of the Challenge Labs end to end, assumed-breach style
- [ ] Practice writing as you go — screenshots, commands, and the *why* for each step
- [ ] Rehearse a clean reporting workflow and template before exam day
- [ ] Time-box a mock 48-hour run against the challenge labs

## Lab and reporting notes

- Keep lab notes per host: context, what was tried, what worked, and the artefact that proves it.
- Separate "a technique ran" from "it gave access" — record the evidence for each.
- Keep real credentials and any sensitive capture out of this repo; commit only redacted notes.
- Confirm the current exam and tooling rules on the official exam guide before your session; OffSec updates them.

## References

Primary sources first, then community notes I found useful. These are links and credit — read them at the source.

- OffSec — [PEN-300 / OSEP course page and syllabus](https://www.offsec.com/courses/pen-300/)
- OffSec — [OSEP exam guide](https://help.offsec.com/hc/en-us/articles/360050293792-OSEP-Exam-Guide) (check for the current version)
- Emmanuel Solis — [OSEP notes](https://www.emmanuelsolis.com/osep.html)

_Add new references here as links with a line on why they helped, rather than pasting their content._

## Notes

Topic notes will be added here as `NN-topic.md` files (e.g. `13-active-directory.md`), each in my own words with the source linked.
