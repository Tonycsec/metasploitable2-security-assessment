# Metasploitable 2 — Final Security Assessment

## Executive Summary

A security assessment was conducted against an intentionally vulnerable **Metasploitable 2** laboratory environment from a dedicated Kali Linux assessment host.

The assessment focused on:

- Network reconnaissance.
- Service discovery.
- Service and version enumeration.
- Authentication testing.
- Access-control validation.
- Security configuration analysis.
- Controlled validation of identified weaknesses.
- Evidence collection.
- Risk assessment and remediation recommendations.

The assessment covered multiple exposed network services, including remote administration protocols, file-sharing services, database servers, RPC services and a Tomcat web management interface.

Multiple significant security weaknesses were identified.

The most severe findings resulted from combinations of:

- Weak or default authentication.
- Excessive privileges.
- Unrestricted network exposure.
- Anonymous access.
- Insecure service configuration.
- Legacy and obsolete software.
- Excessive administrative functionality.

No destructive activity was performed.

Where access or write permissions required validation, testing was deliberately limited to controlled and reversible actions.

---

# Assessment Scope

## Target

**Metasploitable 2**

## Assessment Host

**Kali Linux**

## Laboratory Environment

The assessment was performed inside an isolated virtualized laboratory environment.

The target address used throughout the assessment was:

```text
192.168.56.20
```

The environment was designed specifically for security testing and contained intentionally vulnerable services.

## Assessment Type

The assessment was primarily focused on:

**Reconnaissance → Enumeration → Validation → Security Analysis → Findings**

Exploitation was deliberately limited.

No persistence, destructive modification or post-compromise activity was performed.

---

# Assessment Methodology

The assessment followed a structured methodology.

```text
Reconnaissance
      ↓
Port and Service Discovery
      ↓
Service Enumeration
      ↓
Version Identification
      ↓
Authentication Testing
      ↓
Access-Control Validation
      ↓
Security Configuration Analysis
      ↓
Evidence Collection
      ↓
Risk Assessment
      ↓
Remediation Recommendations
```

The objective was not simply to identify open ports, but to determine what security-relevant functionality was actually exposed and to validate the resulting impact wherever possible.

---

# Attack Surface

The reconnaissance and service assessments identified a broad network attack surface.

| Service | Port | Protocol | Assessment |
|---|---:|---|---|
| SSH | 22 | TCP | Assessed |
| Telnet | 23 | TCP | Assessed |
| SMTP | 25 | TCP | Assessed |
| RPC / Portmapper | 111 | TCP/UDP | Assessed |
| NFS | 2049 | TCP/UDP | Assessed |
| Samba / SMB | 445 | TCP | Assessed |
| MySQL | 3306 | TCP | Assessed |
| PostgreSQL | 5432 | TCP | Assessed |
| Tomcat | 8180 | TCP | Assessed |

Additional services were identified during the reconnaissance phase and are documented within the corresponding service directories.

---

# Key Security Findings

The assessment identified several significant security weaknesses.

## NFS

The NFS server exported the entire root filesystem:

```text
/
```

to:

```text
*
```

The export was successfully mounted remotely.

The assessment subsequently demonstrated access to:

```text
/etc/passwd
/etc/shadow
```

and confirmed remote file creation under:

```text
/tmp
```

This demonstrated both confidentiality and integrity impact.

The password hashes obtained from `/etc/shadow` were redacted from the public evidence.

**Primary risk:** unrestricted remote access to the target root filesystem.

---

## Samba / SMB

The Samba service permitted anonymous SMB enumeration.

Anonymous access to the `tmp` share was successfully established.

The assessment confirmed:

- Anonymous connection.
- Anonymous read access.
- Anonymous write access.

Write permissions were validated by creating and subsequently removing a temporary directory.

The server also exposed SMB1.

The `opt` share was tested separately and correctly rejected anonymous access.

**Primary risk:** unauthenticated users could modify an SMB-accessible resource.

---

## MySQL

The MySQL service exposed on TCP/3306 permitted remote authentication as:

```text
root
```

