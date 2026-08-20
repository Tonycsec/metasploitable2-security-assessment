# Detailed Nmap Scan

## Objective

The objective of the detailed scan was to obtain additional information about the services identified during the initial reconnaissance, including service versions, operating system information and additional details exposed through Nmap's default NSE scripts.

The scan was performed from Kali Linux against the Metasploitable 2 host at `192.168.56.20`.

## Command

```bash
nmap -sV -O -sC 192.168.56.20
```

The scan used the following options:

* `-sV` — service and version detection.
* `-O` — operating system detection.
* `-sC` — execution of Nmap's default NSE scripts.

## Service Detection

The detailed scan confirmed the services identified during the initial reconnaissance and provided additional information about their configuration and exposed functionality.

|     Port | Service     | Version / Information         |
| -------: | ----------- | ----------------------------- |
|   21/tcp | FTP         | vsftpd 2.3.4                  |
|   22/tcp | SSH         | OpenSSH 4.7p1 Debian 8ubuntu1 |
|   23/tcp | Telnet      | Linux telnetd                 |
|   25/tcp | SMTP        | Postfix smtpd                 |
|   53/tcp | DNS         | ISC BIND 9.4.2                |
|   80/tcp | HTTP        | Apache httpd 2.2.8 (Ubuntu)   |
|  111/tcp | RPC         | rpcbind 2                     |
|  139/tcp | NetBIOS-SSN | Samba smbd 3.X - 4.X          |
|  445/tcp | NetBIOS-SSN | Samba smbd 3.0.20-Debian      |
|  512/tcp | exec        | netkit-rsh rexecd             |
|  513/tcp | login       | OpenBSD or Solaris rlogind    |
|  514/tcp | shell       | Netkit rshd                   |
| 1099/tcp | Java RMI    | GNU Classpath grmiregistry    |
| 1524/tcp | Bindshell   | Metasploitable root shell     |
| 2049/tcp | NFS         | NFS 2-4                       |
| 2121/tcp | FTP         | ProFTPD 1.3.1                 |
| 3306/tcp | MySQL       | MySQL 5.0.51a-3ubuntu5        |
| 5432/tcp | PostgreSQL  | PostgreSQL 8.3.0 - 8.3.7      |
| 5900/tcp | VNC         | VNC protocol 3.3              |
| 6000/tcp | X11         | Access denied                 |
| 6667/tcp | IRC         | UnrealIRCd                    |
| 8009/tcp | AJP13       | Apache JServ Protocol v1.3    |
| 8180/tcp | HTTP        | Apache Tomcat 5.5             |

## NSE Script Findings

The default NSE scripts provided additional information beyond simple service detection.

### FTP

Nmap confirmed that anonymous FTP access was permitted:

```text
ftp-anon: Anonymous FTP login allowed
```

The FTP server also reported that both control and data connections were transmitted in plaintext.

### SMTP

The SMTP service exposed several supported commands, including:

* `VRFY`
* `ETRN`
* `STARTTLS`

Nmap also identified support for **SSLv2**, an obsolete and insecure protocol.

### RPC and NFS

The RPC enumeration identified several RPC services associated with NFS and related services, including:

* `rpcbind`
* `nfs`
* `mountd`
* `nlockmgr`
* `status`

This indicated that NFS-related functionality was exposed by the target.

### SMB / Samba

The SMB enumeration identified:

* **Samba 3.0.20-Debian**
* Computer name: `metasploitable`
* Domain: `localdomain`
* FQDN: `metasploitable.localdomain`

Nmap also reported that **SMB message signing was disabled**.

### MySQL

The MySQL service was identified as:

```text
MySQL 5.0.51a-3ubuntu5
```

Nmap's `mysql-info` script provided additional information about the server's protocol and capabilities.

### PostgreSQL

The PostgreSQL service was identified as:

```text
PostgreSQL DB 8.3.0 - 8.3.7
```

Nmap also identified SSL certificate information associated with the service.

### VNC

The VNC service reported:

```text
Protocol version: 3.3
Security type: VNC Authentication
```

### Apache Tomcat

The service running on port `8180/tcp` was identified as:

```text
Apache Tomcat/Coyote JSP engine 1.1
Apache Tomcat/5.5
```

### AJP13

The service exposed on port `8009/tcp` was identified as Apache JServ Protocol (AJP) version 1.3.

The default NSE script was unable to obtain a valid response to its `OPTION` request.

## Operating System Detection

Nmap identified the target as a general-purpose Linux system.

The detected information included:

* **Operating system family:** Linux
* **Kernel:** Linux 2.6.x
* **Estimated kernel range:** Linux 2.6.9 - 2.6.33
* **CPE:** `cpe:/o:linux:linux_kernel:2.6`
* **Network distance:** 1 hop

The MAC address was identified as belonging to an Oracle VirtualBox virtual network interface.

## Host and Network Information

The scan identified the following host information:

* **IP address:** `192.168.56.20`
* **Hostname:** `metasploitable.localdomain`
* **Computer name:** `metasploitable`
* **Domain:** `localdomain`
* **Network distance:** 1 hop

## Initial Security Observations

The detailed scan confirmed that the target exposed a large number of network services, several of which were running legacy software or presented potentially insecure configurations.

Notable observations included:

* Anonymous FTP access was enabled.
* Telnet and other legacy remote access services were exposed.
* Multiple legacy software versions were detected.
* SSLv2 support was identified on the SMTP service.
* SMB message signing was disabled.
* NFS-related RPC services were exposed.
* A VNC service was accessible remotely.
* Apache Tomcat 5.5 was exposed through HTTP.
* AJP13 was exposed on port `8009/tcp`.
* Multiple database services were directly accessible over the network.

These observations were used to determine which services required further enumeration during the subsequent service analysis phase.

## Conclusion

The detailed Nmap scan provided a substantially richer view of the Metasploitable 2 attack surface than the initial service discovery scan.

The information obtained during this phase was used as the basis for the subsequent **service-by-service enumeration and vulnerability assessment**.

[Back to Reconnaissance](../../README.md)

