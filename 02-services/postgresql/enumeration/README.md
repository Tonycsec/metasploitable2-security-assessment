# PostgreSQL Enumeration

## Service Information

- **Service:** PostgreSQL
- **Port:** TCP/5432
- **Target:** Metasploitable 2
- **Version identified:** PostgreSQL 8.3.1
- **Authenticated account:** postgres
- **Privilege level:** Superuser
- **Authentication:** Remote password authentication
- **Assessment approach:** Enumeration and privilege validation only; no exploitation performed.

## 1. Service Discovery

We first identified the service exposed on TCP/5432.

**Command used**

```bash
nmap -p 5432 -sV 192.168.56.20
```

Nmap identified:

```text
5432/tcp open postgresql PostgreSQL DB 8.3.0 - 8.3.7
```

![View PostgreSQL service detection evidence](../evidence/12_POSTGRESQL01_nmap_postgresql_detection.png)

This established that PostgreSQL was exposed remotely and that the target was running a legacy PostgreSQL implementation.

## 2. PostgreSQL Authentication Attempt

We attempted to connect remotely using the PostgreSQL client:

```bash
psql -h 192.168.56.20 -U postgres
```

The server requested authentication:

```text
Password for user postgres:
```

The initial connection attempt encountered an SSL compatibility error because the modern PostgreSQL client attempted to negotiate a protocol that was not supported by the legacy server.

The error was:

```text
SSL error: unsupported protocol
```

This was treated as a client/server compatibility issue rather than evidence that PostgreSQL was inaccessible.

## 3. Successful PostgreSQL Connection

A subsequent connection attempt succeeded:

```bash
psql -h 192.168.56.20 -U postgres
```

The client reported:

```text
psql (18.4 (Debian 18.4-1+b1), server 8.3.1)
WARNING: psql major version 18, server major version 8.3.
Some psql features might not work.
```

The successful session was represented by:

```text
postgres=#
```

![View remote PostgreSQL access evidence](../evidence/12_POSTGRESQL02_remote_postgresql_access.png)

This confirmed that the `postgres` account could authenticate remotely against the database service.

No credentials are documented in this repository.

## 4. PostgreSQL Version Enumeration

Once connected, we queried the server directly.

**Command used**

```sql
SELECT version();
```

The server returned:

```text
PostgreSQL 8.3.1 on i486-pc-linux-gnu,
compiled by GCC cc (GCC) 4.2.3 (Ubuntu 4.2.3-2ubuntu4)
```

![View PostgreSQL version evidence](../evidence/12_POSTGRESQL03_postgresql_version.png)

This confirmed that the target was running PostgreSQL 8.3.1.

The version belongs to a legacy PostgreSQL generation and represents a significant maintenance and security concern.

## 5. Confirming the Current Database User

We then verified the identity associated with the authenticated session.

**Command used**

```sql
SELECT current_user;
```

The result was:

```text
current_user
------------
postgres
```

![View PostgreSQL current-user evidence](../evidence/12_POSTGRESQL04_postgresql_current_user.png)

This confirmed that the remote session was authenticated as the PostgreSQL `postgres` account.

## 6. Checking PostgreSQL Superuser Privileges

Rather than assuming that the `postgres` account had elevated privileges based solely on its name, we explicitly verified its authorization level.

The modern PostgreSQL client also produced compatibility errors when attempting the `\du` meta-command because the PostgreSQL 18 client expected catalog columns that are not present in PostgreSQL 8.3.

We therefore queried the underlying system catalog directly.

**Command used**

```sql
SELECT usesuper
FROM pg_user
WHERE usename = current_user;
```

The result was:

```text
usesuper
--------
t
```

The value `t` means **true**.

Therefore, the authenticated `postgres` account was confirmed to be a PostgreSQL superuser.

![View PostgreSQL superuser confirmation evidence](../evidence/12_POSTGRESQL05_postgresql_superuser_confirmation.png)

This is the most important evidence in the PostgreSQL assessment.

