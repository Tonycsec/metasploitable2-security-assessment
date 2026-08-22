# distccd Security Findings

## Overview

The assessment identified a **high-severity remote command execution vulnerability** affecting the `distccd` service exposed on TCP/3632.

The service was identified as an outdated `distccd` implementation, and a dedicated Nmap NSE vulnerability-detection script confirmed the presence of:

**CVE-2004-2687 — distcc Daemon Command Execution**

The vulnerability was identified through **non-destructive vulnerability detection**.

No arbitrary command was executed through the vulnerable service, no remote shell was obtained, and no exploitation framework was used.

---

## Finding 01 — CVE-2004-2687 Remote Command Execution

### Description

TCP/3632 was exposed and identified as a `distccd` service.

Service detection reported:

```text
3632/tcp open distccd distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))
```

[View distccd service detection evidence](../evidence/10_DISTCCD01_nmap_distccd_detection.png)

Further vulnerability research identified **CVE-2004-2687**, a known command-execution vulnerability affecting vulnerable `distccd` configurations.

[View documented vulnerability reference](../evidence/10_DISTCCD03_documented_vulnerability.png)

A dedicated Nmap NSE vulnerability check was then performed:

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

[View Nmap vulnerability detection evidence](../evidence/10_DISTCCD04_distccd_cve_2004_2687.png)

### Severity

**High**

### Vulnerability

**CVE-2004-2687**

### CVSS

**CVSSv2: 9.3 (High)**

### Security Impact

The vulnerability can allow an attacker to execute arbitrary commands remotely through a vulnerable `distccd` service.

Because the service was directly accessible over TCP/3632, an attacker with network access to the target could potentially interact with the vulnerable service remotely.

Potential consequences include:

- Unauthorized command execution.
- Access to information available to the service account.
- Modification of files accessible to the service.
- Further compromise through additional vulnerabilities or misconfigurations.
- Potential lateral movement depending on the surrounding environment.

The exact impact depends on the privileges of the process and the permissions available to the resulting execution context.

---

## Execution Context

The Nmap vulnerability-detection script reported:

```text
uid=1(daemon)
gid=1(daemon)
groups=1(daemon)
```

This indicates that the vulnerable service was operating under the `daemon` account.

This distinction is important.

The vulnerability provides a potential **remote command-execution capability**, but the assessment did not establish direct root-level execution through `distccd`.

The service's execution context should therefore not be overstated.

---

## Finding 02 — Network Exposure of Vulnerable Service

### Description

The vulnerable `distccd` service was directly accessible on:

```text
TCP/3632
```

The combination of:

```text
Network exposure
        +
Known vulnerable service
```

increases the practical risk associated with the vulnerability.

### Security Impact

A vulnerability that is reachable remotely presents a substantially larger attack surface than one restricted to a trusted internal interface.

### Severity

**High**

### Recommendation

Restrict TCP/3632 to explicitly authorized compilation clients or, preferably, remove the service if distributed compilation is not required.

Firewall rules should prevent untrusted systems from connecting to the service.

---

## Finding 03 — Outdated Service

### Description

The target was running:

```text
distccd v1
(GNU) 4.2.4
```

This is an obsolete software environment associated with known vulnerabilities.

[View service version evidence](../evidence/10_DISTCCD01_nmap_distccd_detection.png)

### Security Impact

Running outdated network-facing software increases the likelihood of exposure to publicly documented vulnerabilities.

### Severity

**High**

### Recommendation

- Upgrade `distccd` to a supported and patched version where the service is required.
- Remove the service if distributed compilation is unnecessary.
- Review other outdated network-facing services on the host.
- Implement a regular vulnerability and patch-management process.

---

## Remediation

The preferred remediation is to **remove or disable `distccd` if it is not required**.

If distributed compilation is necessary:

1. Upgrade to a supported and patched implementation.
2. Restrict access to trusted compilation clients.
3. Filter TCP/3632 using host and network firewalls.
4. Isolate build infrastructure from untrusted networks.
5. Monitor connections to the service.
6. Regularly assess the service for known vulnerabilities.

After remediation, verify that:

```text
TCP/3632
```

is no longer unnecessarily exposed and that the vulnerable version is no longer running.

---

## Evidence Chain

The evidence establishes the finding in three stages.

### 1. Service Identification

Nmap identified the exposed `distccd` service and its version.

[View service detection evidence](../evidence/10_DISTCCD01_nmap_distccd_detection.png)

### 2. Vulnerability Research

The documented Nmap NSE vulnerability reference identifies CVE-2004-2687 as a relevant `distccd` command-execution vulnerability.

[View documented vulnerability reference](../evidence/10_DISTCCD03_documented_vulnerability.png)

### 3. Vulnerability Detection

Nmap NSE performed a non-destructive vulnerability check and classified the target as:

```text
VULNERABLE (Exploitable)
```

[View vulnerability detection evidence](../evidence/10_DISTCCD04_distccd_cve_2004_2687.png)

The complete evidence chain is:

**Service identification → vulnerability research → non-destructive vulnerability detection**

---

## Assessment Boundary

The CVE was **not exploited** during this assessment.

The Nmap NSE script was used solely for vulnerability detection.

No arbitrary command was manually supplied to the vulnerable service.

No remote shell was obtained through CVE-2004-2687.

No exploitation framework was used.

Therefore, the assessment demonstrates that the target is **vulnerable to the documented condition**, but does not claim successful exploitation.

This distinction is important when presenting the finding professionally.

---

## Scope and Limitations

The assessment was conducted against the intentionally vulnerable Metasploitable 2 laboratory environment.

The testing performed was limited to:

- Service identification.
- Version detection.
- Vulnerability research.
- Non-destructive vulnerability detection.

The reported `VULNERABLE (Exploitable)` status originates from the Nmap NSE detection script.

It should not be interpreted as evidence that exploitation was performed.

---

## Commands

### Service detection

```bash
nmap -p 3632 -sV 192.168.56.20
```

### Vulnerability detection

```bash
nmap -p 3632 --script distcc-cve2004-2687 192.168.56.20
```

---

## Portfolio Conclusion

**"TCP/3632 exposed an outdated `distccd` service. Service identification confirmed an old distccd implementation, while documented vulnerability research identified CVE-2004-2687 as a relevant remote command-execution vulnerability. A dedicated Nmap NSE vulnerability check subsequently classified the service as vulnerable. The vulnerability was identified through non-destructive detection only; no exploitation, arbitrary command execution or remote shell was performed."**
