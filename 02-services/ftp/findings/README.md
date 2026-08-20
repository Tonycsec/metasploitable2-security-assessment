# FTP Security Findings

## Overview

The FTP service exposed by the Metasploitable 2 host presented several security weaknesses identified during the service assessment.

The assessment identified two primary findings:

1. **Outdated FTP Software — vsftpd 2.3.4**
2. **Anonymous FTP Access**

No exploitation of the identified vulnerabilities was performed. The findings are based on service identification, configuration analysis and non-destructive enumeration.

---

## Finding 01 — Outdated FTP Software

### Description

The FTP service was running **vsftpd 2.3.4**, an obsolete version of the FTP server.

This version is historically associated with a well-known backdoor vulnerability. The presence of an obsolete network-facing service therefore represents a significant security concern.

The assessment identified the vulnerable software version through Nmap service and version detection.

### Evidence

```text
21/tcp open ftp vsftpd 2.3.4
```

The corresponding service detection evidence is documented in the FTP enumeration section.

### Security Impact

Running outdated network-facing software increases the risk of compromise because known vulnerabilities may remain unpatched.

In this case, the identified version is associated with a historically documented backdoor vulnerability. However, the vulnerability was **not exploited or independently validated during this assessment**.

### Severity

**High**

The severity reflects the combination of an obsolete network-facing service and the historical security issues associated with the identified version.

### Recommendation

* Upgrade vsftpd to a currently supported version.
* Remove obsolete software from production systems.
* Replace FTP with a secure file-transfer mechanism such as SFTP where appropriate.
* Maintain a regular patch and vulnerability management process.
* Restrict network access to file-transfer services to trusted hosts or networks.

---

## Finding 02 — Anonymous FTP Access

### Description

Anonymous authentication was successfully confirmed on the FTP service.

This configuration allows users to access the FTP service without providing a normal user account.

The assessment subsequently enumerated the directory contents available through the anonymous session to determine what information was exposed.

### Evidence

The FTP server accepted an anonymous login and allowed access to the service.

The relevant evidence is documented in the FTP enumeration section.

### Security Impact

Anonymous FTP access can expose files or directories to unauthenticated users.

The actual impact depends on the permissions assigned to the anonymous account and the information available through the exposed directories.

Even when no sensitive data is directly exposed, anonymous access increases the attack surface and may facilitate information gathering.

### Severity

**High**

The severity is based on the exposure of an authenticated service to unauthenticated users and the potential for unauthorized access to files or directories.

### Recommendation

* Disable anonymous FTP access unless it is explicitly required.
* Require authenticated access for FTP users.
* Review the permissions associated with anonymous access.
* Review all files and directories accessible through the anonymous account.
* Restrict FTP access to trusted networks or users.
* Prefer SFTP or another secure file-transfer mechanism where possible.

---

## Overall Assessment

The FTP service presents a significant security risk due to the combination of **obsolete software** and **anonymous authentication**.

These weaknesses should be addressed by upgrading or replacing the FTP service, disabling unnecessary anonymous access and restricting network exposure.

The findings identified during this assessment provide the basis for the remediation recommendations documented in the project's remediation phase.