without requesting a password.

The authenticated account was confirmed as:

```text
root@%
```

and possessed:

```text
ALL PRIVILEGES ON *.*
```

with:

```text
WITH GRANT OPTION
```

Multiple databases were enumerated, including the DVWA database.

Access to the DVWA database was subsequently confirmed.

**Primary risk:** passwordless remote administrative database access combined with unrestricted privileges.

---

## PostgreSQL

PostgreSQL was exposed remotely on TCP/5432.

The `postgres` account successfully authenticated remotely using password authentication.

The authenticated identity was confirmed as:

```text
postgres
```

The account was explicitly verified as a PostgreSQL superuser:

```text
usesuper = true
```

The server was also identified as:

```text
PostgreSQL 8.3.1
```

**Primary risk:** remote authentication using a highly privileged PostgreSQL superuser account.

The assessment did not classify PostgreSQL as unauthenticated or passwordless.

---

## Tomcat

TCP/8180 exposed:

```text
Apache Tomcat/5.5
```

The Tomcat Manager application was remotely accessible and protected by HTTP Basic Authentication.

The credentials:

```text
tomcat:tomcat
```

successfully authenticated to the Manager interface.

The authenticated interface exposed application deployment and WAR upload functionality.

The upload endpoint was tested using a harmless text file.

No malicious WAR was deployed and no code execution was attempted.

**Primary risk:** weak/default administrative credentials combined with remotely accessible application deployment functionality.

---

# RPC Assessment

The RPC portmapper was exposed on TCP/111 and UDP/111.

RPC enumeration using:

```bash
rpcinfo -p 192.168.56.20
```

identified:

```text
100000  portmapper
100024  status
100003  nfs
100021  nlockmgr
100005  mountd
```

NFS versions 2, 3 and 4 were identified.

The RPC assessment itself did not demonstrate a direct compromise.

Its primary value was reconnaissance: RPC enumeration revealed the NFS service and directly guided the subsequent dedicated NFS assessment.

---

# Remote Administration Services

The assessment also investigated remote administration services including SSH and Telnet.

These assessments demonstrated the importance of distinguishing between:

- Service exposure.
- Authentication requirements.
- Legacy protocol support.
- Actual confirmed access.

The SSH assessment also encountered compatibility issues with legacy cryptographic algorithms, while authenticated access was subsequently validated.

Telnet was identified and authenticated access was documented.

These findings contribute to the overall attack surface because remote administration services provide direct network-accessible entry points into the target environment.

---

# SMTP Assessment

SMTP was identified and enumerated.

The assessment included:

- SMTP service detection.
- SMTP command enumeration.
- SMTP banner inspection.
- EHLO interaction.

The purpose was to determine what information the service disclosed and what SMTP functionality was exposed.

No mail-delivery abuse or exploitation was performed.

---

# HTTP / Web Services

The HTTP attack surface was also enumerated during reconnaissance.

The assessment approach focused on identifying exposed web services and their associated applications before moving to more specific service-level investigation.

Tomcat on TCP/8180 represented the most significant web-management finding because the exposed application included an administrative interface.

---

# Risk Analysis

The most significant risks identified during the assessment were caused by combinations of weaknesses rather than isolated configuration issues.

The principal risk chains were:

### NFS

```text
NFS exposed
      ↓
Root filesystem exported to *
      ↓
Remote mount succeeds
      ↓
Filesystem accessible
      ↓
Sensitive system files readable
      ↓
Remote write access confirmed
```

### Samba

```text
SMB exposed
      ↓
Anonymous enumeration
      ↓
Anonymous tmp access
      ↓
Read access
      ↓
Write access
```

### MySQL

```text
TCP/3306 exposed
      ↓
Passwordless remote root authentication
      ↓
root@%
      ↓
ALL PRIVILEGES ON *.*
      ↓
WITH GRANT OPTION
      ↓
Multiple databases accessible
```

### PostgreSQL

```text
TCP/5432 exposed
      ↓
Remote authentication
      ↓
postgres account
      ↓
usesuper = true
      ↓
PostgreSQL superuser access
```

