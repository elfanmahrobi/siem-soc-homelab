# Project Overview

## Introduction

This project documents the development of a practical Security Information and Event Management and Security Operations Center home lab.

The environment is designed to provide hands-on experience in:

- Centralized log collection
- Security event processing
- Event normalization
- Log indexing
- Security monitoring
- Threat detection
- Incident investigation
- Detection validation
- Security automation

The current implementation focuses on the Elastic Stack SIEM foundation.

## Project Scope

The first phase covers:

- Elasticsearch deployment
- Logstash deployment
- Kibana deployment
- Filebeat installation
- Ubuntu system log collection
- Ubuntu authentication log collection
- End-to-end event validation
- Linux security monitoring

Future development phases will include:

- Detection engineering
- MITRE ATT&CK mapping
- Security dashboards
- Windows log collection
- Firewall log collection
- Shuffle SOAR integration
- TheHive integration
- MISP threat intelligence integration

## Lab Components

### Ubuntu ELK Server

The ELK Server provides centralized event processing, storage, and visualization.

```text
IP address: 192.168.56.121
```

Installed components:

- Elasticsearch
- Logstash
- Kibana
- Filebeat for optional local monitoring

### Ubuntu Test Server

The Ubuntu Test Server represents a monitored Linux endpoint.

```text
IP address: 192.168.56.123
```

Installed components:

- Filebeat
- OpenSSH Server
- Linux logging services

Collected events include:

- System logs
- Authentication logs
- SSH login events
- Failed login events
- Sudo activity
- Service events
- User and group changes

### Kali Linux

Kali Linux is used to generate controlled and authorized security events.

```text
IP address: 192.168.56.122
```

Example validation scenarios include:

- SSH connection testing
- Failed authentication testing
- Authorized network discovery
- Controlled port scanning
- Detection rule validation

## Project Objectives

1. Build a centralized Linux security monitoring platform.
2. Understand the complete SIEM data pipeline.
3. Collect logs from a remote Ubuntu server.
4. Process events using Logstash.
5. Store events in Elasticsearch.
6. Analyze events using Kibana.
7. Generate controlled security events.
8. Develop security detection rules.
9. Document investigation procedures.
10. Map detections to MITRE ATT&CK.
11. Develop a practical cybersecurity portfolio.
12. Expand the lab into a complete SOC ecosystem.

## Current Data Flow

```text
Ubuntu Test Server
        |
        | Filebeat
        | TCP/5044
        v
Logstash
        |
        | Parse, filter, and normalize
        v
Elasticsearch
        |
        | Index and store
        v
Kibana
        |
        | Search and visualize
        v
Security Monitoring
```

## Implementation Phases

### Phase 1: Infrastructure

- [x] Create VirtualBox virtual machines
- [x] Configure the Host-Only network
- [x] Assign static lab IP addresses
- [x] Verify communication between virtual machines

### Phase 2: ELK Stack

- [x] Install Elasticsearch
- [x] Install Logstash
- [x] Install Kibana
- [x] Start and enable ELK services
- [x] Validate service status

### Phase 3: Linux Log Onboarding

- [ ] Install Filebeat on the Ubuntu Test Server
- [ ] Enable the Filebeat System module
- [ ] Configure Filebeat output to Logstash
- [ ] Configure Logstash Beats input
- [ ] Validate TCP/5044 connectivity
- [ ] Validate event indexing

### Phase 4: Monitoring

- [ ] Create a Kibana data view
- [ ] Validate system logs
- [ ] Validate authentication logs
- [ ] Build a Linux monitoring dashboard
- [ ] Create detection rules

### Phase 5: Detection Validation

- [ ] Generate controlled events from Kali Linux
- [ ] Detect SSH authentication failures
- [ ] Detect suspicious sudo activity
- [ ] Detect user account changes
- [ ] Document investigation evidence

### Phase 6: SOC Expansion

- [ ] Integrate Shuffle
- [ ] Integrate TheHive
- [ ] Integrate MISP
- [ ] Configure notification workflows
- [ ] Develop automated response playbooks

## Limitations

The current environment has the following limitations:

- The environment is intended for lab use only.
- High availability is not configured.
- Elasticsearch currently runs as a single node.
- Log source coverage is limited to Linux servers.
- Security automation is not yet implemented.
- Detection rules are still under development.
- Performance depends on available host resources.

## Expected Outcome

The completed first phase should demonstrate the following pipeline:

```text
Linux event generated
        |
        v
Filebeat collects event
        |
        v
Logstash receives event
        |
        v
Elasticsearch indexes event
        |
        v
Kibana displays event
        |
        v
Security analyst investigates event
```
