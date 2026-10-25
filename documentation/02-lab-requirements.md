# Lab Requirements

## Virtualization Platform

The home lab uses Oracle VirtualBox.

Recommended configuration:

- Oracle VirtualBox
- VirtualBox Host-Only Adapter
- NAT adapter for package installation
- Host computer with hardware virtualization enabled

## Host System Requirements

Recommended minimum host resources:

| Resource | Minimum | Recommended |
|---|---:|---:|
| CPU | 4 cores | 8 cores |
| Memory | 12 GB | 16 GB or more |
| Free storage | 80 GB | 120 GB or more |
| Network | Host-Only and NAT | Host-Only and NAT |

Running Elasticsearch, Logstash, Kibana, Ubuntu Server, and Kali Linux simultaneously can consume significant memory.

## Virtual Machine Requirements

### Ubuntu ELK Server

Recommended configuration:

| Resource | Configuration |
|---|---:|
| CPU | 2 to 4 vCPU |
| Memory | 6 to 8 GB |
| Storage | 50 GB |
| Adapter 1 | NAT |
| Adapter 2 | Host-Only |
| Host-Only IP | `192.168.56.121` |

Installed software:

- Ubuntu Server
- Elasticsearch
- Logstash
- Kibana
- Filebeat
- OpenSSH Server
- curl
- netcat
- UFW

### Kali Linux

Recommended configuration:

| Resource | Configuration |
|---|---:|
| CPU | 2 vCPU |
| Memory | 2 to 4 GB |
| Storage | 30 GB |
| Adapter 1 | NAT |
| Adapter 2 | Host-Only |
| Host-Only IP | `192.168.56.122` |

Purpose:

- Authorized attack simulation
- Network connectivity testing
- Authentication event generation
- Detection validation

### Ubuntu Test Server

Recommended configuration:

| Resource | Configuration |
|---|---:|
| CPU | 1 to 2 vCPU |
| Memory | 2 GB |
| Storage | 20 GB |
| Adapter 1 | NAT |
| Adapter 2 | Host-Only |
| Host-Only IP | `192.168.56.123` |

Installed software:

- Ubuntu Server
- Filebeat
- OpenSSH Server
- rsyslog or journald
- netcat
- curl

## Network Requirements

The lab uses:

```text
Network: 192.168.56.0/24
Subnet mask: 255.255.255.0
Broadcast: 192.168.56.255
```

IP allocation:

```text
ELK Server:         192.168.56.121
Kali Linux:         192.168.56.122
Ubuntu Test Server: 192.168.56.123
```

## Required Ports

| Port | Protocol | Purpose |
|---:|---|---|
| `22` | TCP | SSH administration and testing |
| `5044` | TCP | Filebeat to Logstash |
| `5601` | TCP | Kibana web interface |
| `9200` | HTTPS | Elasticsearch API |
| `9600` | TCP | Logstash monitoring API |

## Required Connectivity

The Ubuntu Test Server must reach:

```text
192.168.56.121:5044
```

The host workstation must reach:

```text
192.168.56.121:5601
```

Kali Linux must reach the authorized target:

```text
192.168.56.123
```

## Verification Commands

Check interface addresses:

```bash
ip -br address
```

Check routing:

```bash
ip route
```

Check connectivity to the ELK Server:

```bash
ping -c 4 192.168.56.121
```

Check the Logstash port:

```bash
nc -vz 192.168.56.121 5044
```

Check the Ubuntu Test Server from Kali Linux:

```bash
ping -c 4 192.168.56.123
```

Check SSH:

```bash
nc -vz 192.168.56.123 22
```

## Security Requirements

- Do not expose Elasticsearch directly to untrusted networks.
- Restrict TCP/5044 to authorized log sources.
- Restrict Kibana access to the host or trusted network.
- Use strong passwords.
- Do not commit credentials to GitHub.
- Do not commit certificates or private keys.
- Keep attack simulations inside the lab network.
- Use placeholders in public configuration examples.
