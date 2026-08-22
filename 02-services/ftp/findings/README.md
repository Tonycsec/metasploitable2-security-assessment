# FTP Security Findings

## Overview

The FTP service exposed by the Metasploitable 2 host presented several security weaknesses identified during the service assessment.

The assessment identified two primary findings:

1. **Outdated FTP Software — vsftpd 2.3.4**
2. **Anonymous FTP Access**

The findings were identified through service detection, configuration assessment and non-destructive enumeration.

No exploitation of the identified vulnerabilities was performed as part of this assessment.

---

## Finding 01 — Outdated FTP Software

### Description

The FTP service was running **vsftpd 2.3.4**, an obsolete version of the FTP server.

This version is historically associated with a well-known backdoor vulnerability. The presence of an obsolete network-facing service therefore represents a significant security concern.

The software version was identified through Nmap service and version detection.

### Evidence

The service and version were identified during the FTP enumeration:

![View Nmap service detection evidence](../evidence/01_FTP01_nmap_ftp_detection.png)

The detected service was:

```text
21/tcp open ftp vsftpd 2.3.4
```

### Security Impact

Running outdated network-facing software increases the risk of compromise because known vulnerabilities may remain unpatched.

In this case, the identified version is associated with a historically documented backdoor vulnerability. However, the vulnerability was **not exploited or independently validated during this assessment**.

The finding therefore represents a version-based security risk rather than a claim of successful exploitation.

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

The server accepted an anonymous login without requiring a normal user account.

After authentication, the FTP session allowed interaction with the remote directory. The directory enumeration confirmed that the anonymous account could access the FTP service.

### Evidence

Anonymous authentication was successfully established:

![View anonymous FTP login evidence](../evidence/01_FTP02_ftp_anonymous_login.png)

The subsequent FTP session allowed directory interaction:

![View FTP directory enumeration evidence](../evidence/01_FTP03_ftp_directory_listing.png)

The directory enumeration was performed using:

```text
ftp> ls
229 Entering Extended Passive Mode
150 Here comes the directory listing.
226 Directory send OK.
ftp> pwd
Remote directory: /
```

### Security Impact

Anonymous FTP access allows unauthenticated users to interact with the FTP service.

Depending on the permissions assigned to the anonymous account, this may allow unauthorized users to enumerate, retrieve or potentially modify files.

In this assessment, anonymous access was confirmed and the accessible directory was enumerated. The assessment did not attempt to modify or upload files through the anonymous account.

Even where sensitive information is not directly exposed, anonymous access increases the attack surface and can facilitate further information gathering.

### Severity

**High**

The severity reflects the exposure of a network-accessible file-transfer service to unauthenticated users and the potential for unauthorized access to files or directories.

### Recommendation

* Disable anonymous FTP access unless it is explicitly required.
* Require authenticated access for FTP users.
* Review the permissions associated with the anonymous account.
* Review all files and directories accessible through anonymous access.
* Restrict FTP access to trusted networks or users.
* Prefer SFTP or another secure file-transfer mechanism where possible.

---

## Overall Assessment

The FTP service presents a significant security risk due to the combination of **obsolete software** and **anonymous authentication**.

The assessment confirmed that:

* `vsftpd 2.3.4` was exposed on TCP/21.
* Anonymous FTP authentication was enabled.
* The anonymous session allowed interaction with the remote FTP directory.

These weaknesses should be addressed by upgrading or replacing the FTP service, disabling unnecessary anonymous access and restricting network exposure.

The findings identified during this assessment provide the basis for the corresponding remediation recommendations in the project's remediation phase.

These weaknesses should be addressed by upgrading or replacing the FTP service, disabling unnecessary anonymous access and restricting network exposure.

The findings identified during this assessment provide the basis for the remediation recommendations documented in the project's remediation phase.
