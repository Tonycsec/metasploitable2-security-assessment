# Initial Nmap Scan

## Objective

The objective of the initial Nmap scan was to identify the TCP services exposed by the Metasploitable 2 host and obtain their service and version information.

The scan was performed from the Kali Linux attacker machine against the target at `192.168.56.20`.

## Command

```bash
nmap -sV 192.168.56.20
```

The `-sV` option enables service and version detection, allowing Nmap to identify the services running on the discovered open ports.

## Results

The scan identified **22 open TCP ports** on the target system:

|     Port | Service     | Version                             |
| -------: | ----------- | ----------------------------------- |
|   21/tcp | FTP         | vsftpd 2.3.4                        |
|   22/tcp | SSH         | OpenSSH 4.7p1 Debian 8ubuntu1       |
|   23/tcp | Telnet      | Linux telnetd                       |
|   25/tcp | SMTP        | Postfix smtpd                       |
|   53/tcp | DNS         | ISC BIND 9.4.2                      |
|   80/tcp | HTTP        | Apache httpd 2.2.8                  |
|  111/tcp | RPC         | rpcbind 2                           |
|  139/tcp | NetBIOS-SSN | Samba smbd 3.X - 4.X                |
|  445/tcp | NetBIOS-SSN | Samba smbd 3.X - 4.X                |
|  512/tcp | exec        | netkit-rsh rexecd                   |
|  513/tcp | login       | OpenBSD or Solaris rlogind          |
|  514/tcp | shell       | Netkit rshd                         |
| 1099/tcp | Java RMI    | GNU Classpath grmiregistry          |
| 1524/tcp | Bindshell   | Metasploitable root shell           |
| 2049/tcp | NFS         | RPC #100003                         |
| 2121/tcp | FTP         | ProFTPD 1.3.1                       |
| 3306/tcp | MySQL       | MySQL 5.0.51a-3ubuntu5              |
| 5432/tcp | PostgreSQL  | PostgreSQL DB 8.3.0 - 8.3.7         |
| 5900/tcp | VNC         | VNC protocol 3.3                    |
| 6000/tcp | X11         | Access denied                       |
| 6667/tcp | IRC         | UnrealIRCd                          |
| 8009/tcp | AJP13       | Apache JServ Protocol v1.3          |
| 8180/tcp | HTTP        | Apache Tomcat/Coyote JSP engine 1.1 |

## Initial Observations

The initial scan revealed a broad attack surface, with multiple network services exposed simultaneously.

Several services were running **legacy software versions**, including FTP, SSH, HTTP, DNS, database services and Apache Tomcat. The presence of multiple remote administration, file sharing, database and application services warranted further service-specific enumeration.

Particularly notable services included:

* FTP on ports `21` and `2121`
* SSH on port `22`
* Telnet on port `23`
* SMB/Samba on ports `139` and `445`
* NFS on port `2049`
* MySQL on port `3306`
* PostgreSQL on port `5432`
* VNC on port `5900`
* Apache Tomcat on port `8180`

These findings established the initial attack surface used to guide the subsequent service-by-service enumeration and vulnerability assessment.

## Host Information

Nmap reported the target as:

* **IP address:** `192.168.56.20`
* **MAC address:** `08:00:27:1F:24:93`
* **Vendor:** Oracle VirtualBox virtual NIC
* **Operating system family:** Unix / Linux

## Next Step

The initial scan provided a broad overview of the exposed services. A more detailed Nmap scan was subsequently performed using operating system detection and default NSE scripts.

The results are documented in:

[Detailed Nmap Scan](../detailed-scan/)

