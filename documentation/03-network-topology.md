# Network Topology

## Overview

The SIEM and SOC home lab is deployed using Oracle VirtualBox and consists of three virtual machines connected through an isolated Host-Only network.

```text
Network: 192.168.56.0/24
```

## Network Diagram

```text
                    VirtualBox Host-Only Network
                          192.168.56.0/24

          Authorized security testing
┌──────────────────────┐
│ Kali Linux           │
│ 192.168.56.122       │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Ubuntu Test Server   │
│ 192.168.56.123       │
│                      │
│ Filebeat             │
│ SSH                  │
│ Linux logs           │
└──────────┬───────────┘
           │ TCP/5044
           ▼
┌──────────────────────┐
│ Ubuntu ELK Server    │
│ 192.168.56.121       │
│                      │
│ Logstash             │
│ Elasticsearch        │
│ Kibana               │
└──────────────────────┘
```

## Virtual Machine Inventory

| Host | IP Address | Role |
|---|---|---|
| Ubuntu ELK Server | `192.168.56.121` | Centralized SIEM |
| Kali Linux | `192.168.56.122` | Authorized attack simulation |
| Ubuntu Test Server | `192.168.56.123` | Monitored Linux endpoint |

## VirtualBox Adapter Configuration

### Adapter 1: NAT

Used for:

- Operating system updates
- Package installation
- Elastic package repository access

### Adapter 2: Host-Only Adapter

Used for:

- SIEM communication
- Filebeat log forwarding
- Kibana administration
- Authorized security testing

All SIEM and security testing traffic uses:

```text
192.168.56.0/24
```

## Communication Matrix

| Source | Destination | Port | Purpose |
|---|---|---:|---|
| Ubuntu Test Server | ELK Server | `5044/TCP` | Filebeat log forwarding |
| Host Workstation | ELK Server | `5601/TCP` | Kibana access |
| Logstash | Elasticsearch | `9200/HTTPS` | Event indexing |
| Kibana | Elasticsearch | `9200/HTTPS` | Search and visualization |
| Kali Linux | Ubuntu Test Server | `22/TCP` | Authorized SSH testing |
| Host Workstation | All virtual machines | `22/TCP` | Administration |

## Log Flow

```text
Ubuntu Test Server
        |
        | Filebeat
        v
192.168.56.121:5044
        |
        v
Logstash
        |
        v
Elasticsearch
        |
        v
Kibana
```

## Security Testing Flow

```text
Kali Linux
192.168.56.122
        |
        v
Ubuntu Test Server
192.168.56.123
        |
        | Security events
        v
Filebeat
        |
        v
ELK Server
192.168.56.121
```

## Firewall Recommendations

Allow Filebeat only from the monitored server:

```bash
sudo ufw allow from 192.168.56.123 to any port 5044 proto tcp
```

Allow Kibana from the Host-Only network:

```bash
sudo ufw allow from 192.168.56.0/24 to any port 5601 proto tcp
```

Allow SSH from the Host-Only network if required:

```bash
sudo ufw allow from 192.168.56.0/24 to any port 22 proto tcp
```

Review rules:

```bash
sudo ufw status numbered
```

## Validation

Run from the Ubuntu Test Server:

```bash
ping -c 4 192.168.56.121
nc -vz 192.168.56.121 5044
```

Run from Kali Linux:

```bash
ping -c 4 192.168.56.123
nc -vz 192.168.56.123 22
```
