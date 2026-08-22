# distccd Enumeration

## Service Information

- **Service:** distccd
- **Port:** TCP/3632
- **Target:** Metasploitable 2
- **Version identified:** distccd v1 / GCC 4.2.4
- **Vulnerability identified:** CVE-2004-2687
- **Assessment method:** Non-destructive vulnerability detection using Nmap NSE

## 1. Service Discovery

We first identified TCP/3632 using Nmap service detection.

**Command used**

```bash
nmap -p 3632 -sV 192.168.56.20
```

The scan identified:

```text
3632/tcp open distccd distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))
```

This established that the target was running an old version of the `distccd` distributed compilation service.

[View distccd service detection evidence](../evidence/10_DISTCCD01_nmap_distccd_detection.png)

## 2. Initial Exploit-Database Search

We initially attempted to determine whether a known public exploit was available through the local Exploit-DB database.

**Command used**

```bash
searchsploit distccd
```

The local database returned:

```text
Exploits: No Results
Shellcodes: No Results
```

This result was not interpreted as evidence that the service was secure.

A negative SearchSploit result only indicates that the local Exploit-DB database did not return a matching result at the time of the assessment.

The search result was therefore treated as an unsuccessful local lookup rather than a vulnerability assessment.

The SearchSploit screenshot was not included in the public portfolio because it does not provide direct evidence about the security state of the target.

## 3. Documented Vulnerability Research

During the assessment, we identified the Nmap NSE script documentation for:

**CVE-2004-2687 — distcc Daemon Command Execution**

The documented vulnerability affects vulnerable configurations of the distributed compiler daemon `distccd`.

[View documented vulnerability reference](../evidence/10_DISTCCD03_documented_vulnerability.png)

The documentation describes the vulnerability as allowing command execution through the affected service.

This provided a basis for performing a targeted, non-destructive vulnerability check against the laboratory target.

## 4. Non-Destructive Vulnerability Detection

Rather than attempting to exploit the vulnerability or obtain a shell through `distccd`, we used the dedicated Nmap NSE vulnerability-detection script.

**Command used**

```bash
nmap -p 3632 --script distcc-cve2004-2687 192.168.56.20
```

Nmap reported:

```text
VULNERABLE:
distcc Daemon Command Execution

State: VULNERABLE (Exploitable)
IDs: CVE:CVE-2004-2687
Risk factor: High
CVSSv2: 9.3 (HIGH)
```

The script also reported:

```text
uid=1(daemon) gid=1(daemon) groups=1(daemon)
```

This indicates that the vulnerable service was operating under the `daemon` account rather than directly as root.

[View Nmap CVE detection evidence](../evidence/10_DISTCCD04_distccd_cve_2004_2687.png)

### Important Assessment Boundary

The vulnerability was **identified through non-destructive detection using Nmap NSE**.

We did **not** execute the vulnerability, send arbitrary commands through the vulnerable service, obtain a remote shell, or use an exploitation framework.

The `VULNERABLE (Exploitable)` classification shown in the screenshot is Nmap's assessment of the detected condition; it does not mean that exploitation was performed during this assessment.

## 5. Vulnerability Assessment

The assessment established three important facts.

### Service Exposure

TCP/3632 was accessible from the assessment host.

### Vulnerable Implementation

The target was running an old `distccd` implementation identified as:

```text
distccd v1
(GNU) 4.2.4
```

### Known Vulnerability

Nmap's dedicated NSE script identified:

```text
CVE-2004-2687
```

and classified the service as:

```text
VULNERABLE (Exploitable)
```

with a reported CVSSv2 score of:

```text
9.3 (HIGH)
```

The detection also identified the service execution context as:

```text
uid=1(daemon)
gid=1(daemon)
groups=1(daemon)
```

This is important because the assessment confirms the vulnerability without overstating its privileges.

## Security Analysis

The assessment demonstrated the following chain:

```text
TCP/3632 exposed
        ↓
distccd identified
        ↓
Old distccd implementation detected
        ↓
CVE-2004-2687 identified
        ↓
Nmap NSE non-destructive detection
        ↓
VULNERABLE (Exploitable)
        ↓
Execution context: daemon
```

The vulnerability therefore represents a significant remote command-execution risk, but the assessment did not proceed to command execution.

## Evidence Summary

The public portfolio uses three relevant screenshots:

```text
10_DISTCCD01_nmap_distccd_detection.png
10_DISTCCD03_documented_vulnerability.png
10_DISTCCD04_distccd_cve_2004_2687.png
```

### Evidence 01 — Service Identification

Nmap identifies TCP/3632 and the running `distccd` version.

[View service detection evidence](../evidence/10_DISTCCD01_nmap_distccd_detection.png)

### Evidence 02 — Vulnerability Documentation

The Nmap documentation provides the technical reference for CVE-2004-2687.

[View documented vulnerability reference](../evidence/10_DISTCCD03_documented_vulnerability.png)

### Evidence 03 — Vulnerability Detection

Nmap NSE performs a non-destructive vulnerability check and identifies the target as vulnerable to CVE-2004-2687.

[View vulnerability detection evidence](../evidence/10_DISTCCD04_distccd_cve_2004_2687.png)

The evidence chain is:

**Service identification → documented vulnerability research → non-destructive vulnerability detection**

## Scope and Limitations

The assessment was conducted against the intentionally vulnerable Metasploitable 2 laboratory environment.

Testing was limited to:

- Service identification.
- Version detection.
- Vulnerability research.
- Non-destructive Nmap NSE vulnerability detection.

The CVE was **not exploited**.

No arbitrary command was executed through `distccd`.

No remote shell was obtained through the vulnerability.

No exploitation framework was used.

The reported `VULNERABLE (Exploitable)` status originates from Nmap's vulnerability-detection script and should not be interpreted as evidence that exploitation was performed.

## Commands

### Service detection

```bash
nmap -p 3632 -sV 192.168.56.20
```

### Search local Exploit-DB database

```bash
searchsploit distccd
```

### Vulnerability detection

```bash
nmap -p 3632 --script distcc-cve2004-2687 192.168.56.20
```

## Portfolio Conclusion

**"TCP/3632 exposed an outdated `distccd` service. Service identification confirmed an old distccd implementation, while documented vulnerability research identified CVE-2004-2687 as a relevant security issue. A dedicated Nmap NSE vulnerability check subsequently classified the service as vulnerable to remote command execution. The vulnerability was identified through non-destructive detection only; no exploitation, arbitrary command execution or remote shell was performed."**
