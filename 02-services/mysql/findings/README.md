# MySQL Findings

## Critical — Unauthenticated Remote Root Access

### Description

The MySQL service exposed on TCP/3306 accepted a remote connection using the `root` account without requesting a password.

The connection was established with:

```bash
mysql --skip-ssl -h 192.168.56.20 -u root
```

The successful connection demonstrated that the database server was accessible remotely without conventional password authentication.

### Evidence

[View remote MySQL root access](../evidence/11_MYSQL03_remote_mysql_root_access.png)

The successful connection returned:

```text
Welcome to the MariaDB monitor.
Server version: 5.0.51a-3ubuntu5 (Ubuntu)
```

No password was requested during authentication.

### Impact

An attacker capable of reaching TCP/3306 could potentially obtain administrative access to the MySQL server without possessing valid credentials.

This significantly increases the risk of:

- Unauthorized database access.
- Exposure of application data.
- Modification or deletion of database contents.
- Creation or modification of database accounts.
- Further compromise of applications relying on the database.

### Severity

**Critical**

### Recommendation

- Require authentication for all remote database accounts.
- Disable passwordless remote root authentication.
- Disable remote root access where it is not explicitly required.
- Use dedicated, minimally privileged accounts for applications.
- Restrict TCP/3306 to trusted hosts and networks.

---

## Critical — Root Account Accessible from Any Host

### Description

The authenticated MySQL account was identified as:

```text
root@%
```

The `%` wildcard indicates that the root account is configured to accept connections from arbitrary hosts, subject to external network controls.

### Evidence

[View MySQL remote root identity](../evidence/11_MYSQL04_mysql_remote_root_identity.png)

The account was confirmed with:

```sql
SELECT USER();
SELECT CURRENT_USER();
```

The results were:

```text
root@192.168.56.10
root@%
```

`USER()` identifies the client connection, while `CURRENT_USER()` identifies the MySQL account used for authorization.

### Impact

The administrative account is not restricted to localhost or a specific trusted management host.

Combined with the lack of password authentication, this creates a particularly severe remote-access exposure.

### Severity

**Critical**

### Recommendation

- Remove unnecessary remote access for the root account.
- Restrict administrative accounts to specific trusted hosts.
- Use dedicated administrative accounts where remote administration is required.
- Apply network-level restrictions to TCP/3306.

---

## Critical — Excessive Root Privileges

### Description

The authenticated root account was granted unrestricted privileges over all databases and tables.

### Evidence

[View MySQL root privileges](../evidence/11_MYSQL07_mysql_root_full_privileges.png)

The following command was used:

```sql
SHOW GRANTS FOR CURRENT_USER();
```

The server returned:

```text
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION
```

This grants:

```text
ALL PRIVILEGES
```

over:

```text
*.*
```

and additionally enables:

```text
WITH GRANT OPTION
```

### Impact

Compromise of this account potentially provides complete administrative control over the databases hosted by the server.

The `GRANT OPTION` also allows the account to delegate its privileges to other MySQL accounts.

Potential consequences include:

- Unauthorized access to all databases.
- Modification or deletion of database contents.
- Creation or modification of accounts.
- Privilege delegation.
- Compromise of applications relying on the database.

### Severity

**Critical**

### Recommendation

- Apply the principle of least privilege.
- Remove unnecessary global privileges.
- Remove `GRANT OPTION` unless there is a documented administrative requirement.
- Use separate administrative and application accounts.
- Restrict privileges to the databases and operations actually required.

---

## High — Outdated MySQL Version

### Description

The target reported:

```text
MySQL 5.0.51a-3ubuntu5
```

This is a legacy MySQL implementation.

### Evidence

[View MySQL version enumeration](../evidence/11_MYSQL02_mysql_version_enumeration.png)

The version was identified through:

```bash
nmap -p 3306 --script=mysql-info 192.168.56.20
```

### Impact

Legacy database software may contain known vulnerabilities, unsupported components and security weaknesses that are absent from current supported releases.

