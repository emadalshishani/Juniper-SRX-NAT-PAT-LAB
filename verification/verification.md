# Verification Results

## End-host connectivity

Command executed on the Linux end host:

```bash
ping 8.8.8.8 -c 1000
```

Observed output:

```text
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 53.211/53.744/54.198/0.406 ms
```

The test was manually interrupted after three successful replies.

## HQ SRX flow sessions

Command:

```text
run show security flow session
```

Observed sessions:

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

## NAT interpretation

For the three observed TCP sessions:

| Field | Original | Translated |
|---|---|---|
| Source IP | 192.168.30.2 | 10.10.30.1 |
| Source port (session 1431) | 35772 | 21464 |
| Source port (session 1787) | 43090 | 3494 |
| Source port (session 1803) | 35082 | 6233 |

The outgoing interface shown by the SRX is `ge-0/0/1.0`. The source address and source port changes confirm Source NAT with PAT for these sessions.

## Browser test

Browser access was tested from the end host. Screenshot evidence is pending upload.
