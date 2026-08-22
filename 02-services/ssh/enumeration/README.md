# SSH Enumeration

## Service Information

- **Service:** SSH
- **Port:** TCP/22
- **Software:** OpenSSH
- **Version:** 4.7p1 Debian 8ubuntu1
- **Protocol:** SSH version 2.0

## Service Discovery

The SSH service was identified during the reconnaissance phase using Nmap.

The service-specific scan was performed with:

```bash
nmap 192.168.56.20 -p 22 -sV
```

Nmap identified the following service:

```text
22/tcp open ssh OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
```

![View Nmap SSH service detection evidence](../evidence/02_SSH01_nmap_ssh_detection.png)

The detected OpenSSH version indicated that the target was running a legacy SSH implementation and warranted further configuration and authentication assessment.

## SSH Algorithm Enumeration

An initial SSH connection attempt was performed using the default configuration of the modern SSH client.

The connection was rejected because the server offered legacy host-key algorithms that were not accepted by the client's current security policy.

The server reported:

```text
Unable to negotiate with 192.168.56.20 port 22:
no matching host key type found.
Their offer: ssh-rsa,ssh-dss
```

![View legacy SSH algorithm error evidence](../evidence/02_SSH02_ssh_algorithm_error.png)

This demonstrated that the SSH server exposed legacy host-key algorithms, including:

- `ssh-rsa`
- `ssh-dss`

The observation provided an additional configuration finding beyond the identification of the obsolete OpenSSH version.

## Authenticated SSH Connection

After identifying the legacy host-key requirement, an SSH connection was established by explicitly enabling compatibility with `ssh-rsa`.

The connection was performed using:

```bash
ssh -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedAlgorithms=+ssh-rsa msfadmin@192.168.56.20
```

The connection successfully established an interactive shell using the `msfadmin` account.

![View authenticated SSH session evidence](../evidence/02_SSH03_ssh_authenticated_session.png)

This demonstrated that the SSH service was not only exposed remotely, but also permitted interactive authentication using a local system account.

No exploitation or privilege escalation was performed.

## System Information

After establishing the SSH session, system information was collected using:

```bash
uname -a
```

The system reported:

```text
Linux metasploitable 2.6.24-16-server #1 SMP Thu Apr 10 13:58:00 UTC 2008 i686 GNU/Linux
```

![View SSH system information evidence](../evidence/02_SSH04_ssh_system_information.png)

This identified the target as a legacy Linux environment running a 2.6.24 kernel.

The operating system information provides additional context for the security assessment because the SSH daemon is operating within an obsolete system environment.

## Enumeration Summary

The SSH enumeration established the following characteristics:

- OpenSSH 4.7p1 was exposed on TCP/22.
- The server offered legacy host-key algorithms including `ssh-rsa` and `ssh-dss`.
- A modern SSH client initially rejected the connection because of the legacy algorithm configuration.
- Compatibility with `ssh-rsa` allowed an authenticated SSH session to be established.
- The `msfadmin` local account provided interactive remote access.
- The underlying system was identified as a legacy Linux environment running kernel 2.6.24.

These observations were used as the basis for the subsequent SSH security findings.

## Scope and Limitations

The assessment was conducted against the intentionally vulnerable Metasploitable 2 laboratory environment.

The SSH service was enumerated and its configuration and authentication behaviour were assessed without attempting exploitation, privilege escalation or persistence.
