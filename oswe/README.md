# OSWE — WEB-300

Study index for the **OffSec Web Expert (OSWE)**, tied to the **WEB-300: Advanced Web Attacks and Exploitation** course.

WEB-300 is white-box web exploitation: you get the source, and you chain logic and injection flaws into authentication bypasses and remote code execution.

## Syllabus snapshot

Public high-level topic areas, used here as a checklist.

| # | Topic | What it covers (high level) |
|---|-------|-----------------------------------------------------|
| 1 | Source code review methodology | Reading an unfamiliar codebase to find bugs efficiently |
| 2 | Authentication bypasses | Logic flaws that defeat login and session handling |
| 3 | SQL injection (blind & stacked) | From injection to data and, where possible, code execution |
| 4 | Server-side template injection | Language-specific template engine exploits |
| 5 | Deserialization | Unsafe object deserialization across languages |
| 6 | Type juggling / loose comparisons | Language-level quirks that break security checks |
| 7 | Cross-site scripting to impact | Chaining XSS into account takeover or RCE |
| 8 | Remote code execution chains | Combining primitives into full compromise |
| 9 | Working across languages | Patterns across PHP, Java, .NET, Node, and others |

> The through-line is methodology: finding the sink, tracing the source, and proving a working chain — NOT memorising payloads.

## Prep checklist

- [ ] A repeatable source-review process for an unfamiliar app
- [ ] Comfortable debugging the target app locally to confirm a theory
- [ ] Audit auth-systems & session logic by hand in source code
- [ ] Can read snippets of several languages to follow data flow
- [ ] Cross-site scripting to impact (not just alert box)
- [ ] Practised scripting a single-command proof-of-concept exploit

## References

- [OffSec WEB-300](https://www.offensive-security.com/courses/web-300/)
