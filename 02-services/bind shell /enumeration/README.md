# Bind Shell Enumeration

## Service Information

- **Service:** Bind Shell
- **Port:** TCP/1524
- **Target:** Metasploitable 2
- **Detected service:** Metasploitable root shell

## 1. Service Discovery

We identified TCP/1524 as an open service using Nmap.

**Command used**

```bash
nmap -p 1524 -sV 192.168.56.20
```

Nmap identified:

```text
1524/tcp open bindshell Metasploitable root shell
```

[View Nmap bind shell detection evidence](../evidence/09_BINDSHELL01_nmap_bindshell_detection.png)

The service fingerprint immediately indicated that TCP/1524 was not a conventional application service but a network-accessible shell.

## 2. Direct Connection with Netcat

Because Nmap identified the service as a bind shell, we tested whether the TCP service accepted a direct connection.

**Command used**

```bash
nc 192.168.56.20 1524
```

The connection succeeded and presented an interactive shell:

```text
root@metasploitable:/#
```

[View Netcat bind shell connection evidence](../evidence/09_BINDSHELL02_ncat_bindshell_connection.png)

This is important because the finding was not based solely on Nmap's service fingerprint.

The service was manually validated by establishing a direct TCP connection and obtaining an interactive shell.

## 3. Privilege Verification

Once connected, we verified the privileges of the session.

**Command used**

```bash
whoami
```

The result was:

```text
root
```

This confirmed that the shell was operating with root privileges.

## 4. Target Identity Verification

We then confirmed the hostname of the system to which the shell was connected.

**Command used**

```bash
hostname
```

The result was:

```text
metasploitable
```

This confirms that the interactive shell belonged to the intended target system.

## 5. Working Directory Verification

Finally, we checked the current working directory.

**Command used**

```bash
pwd
```

The result was:

```text
/
```

This confirms that the shell was operating directly within the target's root filesystem.

[View root privilege and target verification evidence](../evidence/09_BINDSHELL03_root_privilege_confirmation.png)

The third screenshot provides a compact validation of:

**Root privileges → target identity → filesystem context**

## Security Analysis

The assessment established the following sequence:

```text
TCP/1524 exposed
        ↓
Nmap identifies bind shell
        ↓
Direct Netcat connection succeeds
        ↓
Interactive shell obtained
        ↓
whoami → root
        ↓
hostname → metasploitable
        ↓
pwd → /
```

The important distinction is that this was not merely an open TCP port.

The service provided a directly accessible interactive shell on the target system.

Furthermore, the shell was operating with root privileges.

## Authentication Observation

The connection was established using:

```bash
nc 192.168.56.20 1524
```

No username or password was requested by the service during the connection.

This demonstrates that the bind shell did not provide conventional interactive authentication before presenting the shell.

This observation significantly increases the security impact of the exposed service.

## Enumeration Summary

The assessment confirmed that:

- TCP/1524 was exposed.
- Nmap identified the service as a bind shell.
- A direct TCP connection using Netcat succeeded.
- The service provided an interactive shell.
- The shell operated with root privileges.
- The hostname was `metasploitable`.
- The working directory was `/`.

The evidence chain is:

**Service identification → direct connection → privilege verification → target verification**

## Scope and Limitations

The assessment was conducted against the intentionally vulnerable Metasploitable 2 laboratory environment.

Testing was limited to service identification, direct connection and privilege verification.

No exploitation framework was used.

No persistence, privilege escalation or destructive modification was performed.

The service was validated through direct interaction with the exposed TCP endpoint.

## Evidence

The assessment uses three screenshots:

```text
09_BINDSHELL01_nmap_bindshell_detection.png
09_BINDSHELL02_ncat_bindshell_connection.png
09_BINDSHELL03_root_privilege_confirmation.png
```

The evidence represents three distinct stages:

1. **Nmap** — identifies the bind shell.
2. **Netcat** — demonstrates that the shell is directly accessible.
3. **whoami + hostname + pwd** — confirms root privileges and target context.

## Commands

### Service detection

```bash
nmap -p 1524 -sV 192.168.56.20
```

### Connect to the bind shell

```bash
nc 192.168.56.20 1524
```

### Verify privileges

```bash
whoami
```

### Verify target hostname

```bash
hostname
```

### Verify working directory

```bash
pwd
```

## Portfolio Conclusion

**"TCP/1524 exposed a bind shell that accepted direct network connections without conventional authentication. A connection using Netcat provided an interactive shell, and `whoami` confirmed that the session was running as root. The hostname and working directory further confirmed that the shell belonged to the target system. The service was validated through direct connection and privilege verification without using an exploitation framework."**
