# OSED — EXP-301

Study index for **OffSec Exploit Developer (OSED)**, tied to the **EXP-301: Windows User Mode Exploit Development** course.

OSED is about writing Windows user-mode exploits from the ground up: finding the bug, controlling execution, and getting reliable code execution past modern mitigations.

## Syllabus domains

Public EXP-301 topic areas, used here as a checklist.

| # | Domain | What it covers (high level) |
| --- | --- | --- |
| 1 | WinDbg and x86 refresher | Debugging workflow and assembly fundamentals |
| 2 | Stack buffer overflows | Classic stack corruption and control of execution |
| 3 | Exploiting SEH overflows | Structured exception handler overwrites |
| 4 | Egghunters | Staging shellcode when space is tight |
| 5 | Shellcode from scratch | Writing and debugging custom shellcode |
| 6 | Reverse engineering for bugs | Finding vulnerabilities in closed-source binaries |
| 7 | Bypassing DEP | Return-oriented programming and ROP chains |
| 8 | Bypassing ASLR | Defeating address randomisation |
| 9 | Format string attacks | Information leaks and writes via format bugs |
| 10 | Custom ROP and chaining | Combining primitives into a working exploit |

> Same test as OSEP: can I explain the mitigation and the primitive that beats it, and have I built it myself in the debugger?

## Prep checklist

- [ ] Fluent WinDbg workflow (breakpoints, memory inspection, scripting)
- [ ] Comfortable reading x86 assembly and tracing a crash to root cause
- [ ] Built each exploit class by hand, not from a template
- [ ] Understand each mitigation (DEP, ASLR, SEH protections) and its bypass
- [ ] Practised reverse engineering a closed-source target to locate a bug
- [ ] Rehearsed the reporting workflow and template

## References

_To be added — I'll drop primary sources and useful write-ups here as links, credited, not copied._

- OffSec — [EXP-301 / OSED course page and syllabus](https://www.offsec.com/courses/exp-301/)
- OffSec — OSED exam guide (add the current link)

## Notes

Topic notes will be added as `NN-topic.md` files, each in my own words with the source linked.
