# RPC Security Findings

## Overview

The RPC assessment identified several security-relevant characteristics on the Metasploitable 2 host.

The target exposed the RPC portmapper on TCP/111 and UDP/111, and RPC enumeration revealed multiple registered services, including NFS, mountd, nlockmgr and status.

The assessment was limited to service identification and enumeration. No RPC exploitation was performed.

The primary findings are:

1. **RPC Portmapper Exposure**
2. **Multiple RPC Services Exposed**
3. **NFS Service Exposure**

---

## Finding 01 — RPC Portmapper Exposure

### Description

The target exposed the RPC portmapper service on TCP/111.

Nmap identified:

```text
111/tcp open rpcbind 2 (RPC #100000)
```

The RPC portmapper was subsequently queried using `rpcinfo`, confirming that multiple RPC programs were registered on the host.

### Evidence

The RPC portmapper was identified through Nmap:

[View Nmap RPC portmapper evidence](../evidence/07_RPC01_nmap_rpc_portmapper.png)

The registered RPC services were enumerated using `rpcinfo`:

[View RPC enumeration evidence](../evidence/07_RPC02_rpcinfo_enumeration.png)

### Security Impact

The RPC portmapper provides information about RPC services registered on the host.

If exposed to untrusted networks, this information can assist an attacker during reconnaissance by revealing additional services and dynamically assigned RPC ports.

The exposure of port 111 does not by itself demonstrate a direct vulnerability.

### Severity

**Medium**

The severity reflects the additional reconnaissance information available through an exposed RPC service and the presence of multiple RPC-dependent services.

### Recommendation

- Restrict RPC access to trusted networks and authorized hosts.
- Block unnecessary external access to TCP/111 and UDP/111.
- Disable RPC services that are not required.
- Review registered RPC programs regularly.
- Use network segmentation where appropriate.

---

## Finding 02 — Multiple RPC Services Exposed

### Description

RPC enumeration identified multiple services registered with the portmapper.

The following programs were observed:

| Program | Service | Versions observed |
|---|---|---|
| `100000` | portmapper | 2 |
| `100024` | status | 1 |
| `100003` | nfs | 2, 3, 4 |
| `100021` | nlockmgr | 1, 3, 4 |
| `100005` | mountd | 1, 2, 3 |

These services were registered over both TCP and UDP.

### Evidence

The complete RPC program enumeration is documented in:

[View complete rpcinfo evidence](../evidence/07_RPC02_rpcinfo_enumeration.png)

### Security Impact

Exposing multiple RPC-dependent services increases the network attack surface of the host.

Each registered service may provide additional functionality or disclose information that could be useful during reconnaissance.

The presence of a service does not, by itself, demonstrate that the service is vulnerable.

### Severity

**Medium**

The finding reflects the exposure of multiple RPC services and the resulting increase in attack surface.

### Recommendation

- Disable unnecessary RPC programs.
- Restrict RPC-dependent services to authorized hosts and networks.
- Review firewall rules controlling access to RPC services.
- Regularly audit registered RPC programs.
- Avoid exposing RPC services directly to untrusted networks.

---

## Finding 03 — NFS Service Exposure

### Description

RPC enumeration identified NFS as program `100003`.

The target registered NFS versions:

```text
2
3
4
```

Additional RPC services associated with NFS were also identified:

```text
100005  mountd
100021  nlockmgr
```

The discovery of these services indicated that NFS required a dedicated security assessment.

### Evidence

The NFS and related RPC services were identified through:

[View RPC enumeration evidence](../evidence/07_RPC02_rpcinfo_enumeration.png)

### Security Impact

NFS provides network-based filesystem access.

If NFS exports are incorrectly configured, unauthorized clients may potentially access files or directories that should not be exposed.

Older NFS configurations may also rely on weaker authentication or authorization models.

However, the RPC assessment alone does **not** demonstrate that an NFS export is accessible or improperly configured.

A separate NFS assessment is therefore required before classifying a specific NFS configuration as vulnerable.

### Severity

**Medium**

The severity reflects the exposure of a network filesystem service and the need to verify its export and access-control configuration.

### Recommendation

- Review all NFS exports.
- Restrict exports to authorized clients and trusted networks.
- Avoid exporting sensitive directories unnecessarily.
- Apply appropriate filesystem permissions.
- Review NFS access-control configuration.
- Disable NFS versions and functionality that are not required.
- Restrict `mountd` and related RPC services through firewall rules.
- Monitor access to network filesystem services.

---

## Assessment Methodology

The RPC assessment followed a simple enumeration workflow:

```text
RPC portmapper discovery
          ↓
RPC program enumeration
          ↓
Identification of registered services
          ↓
Identification of NFS
          ↓
Dedicated NFS assessment
```

This approach allowed the assessment to identify additional network services without attempting exploitation.

The RPC findings therefore serve both as security observations and as the basis for the subsequent NFS assessment.

---

## Evidence Handling

The RPC findings are supported by two screenshots:

```text
07_RPC01_nmap_rpc_portmapper.png
07_RPC02_rpcinfo_enumeration.png
```

The Nmap screenshot establishes that the RPC portmapper is exposed.

The `rpcinfo` screenshot provides the primary evidence for the RPC services registered on the target.

No screenshots or claims relating to exploitation are included because no RPC exploitation was performed.

---

## Overall Assessment

The target exposed RPC services that provided visibility into several network services running on the host.

The assessment confirmed:

- RPC portmapper exposure on TCP/111.
- Registration of RPC services on both TCP and UDP.
- Presence of `status`.
- Presence of NFS versions 2, 3 and 4.
- Presence of `mountd`.
- Presence of `nlockmgr`.
- A clear relationship between RPC enumeration and the subsequent NFS assessment.

The most significant observation was the exposure of NFS and its associated RPC services.

The RPC assessment did not demonstrate a direct compromise or a specific exploitable RPC vulnerability.

The appropriate next step was therefore to investigate the NFS configuration separately.

## Portfolio Conclusion

**"The target exposed the RPC portmapper on TCP/UDP 111. RPC enumeration identified multiple registered services, including NFS, mountd, nlockmgr and status. NFS versions 2, 3 and 4 were identified, providing a clear direction for the subsequent NFS security assessment. No RPC exploitation was performed."**
