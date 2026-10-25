# Security lab writeups

Six hands-on security exercises from my home lab, written up the way I would hand them to a team
lead: what the situation was, what I ran, what the evidence showed, and what I would change.

They are not notes. Each one separates **observation** from **confirmed finding**, states its
limitations, and ends with something actionable — because the two failure modes of a junior security
writeup are claiming more than the evidence supports and claiming nothing at all.

## The labs

| # | Lab | Focus | Tools |
|---|---|---|---|
| 1 | [Active Directory reconnaissance](01-active-directory-reconnaissance.md) | Enumeration of a lab domain controller before any password is touched | Kali, Nmap, dig, ldapsearch, smbclient |
| 2 | [Botium Toys — internal security audit](02-botium-toys-audit.md) | Risk-rating a whole security programme and sequencing remediation | Risk analysis, PCI DSS, GDPR |
| 3 | [NIST CSF analysis of a DoS incident](03-nist-csf-dos-analysis.md) | Two-hour ICMP flood traced through all five CSF functions | NIST CSF |
| 4 | [SQL filtering for suspicious logins](04-sql-login-investigation.md) | Narrowing authentication logs and joining them to employee records | SQL |
| 5 | [File permissions in Linux](05-linux-file-permissions.md) | Reading permission strings and enforcing least privilege | `ls`, `chmod` |
| 6 | [Password hash auditing](06-password-hash-auditing.md) | Dictionary attack against a self-created SHA-256 hash | Hashcat, Kali |

## What the collection shows

**Both sides of the same network.** Lab 1 is offensive: what can be learned about a domain controller
from an unauthenticated position. Labs 3 and 5 are the defensive reading of the same kind of gap —
an exposure nobody monitored, and a permission nobody tightened.

**Evidence discipline.** In the Active Directory lab, six services came back exposed, an anonymous
LDAP query returned directory metadata, and Kerberos confirmed three usernames — and none of it was
written up as a vulnerability, because none of it was one. That distinction is stated explicitly in
the report rather than left for the reader to infer.

**Failed commands are in the writeup.** Two commands in lab 1 and one in lab 5 returned errors before
they worked. Each time the useful skill was noticing that the command was wrong, not that the system
was secure.

**Recommendations that can be implemented.** Every lab ends with remediation: rate limiting instead of
blocking ICMP, a KDF instead of a fast hash, a backup that is tested rather than a backup that exists.

## Scope and ethics

Every exercise ran in an isolated VirtualBox environment against systems and data I created for the
purpose. Where a scenario comes from a course — labs 2, 3 and 4 use fictional or provided data sets —
that is stated at the top of the writeup. Nothing here was tested against a system I do not own.

## Related work

- **[websec-audit](https://github.com/biggjayy2000/websec-audit)** — a Python web security scanner
  with an authorization guard and severity-ranked reporting.
- **[logtriage](https://github.com/biggjayy2000/logtriage)** — detection over SSH and web access
  logs, with MITRE ATT&CK mappings and incident reports.
- **[Portfolio page](https://biggjayy2000.github.io/portfolio/)** — the two tools and these labs in
  one page.

## Structure

```
01-… 06-…      one Markdown file per lab
images/        terminal evidence extracted from the original documents
```

Screenshots are real terminal output from the runs described. Larger archives with additional
screenshots live in my [Google Drive portfolio](https://drive.google.com/drive/folders/1srtrsu9i6lvEGg4dojatligUGTaoswh3?usp=sharing);
each writeup links to its original document.

## Licence

MIT — see [LICENSE](LICENSE).
