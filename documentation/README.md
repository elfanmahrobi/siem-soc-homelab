# SIEM and SOC Home Lab Documentation

This directory contains the technical documentation for the SIEM and SOC home lab.

The current environment uses three virtual machines running on Oracle VirtualBox:

- Ubuntu ELK Server
- Ubuntu monitored server
- Kali Linux security testing machine

## Documentation Index

| Document | Description |
|---|---|
| [Project Overview](./01-project-overview.md) | Project scope, objectives, and implementation status |
| [Lab Requirements](./02-lab-requirements.md) | Hardware, software, and network requirements |
| [Network Topology](./03-network-topology.md) | VirtualBox network design, IP addressing, and traffic flow |
| [ELK Stack Deployment](./04-elk-stack-deployment.md) | Elasticsearch, Logstash, and Kibana deployment documentation |
| [Linux Log Onboarding](./05-linux-log-onboarding.md) | Filebeat installation and Ubuntu log collection |
| [Log Validation](./06-log-validation.md) | End-to-end log forwarding and indexing validation |
| [Security Testing](./07-security-testing.md) | Authorized security event generation from Kali Linux |
| [Troubleshooting](./08-troubleshooting.md) | Common issues and troubleshooting commands |

## Current Architecture

```text
Kali Linux
192.168.56.122
        |
        | Authorized security simulation
        v
Ubuntu Test Server
192.168.56.123
        |
        | Filebeat / TCP 5044
        v
Ubuntu ELK Server
192.168.56.121
        |
        +--> Logstash
        +--> Elasticsearch
        +--> Kibana
```

## Current Status

| Component | Status |
|---|---|
| VirtualBox Host-Only Network | Implemented |
| Ubuntu ELK Server | Implemented |
| Elasticsearch | Running |
| Logstash | Running |
| Kibana | Running |
| Ubuntu Test Server | Available |
| Filebeat on Test Server | In validation |
| Linux log forwarding | In validation |
| Kali Linux | Available |
| Security testing | Planned |
| Detection rules | Planned |
| SOAR integration | Planned |

## Documentation Principles

All configuration examples in this repository:

- Use placeholders for credentials
- Exclude passwords and API keys
- Exclude private keys
- Exclude sensitive system information
- Represent lab configurations only
- Must not be used directly in production without review

## Security Notice

This project is intended for educational, defensive security, and portfolio purposes.

Security simulations must only be performed against systems owned by the lab operator or systems where explicit authorization has been granted.
