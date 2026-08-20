# Reconnaissance

## Objective

The objective of this phase was to identify the Metasploitable 2 host, determine its exposed network services, and establish an initial view of the target's attack surface.

The reconnaissance phase was performed from a Kali Linux virtual machine against the intentionally vulnerable Metasploitable 2 virtual machine within an isolated laboratory network.

## Lab Environment

* **Attacker:** Kali Linux
* **Target:** Metasploitable 2
* **Virtualization:** Oracle VirtualBox
* **Network:** Isolated internal laboratory network
* **Primary tool:** Nmap

## Methodology

The reconnaissance process was performed in two stages:

1. **Initial scan** — identification of exposed TCP ports and an initial overview of the target.
2. **Detailed scan** — service and version detection, operating system detection and Nmap's default NSE scripts.

The results of each stage are documented separately:

* [Initial Scan](./nmap/initial-scan/)
* [Detailed Scan](./nmap/detailed-scan/)

## Tools

### Nmap

Nmap was used to identify exposed services and collect information about the target system.

The initial service detection was performed with:

```bash
nmap -sV <TARGET-IP>
```

A more detailed scan was subsequently performed using:

```bash
nmap -sV -O -sC <TARGET-IP> -oA reconocimiento
```

Where:

* `-sV` enables service and version detection.
* `-O` attempts operating system detection.
* `-sC` runs Nmap's default NSE scripts.
* `-oA` saves the scan results in multiple output formats.

## Initial Reconnaissance

The initial reconnaissance identified a large number of exposed network services on the Metasploitable 2 system.

The results provided the first overview of the target's attack surface and allowed the subsequent analysis to be focused on the individual services discovered.

The complete output of the initial scan is documented in:

[Initial Nmap Scan](./nmap/initial-scan/)

## Detailed Reconnaissance

A second, more comprehensive Nmap scan was performed to obtain additional information about the exposed services, their versions and the underlying operating system.

The scan identified services including FTP, SSH, Telnet, SMTP, DNS, HTTP, SMB, MySQL, PostgreSQL, VNC, IRC and Apache Tomcat.

The detailed results are documented in:

[Detailed Nmap Scan](./nmap/detailed-scan/)

## Results

The reconnaissance phase established the initial attack surface of the Metasploitable 2 system and provided the information required for the subsequent service-by-service analysis.

The identified services were subsequently investigated individually as part of the service enumeration and vulnerability assessment phases.


