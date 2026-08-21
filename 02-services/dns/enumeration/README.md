# DNS Enumeration

## Service Information

- **Service:** DNS
- **Ports:** TCP/53 and UDP/53
- **Software:** BIND DNS
- **Version:** BIND 9.4.2
- **Target:** Metasploitable 2

## Service Discovery

The DNS service was identified during the reconnaissance phase using Nmap.

The initial service-specific scan was performed with:

```bash
nmap 192.168.56.20 -p 53 -sV
```

Nmap identified the following service:

```text
53/tcp open domain ISC BIND 9.4.2
```

A more explicit TCP/UDP check was also performed:

```bash
nmap 192.168.56.20 -p T:53,U:53 -sV
```

The scan confirmed the DNS service on TCP/53.

[View Nmap DNS service detection evidence](../evidence/05_DNS01_nmap_dns_detection.png)

The detected BIND version indicated that the target was running a legacy DNS implementation.

## DNS Query

A direct DNS query was performed against the target's DNS server using `dig`.

The command used was:

```bash
dig @192.168.56.20 metasploitable.localdomain
```

The query was sent directly to the DNS service at `192.168.56.20` rather than relying on the assessment host's configured resolver.

The server returned an authoritative response for the requested domain:

```text
;; ->>HEADER<<- opcode: QUERY, status: SERVFAIL, id: 50970
;; flags: qr rd; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1
```

The query was processed directly by the target DNS service over UDP/53.

[View DNS query evidence](../evidence/05_DNS02_dns_query.png)

This confirmed that the DNS service was reachable and capable of processing direct DNS queries.

## DNS Zone Transfer Attempt

The next step was to test whether the DNS server permitted an AXFR zone transfer.

The following command was executed:

```bash
dig @192.168.56.20 axfr metasploitable.localdomain
```

The server rejected the transfer request:

```text
; Transfer failed.
```

[View AXFR denied evidence](../evidence/05_DNS03_axfr_denied.png)

An unrestricted AXFR transfer could potentially disclose the complete contents of a DNS zone, including hostnames, addresses and other infrastructure records.

However, the observed result was that the transfer was denied.

Therefore, no unrestricted DNS zone-transfer vulnerability was confirmed during this assessment.

## DNS Security Analysis

The AXFR test was performed as part of the DNS security assessment because improperly configured zone transfers can provide significant information to an attacker.

A successful AXFR could potentially expose records such as:

```text
server1
mail
www
ftp
db
internal
```

along with their associated addresses.

In this assessment, the AXFR request was rejected. This represents a positive security control rather than a confirmed vulnerability.

The assessment therefore distinguishes between:

**Test performed**

DNS zone transfer requested using AXFR.

**Observed result**

Transfer rejected.

**Security conclusion**

No unrestricted zone-transfer vulnerability was confirmed.

## Enumeration Summary

The DNS enumeration established the following characteristics:

- DNS was exposed on the target.
- BIND 9.4.2 was identified through service detection.
- TCP/53 was confirmed as an accessible DNS service.
- A direct DNS query was successfully sent to the target DNS server.
- The target responded to the DNS query with a `SERVFAIL` status.
- An AXFR zone-transfer request was explicitly tested.
- The AXFR request was rejected by the server.
- No unrestricted zone-transfer vulnerability was confirmed.

These observations were used as the basis for the subsequent DNS security findings.

## Scope and Limitations

The assessment was conducted against the intentionally vulnerable Metasploitable 2 laboratory environment.

The DNS service was identified and tested using Nmap and `dig`. The assessment focused on service exposure, DNS query behaviour and zone-transfer configuration without attempting exploitation or unauthorized access to DNS data.