In this assessment, however, the primary confirmed vulnerabilities are the authentication and authorization misconfigurations documented above.

### Severity

**High**

### Recommendation

- Upgrade to a supported MySQL release.
- Establish a regular database patch-management process.
- Remove legacy software where it is no longer required.
- Validate application compatibility before upgrading.

---

## High — Excessive Database Exposure

### Description

The authenticated account was able to enumerate multiple databases hosted by the target, including the `dvwa` application database.

### Evidence

[View MySQL database enumeration](../evidence/11_MYSQL05_mysql_database_enumeration.png)

The following command was used:

```sql
SHOW DATABASES;
```

The server returned databases including:

```text
dvwa
metasploit
mysql
owasp10
tikiwiki
tikiwiki195
```

Access to the DVWA database was subsequently confirmed.

[View DVWA database access](../evidence/11_MYSQL08_dvwa_database_access.png)

The following commands were used:

```sql
USE dvwa;
SHOW TABLES;
```

The database contained tables including:

```text
guestbook
users
```

### Impact

Access to multiple application databases increases the potential impact of a compromised database account.

Depending on the information stored within those databases, unauthorized access could expose application data, credentials or other sensitive information.

### Severity

**High**

### Recommendation

- Use separate database accounts for individual applications.
- Restrict each account to only the databases it requires.
- Avoid using administrative accounts for application connectivity.
- Restrict network access to the database server.
- Review database permissions regularly.

---

## Security Assessment Summary

The MySQL assessment demonstrated a complete misconfiguration chain:

```text
TCP/3306 exposed
        ↓
Legacy MySQL service
        ↓
Remote root connection without password
        ↓
root@%
        ↓
ALL PRIVILEGES ON *.*
        ↓
WITH GRANT OPTION
        ↓
Multiple databases accessible
        ↓
DVWA database accessible
```

The most severe issue is the combination of **passwordless remote root authentication**, **wildcard host access** and **unrestricted administrative privileges**.

Individually, each configuration weakness represents a significant security concern. Combined, they provide a path to complete administrative control of the database server for any attacker able to reach TCP/3306.

## Overall Risk

**Critical**

The critical rating is driven primarily by the confirmed remote administrative access and unrestricted privileges, rather than solely by the legacy MySQL version.

## Remediation Priority

### Immediate

1. Disable passwordless remote root authentication.
2. Remove unnecessary `root@%` access.
3. Restrict TCP/3306 to trusted hosts and networks.
4. Replace the remote root account with appropriately scoped administrative accounts.

### Short Term

5. Remove unnecessary global privileges.
6. Remove `GRANT OPTION` unless explicitly required.
7. Separate application database accounts.
8. Restrict application accounts to their required databases.

### Longer Term

9. Upgrade the legacy MySQL installation.
10. Establish regular database patching and permission reviews.

## Evidence

The finding is supported by the following evidence:

```text
11_MYSQL03_remote_mysql_root_access.png
11_MYSQL04_mysql_remote_root_identity.png
11_MYSQL05_mysql_database_enumeration.png
11_MYSQL06_mysql_user_host_enumeration.png
11_MYSQL07_mysql_root_full_privileges.png
11_MYSQL08_dvwa_database_access.png
```

The service and version identification are additionally supported by:

```text
11_MYSQL01_nmap_mysql_detection.png
11_MYSQL02_mysql_version_enumeration.png
```

## Portfolio Conclusion

**"The MySQL service exposed on TCP/3306 was found to permit remote authentication as the root account without a password. The authenticated account was configured as root@% and possessed ALL PRIVILEGES on all databases with GRANT OPTION. Multiple databases were enumerated and access to the DVWA application database was confirmed. The combination of passwordless remote administrative access, unrestricted host access and excessive privileges represents a critical security misconfiguration. Remediation should prioritize restricting remote root access, enforcing authentication, applying least privilege and limiting database network exposure."**
