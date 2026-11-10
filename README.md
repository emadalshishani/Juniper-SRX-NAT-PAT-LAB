# Juniper SRX Source NAT / PAT Lab

## Overview

This lab demonstrates interface-based Source NAT with Port Address Translation (PAT) on a Juniper SRX firewall.

The HQ SRX translates traffic originating from three internal IPv4 subnets to the IP address configured on its egress interface, `ge-0/0/1.0`. The source port is also translated when required, allowing multiple internal sessions to share the interface address.

## Topology

The diagram below shows the lab topology, including the HQ and Branch environments, upstream ISP routers, and the route-based IPsec connection.

![Source NAT/PAT Lab Topology](docs/images/Source%20NAT-PAT%20Topology.svg)

## Lab Scope

- Configure a Source NAT rule set on the HQ SRX.
- Match traffic from three internal subnets.
- Match any destination IPv4 address (`0.0.0.0/0`).
- Use interface-based source NAT.
- Verify outbound connectivity from an end host.
- Inspect SRX flow sessions to confirm source-address and source-port translation.

## NAT Design

| Setting | Value |
|---|---|
| SRX device | HQ SRX |
| Rule set | `hq-source-nat` |
| From zone | `internal` |
| To zone | `branch` |
| Internal subnet 1 | `192.168.10.0/29` |
| Internal subnet 2 | `192.168.20.0/29` |
| Internal subnet 3 | `192.168.30.0/29` |
| Destination match | `0.0.0.0/0` |
| NAT method | Interface-based Source NAT |
| Egress interface | `ge-0/0/1.0` |
| Translated interface address observed | `10.10.30.1` |

> The translated address above is taken from the observed SRX flow-session output.

## Configuration

The configuration applied on the HQ SRX:

```text
set security nat source rule-set hq-source-nat from zone internal
set security nat source rule-set hq-source-nat to zone branch
set security nat source rule-set hq-source-nat rule internal-pat match source-address 192.168.10.0/29
set security nat source rule-set hq-source-nat rule internal-pat match source-address 192.168.20.0/29
set security nat source rule-set hq-source-nat rule internal-pat match source-address 192.168.30.0/29
set security nat source rule-set hq-source-nat rule internal-pat match destination-address 0.0.0.0/0
set security nat source rule-set hq-source-nat rule internal-pat then source-nat interface
```

The standalone configuration is available in [configurations/HQ-SRX-source-nat.conf](configurations/HQ-SRX-source-nat.conf).

## Verification

### 1. End-host connectivity test

A Linux end host in the internal network successfully pinged `8.8.8.8`.

Command:

```bash
ping 8.8.8.8 -c 1000
```

Observed result before the test was manually interrupted:

```text
3 packets transmitted, 3 received, 0% packet loss
rtt min/avg/max/mdev = 53.211/53.744/54.198/0.406 ms
```

The command was stopped manually after three successful replies; this is not a completed 1,000-packet test.

### 2. SRX flow-session verification

The following sessions were observed on the HQ SRX:

```text
Session ID: 1431, Policy name: internal-branch/6, Timeout: 534, Session State: Valid
  In: 192.168.30.2/35772 --> 34.107.243.93/443;tcp, Conn Tag: 0x0, If: ge-0/0/0.0, Pkts: 8, Bytes: 2510,
  Out: 34.107.243.93/443 --> 10.10.30.1/21464;tcp, Conn Tag: 0x0, If: ge-0/0/1.0, Pkts: 16, Bytes: 2298,

Session ID: 1787, Policy name: internal-branch/6, Timeout: 520, Session State: Valid
  In: 192.168.30.2/43090 --> 18.205.187.118/443;tcp, Conn Tag: 0x0, If: ge-0/0/0.0, Pkts: 24, Bytes: 4264,
  Out: 18.205.187.118/443 --> 10.10.30.1/3494;tcp, Conn Tag: 0x0, If: ge-0/0/1.0, Pkts: 27, Bytes: 2646,

Session ID: 1803, Policy name: internal-branch/6, Timeout: 480, Session State: Valid
  In: 192.168.30.2/35082 --> 52.0.218.127/443;tcp, Conn Tag: 0x0, If: ge-0/0/0.0, Pkts: 35, Bytes: 6732,
  Out: 52.0.218.127/443 --> 10.10.30.1/6233;tcp, Conn Tag: 0x0, If: ge-0/0/1.0, Pkts: 33, Bytes: 7751,
```

These sessions show:

- Original source IP: `192.168.30.2`
- Translated source IP: `10.10.30.1`
- Original source ports: `35772`, `43090`, and `35082`
- Translated source ports: `21464`, `3494`, and `6233`
- Outgoing interface: `ge-0/0/1.0`

The change in both source IP address and source port demonstrates interface-based Source NAT with PAT for the observed TCP sessions.

### 3. Browser test

The Linux end host was also used to browse the HPE website successfully, providing an application-level connectivity check.

![Linux end host browsing the Internet](docs/images/linux%20device%20browsing%20internet.svg)

## Repository Structure

```text
Juniper-SRX-NAT-PAT-LAB/
├── README.md
├── configurations/
│   └── HQ-SRX-source-nat.conf
├── docs/
│   └── images/
│       ├── Source NAT-PAT Topology.svg
│       └── linux device browsing internet.svg
└── verification/
    └── verification.md
```

## Notes

- This repository documents the configuration and test evidence provided for this lab.
- The Reverse DNS lookup output is intentionally excluded because it does not verify Source NAT/PAT operation.