### Tomcat

```text
TCP/8180 exposed
      ↓
Tomcat Manager accessible
      ↓
tomcat:tomcat accepted
      ↓
Authenticated administrative access
      ↓
WAR deployment functionality
```

These chains demonstrate why service enumeration should not stop at identifying an open port.

The security impact depends on what functionality is exposed behind the service and what access controls are actually enforced.

---

# Most Significant Findings

The most significant confirmed findings from the assessment were:

| Priority | Service | Finding |
|---|---|---|
| Critical | MySQL | Passwordless remote root access |
| Critical | MySQL | `root@%` with unrestricted privileges |
| Critical | PostgreSQL | Remote PostgreSQL superuser access |
| Critical | Tomcat | Weak/default Manager credentials |
| Critical | Tomcat | Administrative application deployment functionality |
| High | NFS | Root filesystem exported to unrestricted clients |
| High | NFS | Remote access to `/etc/shadow` |
| High | NFS | Remote write access confirmed |
| High | Samba | Anonymous writable SMB share |
| High | Samba | SMB1 available |
| High | Tomcat | Legacy Apache Tomcat 5.5 |
| High | PostgreSQL | Legacy PostgreSQL 8.3.1 |
| High | Samba | Anonymous access to SMB resources |
| Medium | RPC | Exposed RPC portmapper |
| Medium | RPC | Multiple RPC services exposed |

The exact severity classification for each individual finding is documented in the corresponding service `findings` files.

---

# Overall Security Assessment

The target presents a **highly insecure network configuration**, with multiple independent paths to significant unauthorized access.

The most concerning characteristics are:

1. **Administrative services exposed over the network.**
2. **Default or weak credentials.**
3. **Anonymous access to network resources.**
4. **Excessive administrative privileges.**
5. **Unrestricted filesystem exports.**
6. **Legacy and obsolete software.**
7. **Legacy protocols such as SMB1 and Telnet.**
8. **Management interfaces exposed remotely.**

Several findings were independently sufficient to represent serious security weaknesses.

However, the greatest risk comes from their combination.

For example, the MySQL configuration combined:

```text
Remote exposure
+
Passwordless authentication
+
Root account
+
Wildcard host
+
Global privileges
+
GRANT OPTION
```

Similarly, the NFS configuration combined:

```text
Remote exposure
+
Root filesystem export
+
Wildcard client access
+
Sensitive file disclosure
+
Remote write access
```

These are examples of configurations where multiple individual weaknesses compound into a significantly greater overall risk.

---

# Remediation Priorities

## Immediate

1. Remove passwordless remote MySQL root access.
2. Remove unnecessary `root@%` access.
3. Restrict MySQL TCP/3306 to trusted hosts.
4. Restrict PostgreSQL TCP/5432 to authorized hosts.
5. Remove unnecessary remote PostgreSQL superuser access.
6. Replace weak/default Tomcat Manager credentials.
7. Restrict access to the Tomcat Manager application.
8. Remove anonymous SMB write access.
9. Restrict NFS exports to explicitly authorized clients.
10. Remove unnecessary root filesystem exports.

---

## Short Term

11. Apply least-privilege permissions to database accounts.
12. Remove unnecessary `GRANT OPTION` privileges.
13. Separate application database accounts.
14. Review all NFS exports and filesystem permissions.
15. Review all Samba shares and anonymous-access settings.
16. Disable SMB1 where compatibility requirements permit.
17. Restrict RPC services to trusted networks.
18. Disable unnecessary RPC-dependent services.
19. Review Tomcat administrative roles and deployment permissions.
20. Restrict remote administration services to trusted networks.

---

## Longer Term

21. Upgrade legacy MySQL components.
22. Upgrade PostgreSQL 8.3.1 to a supported release.
23. Replace Apache Tomcat 5.5 with a supported release.
24. Remove obsolete protocols and services.
25. Establish regular patch-management procedures.
26. Establish regular permission and account reviews.
27. Implement network segmentation between users, servers and management interfaces.
28. Monitor authentication and administrative activity.
29. Establish centralized logging and security monitoring.
30. Perform periodic vulnerability assessments.

