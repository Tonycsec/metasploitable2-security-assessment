# SMTP Security Findings

## Overview

The SMTP service exposed by the Metasploitable 2 host presented several security considerations identified during the service assessment.

The assessment identified three primary findings:

1. **SMTP Service Exposure**
2. **SMTP Information Disclosure**
3. **Potentially Unnecessary SMTP Functionality**

The findings were identified through service detection, SMTP command enumeration and manual protocol interaction.

No exploitation, unauthorized mail delivery or open-relay abuse was performed as part of this assessment.

---

## Finding 01 — SMTP Service Exposure

### Description

The target exposed an SMTP service on TCP/25.

Nmap identified the service as Postfix SMTP, confirming that the host was providing a network-accessible mail service.

### Evidence

The SMTP service was identified through Nmap:

[View Nmap SMTP service detection evidence](../evidence/04_SMTP01_nmap_smtp_detection.png)

The detected service was:

```text
25/tcp open smtp Postfix smtpd
```

### Security Impact

An exposed mail service increases the host's attack surface and introduces additional functionality that must be securely configured and maintained.

SMTP exposure is not inherently a vulnerability. The security impact depends on the intended role of the system, the networks from which the service is accessible and the server's mail-relay configuration.

### Severity

**Medium**

The severity reflects the exposure of a network-facing mail service whose configuration requires appropriate access controls and hardening.

### Recommendation

- Restrict SMTP access to networks and systems that require it.
- Expose SMTP only when operationally necessary.
- Apply firewall and network segmentation controls.
- Keep Postfix fully patched and supported.
- Regularly review exposed network services.
- Ensure that mail-relay restrictions are correctly configured.

---

## Finding 02 — SMTP Information Disclosure

### Description

The SMTP service disclosed implementation and host information through its banner and protocol responses.

The server identified itself as:

```text
220 metasploitable.localdomain ESMTP Postfix (Ubuntu)
```

The SMTP `EHLO` response also disclosed the configured hostname and the functionality supported by the service.

### Evidence

The SMTP banner and `EHLO` response were obtained through a direct connection:

[View SMTP banner and EHLO evidence](../evidence/04_SMTP03_smtp_banner_and_ehlo.png)

The server disclosed:

```text
metasploitable.localdomain
Postfix (Ubuntu)
PIPELINING
SIZE 10240000
VRFY
ETRN
STARTTLS
ENHANCEDSTATUSCODES
8BITMIME
DSN
```

The same functionality was also identified through Nmap SMTP command enumeration:

[View SMTP command enumeration evidence](../evidence/04_SMTP02_smtp_commands.png)

### Security Impact

SMTP banners and protocol responses can provide useful information during reconnaissance.

Information such as the mail-server implementation, hostname and supported functionality may assist an attacker in identifying the target's technology stack and determining which areas warrant further investigation.

The disclosed information does not by itself constitute a direct compromise.

### Severity

**Low**

The finding primarily represents reconnaissance-related information disclosure rather than a demonstrated vulnerability.

### Recommendation

- Minimize unnecessary information in SMTP banners where practical.
- Avoid exposing unnecessary implementation details.
- Review the hostname and server information returned by SMTP responses.
- Keep the underlying mail software fully patched.
- Consider whether each advertised SMTP extension is operationally required.

---

## Finding 03 — Potentially Unnecessary SMTP Functionality

### Description

The SMTP service advertised several protocol extensions and commands, including:

- `VRFY`
- `ETRN`
- `STARTTLS`
- `PIPELINING`
- `SIZE`
- `ENHANCEDSTATUSCODES`
- `8BITMIME`
- `DSN`

The `VRFY` and `ETRN` commands are particularly relevant from a security-hardening perspective and should be reviewed to determine whether they are required by the intended mail-server configuration.

### Evidence

Nmap identified the supported SMTP functionality:

[View SMTP command enumeration evidence](../evidence/04_SMTP02_smtp_commands.png)

The same capabilities were confirmed during manual SMTP interaction:

[View SMTP banner and EHLO evidence](../evidence/04_SMTP03_smtp_banner_and_ehlo.png)

### Security Impact

Unnecessary protocol functionality can increase the attack surface of a network-facing service.

For example, SMTP user-verification functionality such as `VRFY` may potentially contribute to account enumeration depending on the server configuration.

Similarly, `ETRN` should be restricted according to the intended mail-server architecture and operational requirements.

However, the presence of these commands alone does **not** demonstrate a confirmed vulnerability.

No user-enumeration or `ETRN` abuse testing was performed during this assessment.

### Severity

**Medium**

The finding represents a hardening concern requiring configuration review rather than a confirmed exploitable vulnerability.

### Recommendation

- Disable or restrict `VRFY` if it is not operationally required.
- Review and restrict `ETRN` according to the intended mail-server configuration.
- Disable unnecessary SMTP extensions where appropriate.
- Apply a restrictive SMTP configuration based on the system's actual mail-delivery requirements.
- Review Postfix configuration regularly.
- Ensure that SMTP relay permissions are explicitly defined.

---

## Open Relay Assessment

An open relay would represent a significant SMTP security vulnerability because it could allow unauthorized users to relay email through the server.

However, **open-relay testing was not performed during this assessment**.

Therefore, the assessment does not classify the target as an open relay.

The appropriate recommendation is to verify that Postfix permits mail relay only for authorized destinations, users or networks.

### Recommendation

- Configure Postfix to relay only for authorized users or networks.
- Prevent unrestricted external relay.
- Regularly test relay restrictions as part of mail-server security assessments.
- Monitor mail activity for unauthorized relay attempts.

---

## Overall Assessment

The SMTP service was successfully identified and enumerated on TCP/25.

The assessment confirmed that:

- Postfix SMTP was exposed on TCP/25.
- The SMTP banner disclosed the hostname and implementation.
- The server advertised multiple SMTP commands and extensions.
- `VRFY` and `ETRN` were exposed and should be reviewed as part of service hardening.
- `STARTTLS` was advertised by the server.
- No open-relay testing was performed.
- No exploitation or unauthorized mail delivery was attempted.

The primary security concerns are therefore **network service exposure, information disclosure and potentially unnecessary SMTP functionality**.

Recommended remediation includes restricting SMTP exposure, minimizing unnecessary information disclosure, reviewing advertised functionality and ensuring that mail-relay permissions are explicitly restricted.

No exploitation was performed during this assessment.
