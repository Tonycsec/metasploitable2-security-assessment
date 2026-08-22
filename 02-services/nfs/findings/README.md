# NFS Security Findings

## Overview

The NFS assessment identified a critical filesystem-export misconfiguration on the Metasploitable 2 host.

The server exported the entire root filesystem (`/`) to all clients (`*`). The export was successfully mounted from the assessment host, providing access to the target filesystem.

The assessment subsequently confirmed access to sensitive system files, including `/etc/passwd` and `/etc/shadow`, as well as remote write access through controlled file creation under `/tmp`.

The primary finding is:

**Unrestricted NFS export of the root filesystem with confirmed remote read and write access.**

---

## Finding 01 — Unrestricted Root Filesystem Export

### Description

The NFS server exported:

```text
/
```

to:

```text
*
```

The export was identified using:

```bash
showmount -e 192.168.56.20
```

The server returned:

```text
Export list for 192.168.56.20:
/ *
```

[View NFS export evidence](../evidence/14_NFS02_nfs_export_root_to_all.png)

The wildcard `*` indicates that the export was not restricted to a specific trusted client or network.

Exporting the entire root filesystem in this manner represents a severe NFS configuration weakness.

### Security Impact

An unauthorized NFS client may be able to mount the exported filesystem and access files belonging to the target operating system.

Because the exported path is `/`, the exposure potentially extends to the complete filesystem, including:

- System configuration.
- User information.
- Application data.
- Service configuration.
- Authentication material.
- Other sensitive operating-system resources.

### Severity

**Critical**

The severity reflects the combination of an unrestricted root filesystem export and the successful remote mounting demonstrated during the assessment.

### Recommendation

- Never export the entire root filesystem unnecessarily.
- Export only directories that genuinely require NFS access.
- Restrict exports to specific trusted hosts or networks.
- Use appropriate read/write restrictions.
- Review NFS access-control configuration regularly.
- Restrict TCP/UDP 2049 and associated RPC services at the network layer.

---

## Finding 02 — Remote Access to Sensitive System Files

### Description

After mounting the exported root filesystem, the assessment host was able to access sensitive system files.

The following file was successfully read:

```text
/etc/passwd
```

[View /etc/passwd disclosure evidence](../evidence/14_NFS04_nfs_passwd_disclosure.png)

The file contained information about local system accounts, including usernames, UIDs, GIDs, home directories and login shells.

More significantly, the assessment also successfully accessed:

```text
/etc/shadow
```

[View sanitized /etc/shadow disclosure evidence](../evidence/14_NFS05_nfs_shadow_disclosure_%28REDACTED%29.png)

The `/etc/shadow` file contains password authentication material and should not be remotely readable by an unauthorized client.

### Security Impact

Exposure of `/etc/passwd` provides useful account information for further reconnaissance.

Exposure of `/etc/shadow` is substantially more serious because it provides password hashes that may potentially be subjected to offline password-cracking attacks.

The underlying security issue is the unauthorized filesystem access enabled by the NFS export configuration.

### Severity

**Critical**

The finding involves confirmed remote access to sensitive authentication-related data.

### Recommendation

- Restrict NFS exports to authorized clients.
- Do not export sensitive system directories through NFS.
- Ensure `/etc/shadow` cannot be accessed through network filesystem exports.
- Review NFS identity and access-control configuration.
- Use appropriate filesystem permissions.
- Rotate credentials if authentication material may have been exposed.
- Monitor for unauthorized access to NFS resources.

### Evidence Redaction

The original `/etc/shadow` screenshot contained password hashes.

Before publication to the public GitHub repository, the hashes were visually redacted using an AI-assisted image-editing tool.

The redaction does not alter the technical evidence relevant to the finding. It was performed solely to prevent credential material from being unnecessarily exposed in the public repository.

The finding is the **confirmed unauthorized read access to `/etc/shadow`**, not the publication of the hashes themselves.

---

## Finding 03 — Confirmed Remote Write Access

### Description

The assessment also tested whether the mounted filesystem permitted file creation.

A controlled test was performed under `/tmp`:

```bash
touch /mnt/metasploitable2_nfs/tmp/test2_nfs
```

The file was then verified:

```bash
ls -l /mnt/metasploitable2_nfs/tmp/test2_nfs
```

The file was successfully created.

[View NFS write-access evidence](../evidence/14_NFS06_nfs_write_access.png)

This demonstrated that the NFS exposure was not limited to read access.

The assessment host was able to create a file through the remotely mounted filesystem.

### Security Impact

Remote write access introduces an integrity impact in addition to the confidentiality impact demonstrated by the sensitive-file disclosure.

The exact consequences depend on:

- NFS identity mapping.
- UID/GID configuration.
- Filesystem permissions.
- The location being modified.
- The privileges available to the NFS client.

The assessment deliberately limited the test to creation of a harmless file under `/tmp`.

### Severity

**High**

In combination with the unrestricted root export, confirmed write access substantially increases the potential impact of the misconfiguration.

### Recommendation

- Use read-only NFS exports wherever write access is unnecessary.
- Restrict writable exports to authorized clients.
- Apply appropriate filesystem permissions.
- Review UID/GID mapping.
- Use `root_squash` where appropriate.
- Prevent unauthorized clients from accessing writable NFS exports.

