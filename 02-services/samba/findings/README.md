# Samba — Security Findings

## Overview

The assessment identified several security weaknesses in the Samba configuration:

- Anonymous SMB share enumeration.
- Anonymous access to the `tmp` share.
- Anonymous read access to the `tmp` share.
- Anonymous write access to the `tmp` share.
- SMB1 available on the server.

The most significant finding was the ability of an unauthenticated SMB client to create and remove directories on the `tmp` share.

No exploitation beyond controlled access-permission validation was performed.

---

# Finding 01 — Anonymous SMB Share Enumeration

**Severity:** Medium

**Service:** Samba / SMB

**Port:** TCP/445

### Description

The Samba service permitted unauthenticated users to enumerate the SMB resources available on the target.

The following command successfully returned the available shares without requiring credentials:

```bash
smbclient -L //192.168.56.20 -N
```

The following resources were identified:

```text
print$
tmp
opt
IPC$
ADMIN$
```

The server was identified as:

```text
Samba 3.0.20-Debian
```

### Security Impact

Anonymous share enumeration provides unauthenticated users with information about the resources exposed by the SMB server.

This information can assist subsequent reconnaissance and help identify potentially accessible or misconfigured resources.

### Evidence

[Anonymous SMB share enumeration](../enumeration/08_SAMBA01_smb_anonymous_enumeration.png)

### Recommendation

- Disable anonymous SMB enumeration where it is not explicitly required.
- Require authentication for SMB resources.
- Review all exposed shares and remove unnecessary ones.
- Restrict SMB access to trusted networks.

---

# Finding 02 — Anonymous Access to `tmp`

**Severity:** High

**Service:** Samba / SMB

**Port:** TCP/445

### Description

The `tmp` SMB share accepted anonymous connections.

The following command established a session without providing a username or password:

```bash
smbclient //192.168.56.20/tmp -N
```

The connection succeeded and provided an interactive SMB session.

### Security Impact

An unauthenticated network client was able to interact directly with an SMB share without first authenticating.

This bypasses the normal access-control boundary expected for protected SMB resources.

### Evidence

[Anonymous access to tmp share](../enumeration/08_SAMBA02_smb_tmp_anonymous_access.png)

### Recommendation

- Disable anonymous access unless it is explicitly required.
- Require authentication for SMB shares.
- Restrict SMB access to trusted hosts and networks.
- Review the necessity of the `tmp` share.

---

# Finding 03 — Anonymous Read Access to `tmp`

**Severity:** High

**Service:** Samba / SMB

**Port:** TCP/445

### Description

After establishing an anonymous connection to the `tmp` share, we successfully listed its contents using:

```text
ls
```

This confirmed that anonymous users had read access to the share.

### Security Impact

An unauthenticated user could inspect the contents of an SMB-accessible resource.

The potential impact depends on what files are stored within the share and how those files are subsequently used by the operating system or applications.

### Evidence

[Anonymous read access](../enumeration/08_SAMBA03_smb_tmp_read_access.png)

### Recommendation

- Remove anonymous read permissions.
- Apply least-privilege access controls.
- Restrict the share to authenticated and authorized users.
- Review the contents of the share for unnecessary or sensitive data.

---

# Finding 04 — Anonymous Write Access to `tmp`

**Severity:** High

**Service:** Samba / SMB

**Port:** TCP/445

### Description

The anonymous SMB session was not limited to read access.

Write permissions were explicitly validated by creating a temporary directory:

```text
mkdir test2
```

The directory was successfully created.

We then removed the test directory:

```text
rmdir test2
```

This confirmed that an unauthenticated SMB client had write access to the `tmp` share.

The test was deliberately limited to creating and removing a harmless temporary directory. No further modification or exploitation was performed.

### Security Impact

An unauthenticated network user could modify the contents of an SMB-accessible resource.

Depending on how the share is used by the operating system or applications, unauthorized write access could potentially allow:

- Modification of existing files.
- Placement of unauthorized files.
- Tampering with application data.
- Additional attacks if another service consumes files from the share.

The assessment did not test these scenarios. The confirmed finding is the **anonymous write permission itself**.

### Evidence

[Anonymous write access](../enumeration/08_SAMBA04_smb_tmp_write_access.png)

### Recommendation

- Disable anonymous write access.
- Remove write permissions from unauthenticated users.
- Apply least-privilege permissions.
- Use read-only access where write access is unnecessary.
- Review whether the `tmp` share is required.
- Restrict SMB access to trusted networks.

---

# Finding 05 — SMB1 Available

**Severity:** High

**Service:** Samba / SMB

**Port:** TCP/445

### Description

The assessment identified SMB1 availability on the target.

SMB1 is an obsolete SMB protocol version and should not normally be enabled on modern systems.

### Security Impact

Maintaining legacy protocol support increases the attack surface and may expose systems to vulnerabilities associated with obsolete SMB implementations.

The presence of SMB1 is therefore an additional security weakness in the Samba configuration.

### Recommendation

- Disable SMB1 where compatibility requirements permit.
- Use current SMB protocol versions.
- Review legacy systems that may depend on SMB1 before removing it.
- Monitor SMB connections for unexpected legacy-protocol usage.

---

# Supporting Observation — `opt` Access Denied

During the assessment, the `opt` share was also tested anonymously:

```bash
smbclient //192.168.56.20/opt -N
```

The server rejected the connection with:

```text
NT_STATUS_ACCESS_DENIED
```

This is **not a vulnerability finding**.

It is useful supporting evidence because it demonstrates that anonymous access was not universally permitted across every discovered SMB resource.

### Evidence

[Anonymous access denied to opt](../enumeration/08_SAMBA05_smb_denied_access.png)

---

# Risk Summary

| Finding | Severity | Confirmed |
|---|---|---|
| Anonymous SMB share enumeration | Medium | Yes |
| Anonymous access to `tmp` | High | Yes |
| Anonymous read access to `tmp` | High | Yes |
| Anonymous write access to `tmp` | High | Yes |
| SMB1 available | High | Yes |
| Anonymous access to `opt` | — | Denied |

The primary security concern is the combination of:

**Anonymous authentication → accessible SMB share → read access → write access**

This represents a significant access-control misconfiguration.

---

# Remediation Summary

The Samba configuration should be reviewed with the following priorities:

1. **Disable anonymous access** unless explicitly required.
2. **Remove anonymous write permissions** from the `tmp` share.
3. **Restrict SMB shares to authorized users and hosts.**
4. **Review and remove unnecessary shares.**
5. **Disable SMB1** where compatibility requirements permit.
6. **Restrict TCP/445 to trusted networks** using appropriate firewall controls.
7. **Apply least-privilege permissions** to all SMB resources.

---

# Assessment Conclusion

The Samba assessment identified a significant access-control misconfiguration.

The server allowed unauthenticated SMB enumeration and provided anonymous access to the `tmp` share. The share permitted both read and write operations, with write access validated through the controlled creation and removal of a temporary directory.

The assessment also identified SMB1 availability, adding an obsolete protocol to the exposed attack surface.

The `opt` share was deliberately tested as a comparison and correctly rejected anonymous access.

The assessment therefore demonstrates both **positive and negative security findings**: vulnerabilities and misconfigurations were documented where access was actually confirmed, while protected resources were not incorrectly reported as vulnerable.

**No exploitation beyond controlled access-permission validation was performed.**