## 7. Client Compatibility Issues

During enumeration, we also attempted PostgreSQL meta-commands such as:

```text
\l
```

and:

```text
\du
```

These produced errors because the modern PostgreSQL 18 client generated queries referencing catalog columns that do not exist on the PostgreSQL 8.3 server.

For example:

```text
ERROR: column d.datcollate does not exist
```

and:

```text
ERROR: column r.rolreplication does not exist
```

These errors were treated as **client/server compatibility issues**, not as security findings.

The issue demonstrated that modern PostgreSQL client tooling may not be fully compatible with a legacy database server.

Where the meta-commands failed, direct SQL queries against the appropriate legacy system catalogs were used to obtain the required information.

No additional screenshot is required for these compatibility errors in the public portfolio.

## Security Analysis

The assessment established the following chain:

```text
TCP/5432 exposed
        ↓
Legacy PostgreSQL service identified
        ↓
Remote authentication succeeds
        ↓
Authenticated as postgres
        ↓
usesuper = true
        ↓
PostgreSQL superuser access confirmed
```

The principal security concern is the combination of:

- A remotely accessible database service.
- A legacy PostgreSQL implementation.
- Remote authentication using the highly privileged `postgres` account.
- Confirmed PostgreSQL superuser privileges.

## Security Assessment Summary

### Legacy PostgreSQL Version

The target was running:

```text
PostgreSQL 8.3.1
```

**Impact:**

The server is running a very old PostgreSQL generation and should not be considered appropriate for a modern production environment.

**Recommendation:**

Upgrade to a supported PostgreSQL release and maintain a regular patch-management lifecycle.

### Remote Database Access

PostgreSQL was accessible remotely on:

```text
TCP/5432
```

and the `postgres` account successfully authenticated from the assessment host.

**Impact:**

Exposing a database service increases the network attack surface and permits remote authentication attempts.

**Recommendation:**

Restrict TCP/5432 to trusted application servers and authorized administration hosts using network controls and PostgreSQL access rules.

### Remote Superuser Access

The remotely authenticated `postgres` account was explicitly confirmed to have:

```text
usesuper = true
```

**Impact:**

A compromised PostgreSQL superuser account has extensive administrative capabilities over the database environment.

**Recommendation:**

- Avoid routine remote access using the `postgres` superuser.
- Use dedicated administrative accounts where appropriate.
- Restrict administrative access to trusted management hosts.
- Apply least privilege to application accounts.
- Use strong authentication controls.
- Review and restrict `pg_hba.conf` rules.

## Scope and Limitations

The assessment was conducted against the intentionally vulnerable Metasploitable 2 laboratory environment.

Testing was limited to:

- Service identification.
- Version detection.
- Remote authentication validation.
- Current-user identification.
- Superuser privilege validation.
- Client/server compatibility troubleshooting.

No exploitation was performed.

No destructive database actions were performed.

No database contents were modified or deleted.

## Evidence

The PostgreSQL assessment uses five screenshots:

```text
12_POSTGRESQL01_nmap_postgresql_detection.png
12_POSTGRESQL02_remote_postgresql_access.png
12_POSTGRESQL03_postgresql_version.png
12_POSTGRESQL04_postgresql_current_user.png
12_POSTGRESQL05_postgresql_superuser_confirmation.png
```

The evidence chain is:

**Service identification → remote authentication → version identification → authenticated identity → superuser privilege validation**

## Commands

### Service detection

```bash
nmap -p 5432 -sV 192.168.56.20
```

### PostgreSQL connection

```bash
psql -h 192.168.56.20 -U postgres
```

### Server version

```sql
SELECT version();
```

### Current user

```sql
SELECT current_user;
```

### Superuser verification

```sql
SELECT usesuper
FROM pg_user
WHERE usename = current_user;
```

### Client compatibility commands attempted

```text
\l
\du
```

These were retained as lab troubleshooting notes because of the PostgreSQL 18 client / PostgreSQL 8.3 server compatibility issue.