---

# Evidence and Documentation

The assessment evidence is organized by service.

Each service directory contains:

```text
enumeration/
evidence/
findings/
```

This provides a clear separation between:

**Enumeration**

What was discovered and how it was validated.

**Evidence**

The screenshots and technical artifacts supporting the assessment.

**Findings**

The security significance, impact, severity and remediation associated with the observed weaknesses.

The structure allows each finding to be traced back to the evidence that supports it.

---

# Assessment Limitations

The assessment was performed against the intentionally vulnerable Metasploitable 2 laboratory environment.

The results should therefore not be interpreted as representative of a modern production system.

Testing was deliberately constrained to enumeration, validation and controlled security testing.

The following activities were not performed:

- Destructive exploitation.
- Persistence.
- Malware deployment.
- Privilege escalation beyond what was necessary to validate findings.
- Destructive database operations.
- Destructive filesystem modification.
- Malicious WAR deployment.
- Remote code execution testing against Tomcat.
- Post-compromise persistence.

Where write access was tested, the activity was limited to controlled and reversible operations.

For example:

- NFS: harmless file creation under `/tmp`.
- Samba: creation and removal of a temporary directory.
- Tomcat: controlled upload-endpoint interaction using a harmless text file.

---

# Key Lessons

The assessment demonstrates several fundamental principles of security assessment.

## 1. An Open Port Is Only the Starting Point

Identifying a service does not establish whether it is vulnerable.

The assessment therefore progressed from:

```text
Port
↓
Service
↓
Version
↓
Authentication
↓
Authorization
↓
Exposed functionality
↓
Security impact
```

---

## 2. Configuration Can Be More Important Than Exploitation

Several of the most serious findings did not require exploitation.

The security impact was demonstrated directly through configuration and access validation.

Examples include:

```text
root@%
ALL PRIVILEGES ON *.*
```

```text
/
*
```

and:

```text
tomcat:tomcat
```

These configurations alone provided sufficient evidence of significant security weaknesses.

---

## 3. Least Privilege Is Fundamental

The assessment repeatedly demonstrated the consequences of excessive privileges.

Examples included:

- MySQL root access.
- PostgreSQL superuser access.
- NFS access to the root filesystem.
- Tomcat administrative deployment functionality.

Accounts and services should receive only the permissions required for their intended purpose.

---

## 4. Network Exposure Matters

Several findings became significantly more serious because the affected services were remotely accessible.

Examples include:

```text
TCP/3306
TCP/5432
TCP/8180
TCP/445
TCP/2049
TCP/111
```

Network segmentation and access-control restrictions are therefore important layers of defense.

---

# Final Assessment Conclusion

The Metasploitable 2 assessment demonstrated a broad and highly exposed attack surface containing multiple serious security weaknesses.

The assessment identified weaknesses across:

- Remote administration.
- File sharing.
- Network filesystem services.
- Database services.
- RPC services.
- Web application management.

The most severe confirmed issues included passwordless remote MySQL root access, remote PostgreSQL superuser access, unrestricted NFS root filesystem exposure, anonymous writable Samba access and weak Tomcat Manager credentials combined with administrative deployment functionality.

The assessment also identified multiple legacy technologies and protocols, including obsolete database versions, Apache Tomcat 5.5, SMB1 and Telnet.

Importantly, the assessment did not rely solely on vulnerability identification. Where appropriate, access controls and security impact were validated through controlled testing, while avoiding destructive exploitation.

The complete assessment therefore demonstrates the following workflow:

```text
Reconnaissance
      ↓
Service Discovery
      ↓
Enumeration
      ↓
Authentication Validation
      ↓
Authorization Validation
      ↓
Controlled Security Testing
      ↓
Evidence Collection
      ↓
Risk Analysis
      ↓
Remediation
```

The project demonstrates an end-to-end security assessment methodology in an isolated laboratory environment, with each significant observation supported by technical evidence and documented separately at the service level.