---

## Finding 04 — Complete Root Filesystem Exposure

### Description

The individual observations above form a single, severe configuration issue.

The assessment demonstrated the following chain:

```text
/ exported to *
       ↓
Remote NFS mount succeeds
       ↓
Complete root filesystem accessible
       ↓
/etc/passwd readable
       ↓
/etc/shadow readable
       ↓
Remote file creation possible
```

[View NFS service identification evidence](../evidence/14_NFS01_nmap_nfs_detection.png)

[View mounted root filesystem evidence](../evidence/14_NFS03_nfs_root_filesystem_mounted.png)

The evidence demonstrates that the exposure was not theoretical: the exported root filesystem was successfully mounted and accessed from the assessment host.

### Security Impact

The combination of unrestricted export, sensitive-file access and remote write capability creates a serious confidentiality and integrity risk.

An unauthorized client with network access to the NFS service could potentially:

- Read system files.
- Enumerate local accounts.
- Obtain password authentication material.
- Access application configuration.
- Access user and service data.
- Modify files where filesystem permissions permit it.

The exact consequences depend on NFS identity mapping and filesystem permissions.

### Severity

**Critical**

This is the primary NFS finding.

### Recommendation

The NFS configuration should be redesigned so that:

- Only required directories are exported.
- Exports are restricted to trusted clients.
- Read-only access is used where possible.
- `root_squash` is enabled where appropriate.
- Sensitive system directories are not exported.
- NFS services are restricted at the network layer.
- NFS configuration is reviewed regularly.

---

## Evidence Chain

The six pieces of evidence demonstrate the finding progressively:

### 1. NFS Service Identification

Nmap confirmed NFS on TCP/2049:

[View NFS service detection evidence](../evidence/14_NFS01_nmap_nfs_detection.png)

### 2. Export Enumeration

`showmount` demonstrated that `/` was exported to `*`:

[View NFS export evidence](../evidence/14_NFS02_nfs_export_root_to_all.png)

### 3. Remote Mount

The exported root filesystem was successfully mounted:

[View mounted root filesystem evidence](../evidence/14_NFS03_nfs_root_filesystem_mounted.png)

### 4. Account Information Disclosure

`/etc/passwd` was readable through the mounted filesystem:

[View /etc/passwd disclosure evidence](../evidence/14_NFS04_nfs_passwd_disclosure.png)

### 5. Authentication Material Disclosure

`/etc/shadow` was readable through the mounted filesystem:

[View sanitized /etc/shadow evidence](../evidence/14_NFS05_nfs_shadow_disclosure_%28REDACTED%29.png)

### 6. Remote Write Access

A controlled file-creation test succeeded under `/tmp`:

[View NFS write-access evidence](../evidence/14_NFS06_nfs_write_access.png)

The complete evidence chain is:

**Service identification → unrestricted export → successful mount → sensitive-file disclosure → authentication-material disclosure → remote write access**

---

## Remediation Priority

This finding should be treated as a **critical remediation priority**.

The recommended remediation order is:

1. Remove the unrestricted `/` export.
2. Restrict NFS exports to explicitly authorized clients.
3. Review all existing exports and remove unnecessary ones.
4. Use read-only exports wherever possible.
5. Enable `root_squash` where appropriate.
6. Review UID/GID mapping and filesystem permissions.
7. Rotate credentials if password authentication material was exposed.
8. Restrict NFS and related RPC services at the firewall/network layer.
9. Review logs for unauthorized NFS access.

After remediation, the export configuration should be re-tested to verify that unauthorized clients can no longer mount or access sensitive files.

---

## Scope and Limitations

The assessment was conducted against the intentionally vulnerable Metasploitable 2 laboratory environment.

Testing was limited to:

- NFS service identification.
- Export enumeration.
- Controlled remote mounting.
- File-access validation.
- Sensitive-file disclosure validation.
- Controlled write-access testing.

No persistence, privilege escalation, destructive modification or post-compromise activity was performed.

The write-access test was deliberately limited to creating a harmless file under `/tmp`.

---

## Overall Assessment

The NFS configuration represents a **critical security weakness**.

The server exported the entire root filesystem (`/`) to all clients (`*`), and the assessment host was able to mount that filesystem remotely.

This provided confirmed access to sensitive system files, including `/etc/passwd` and `/etc/shadow`, and allowed controlled remote file creation.

The issue demonstrates both:

- **Confidentiality impact** through unauthorized filesystem and authentication-material access.
- **Integrity impact** through confirmed remote write access.

The primary remediation is to remove the unrestricted root export and implement strict NFS export and client-access controls.

## Portfolio Conclusion

**"The NFS service was critically misconfigured, exporting the root filesystem (`/`) to all clients (`*`). The export was successfully mounted remotely, providing access to the target filesystem. Sensitive files including `/etc/passwd` and `/etc/shadow` were readable, and remote write access was demonstrated through controlled file creation under `/tmp`. Password hashes were redacted from the public evidence. The assessment was limited to controlled configuration and access validation, with no persistence, privilege escalation or destructive modification performed."**
