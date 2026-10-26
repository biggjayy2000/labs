# Active Directory reconnaissance — `corp.local`

**Internal network discovery and Kerberos enumeration against a lab domain controller**

| | |
|---|---|
| Environment | VirtualBox host-only network, `192.168.56.0/24` |
| Attacker workstation | Kali Linux |
| Target | `DC01` — Windows Server domain controller, `192.168.56.10` |
| Domain | `corp.local` (NetBIOS `CORP`) |
| Test date | 10 October 2026 |
| Status | Phase 1 complete — later phases planned |

## Objective

Work out what an attacker sitting on the same network segment can learn about a domain controller
*before* touching a password. No exploitation, no password guessing, no persistence — this phase is
discovery only, and everything is written up from the defender's side as well as the attacker's.

The question I actually wanted answered was: **what does a domain controller give away to somebody
who has never authenticated?**

## Scope and authorisation

Two machines, both mine: a Kali VM and `DC01`, joined on an isolated VirtualBox host-only network so
nothing is reachable from the physical LAN. The Kerberos test used three accounts created for the
lab — `ajohnson`, `mlee`, `tsmith` — and never tested a password. Explicitly out of scope:
third-party systems, password spraying, destructive testing, persistence, lateral movement.

## What I ran

Seven phases, each one chosen because the previous result pointed at it.

| # | Phase | Command (abbreviated) | Result |
|---|---|---|---|
| 1 | Service discovery | `nmap -Pn -sV --top-ports 100 192.168.56.10` | 6 services exposed: DNS 53, Kerberos 88, MSRPC 135, NetBIOS 139, LDAP 389, SMB 445 |
| 2 | DNS enumeration | `dig @… corp.local SOA` and `_ldap._tcp.dc._msdcs… SRV`, `_kerberos._tcp… SRV` | Hostname `dc01.corp.local`, `hostmaster.corp.local`, LDAP on 389, Kerberos on 88 |
| 3 | LDAP Root DSE | `ldapsearch -x -H ldap://… -s base -b "" "(objectClass=*)" defaultNamingContext dnsHostName supportedLDAPVersion` | Anonymous query returned `defaultNamingContext: DC=corp,DC=local` |
| 4 | Anonymous SMB | `smbclient -L //192.168.56.10 -N` | Anonymous session accepted; **no shares listed** |
| 5 | SMB dialects | `nmap -p445 --script smb-protocols,smb-security-mode` | SMB 2.0.2, 2.1, 3.0, 3.0.2, 3.1.1 |
| 6 | SMB signing | `nmap -p445 --script smb2-security-mode` | `Message signing enabled and required` (3.1.1) |
| 7 | Kerberos users | `nmap -p88 --script krb5-enum-users` with a 3-name list | All three lab principals discovered, no passwords tested |

94 of the 100 scanned ports came back filtered. `filtered` means Nmap got no response — it does not
mean the port is closed, and I did not write it up as if it did.

## What the results actually mean

This is the part I spent the most time on, because it is the part that separates a report from a
scan dump. Each finding is classified as *service exposure*, *information disclosure*, *positive
control observed*, or *confirmed vulnerability* — and on this run, nothing reached the last category.

**Service exposure (expected, not a finding).** DNS, Kerberos, LDAP, RPC, NetBIOS and SMB are what a
domain controller is. Their presence tells an assessor where to look next; it is not a weakness by
itself.

**Information disclosure (real, low severity).**

- DNS answered authoritatively for the domain, publishing the DC hostname and its LDAP and Kerberos
  service records. This is normal AD service discovery, and it is also exactly what an attacker uses
  to find the controller without scanning for it.
- An unauthenticated LDAP Root DSE query returned the directory naming context and server name. Root
  DSE is *designed* to expose server capabilities, so this is an observation to document, not a
  vulnerability to escalate.
- An anonymous SMB session was accepted. No share names came back, so no file access was
  demonstrated — an accepted session is not the same as access, and I did not report it as such.

**Positive control (worth stating).** SMB message signing is enabled and required for SMB 3.1.1,
which makes SMB relay attacks materially harder. A finding that says "this is configured correctly"
is as useful to a client as one that says "this is broken".

**Contains a usable signal.** Kerberos responses let me confirm all three lab usernames without
testing a single password. That is username enumeration: it is not a compromise, but confirmed
usernames are the input to every later authentication attack, which is why it belongs in the report.

## Two commands that failed first

Both are in the writeup because the recovery is the interesting part.

The first `ldapsearch` returned **result code 1 (operations error)** with a message saying a
successful bind was required. The arguments had also been mangled by shell quoting. A result like
that is not evidence that anonymous LDAP is blocked in general — it is evidence that my command was
wrong. I fixed the syntax, re-ran it, and got result code 0 with the Root DSE fields.

The first `krb5-enum-users` attempt used backslashes that swallowed the target, so Nmap reported "no
targets specified" and scanned nothing. The corrected one-line invocation produced three principals.
Treating a broken command as a finding would have been the easiest mistake in the whole lab.

## Limitations

- One target, and a limited port range — `--top-ports 100` is not a full sweep.
- Anonymous SMB access to files was never demonstrated; the accepted session is the extent of it.
- SMB signing was checked; other SMB hardening was not.
- Three known usernames only, so the test says nothing about how easily other accounts could be
  enumerated in a larger directory.
- No password strength, lockout, escalation, lateral movement or domain compromise testing.
- Defensive log correlation was not completed in this phase.

## Defensive recommendations

1. Review domain controller auditing for unusual patterns of Kerberos requests, which is the
   telemetry this enumeration generates.
2. Confirm anonymous SMB sessions cannot reach sensitive shares.
3. Keep SMB signing in the security baseline — it was correct here, and it is worth keeping.
4. Review LDAP access controls so unauthenticated clients get only intended metadata.
5. Restrict network reachability of DC services to the segments that need them; the whole exercise
   started with plain network adjacency.
6. Correlate offensive test timestamps against Windows security events to prove monitoring coverage.

## The rest of the engagement

This page covers the anonymous phase. The full lab — authenticated LDAP, group and delegation
enumeration, SYSVOL and group policy extraction, and Kerberos SPN testing — is in
**[ad-home-lab](https://github.com/biggjayy2000/ad-home-lab)**, with all 29 screenshots and the two
Kerberos failures documented rather than hidden.

## Skills this demonstrates

Active Directory service discovery, DNS/SRV enumeration, LDAP Root DSE queries, SMB enumeration and
signing assessment, Kerberos username enumeration, and the discipline of separating an observation
from a confirmed vulnerability.

---

*Original document with the raw terminal output: [`Active Directory Red-Team Lab`](https://docs.google.com/document/d/1WFX0furhPBig9N7AEazqL2_bbrtz8A-XqqKO8z5lnHY/edit)
in the Drive archive. Screenshots are in the [full lab repository](https://github.com/biggjayy2000/ad-home-lab).*
