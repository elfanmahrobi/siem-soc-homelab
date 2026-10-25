<h1 align="center">SIEM and SOC Home Lab</h1>

<p align="center">
  <img src="https://img.shields.io/badge/SIEM-Implemented-2ea44f?style=for-the-badge" alt="SIEM Implemented">
  <img src="https://img.shields.io/badge/SOC-In%20Development-f59e0b?style=for-the-badge" alt="SOC In Development">
  <img src="https://img.shields.io/badge/SOAR-Planned-8b949e?style=for-the-badge" alt="SOAR Planned">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Elasticsearch-005571?style=flat-square&logo=elasticsearch&logoColor=white" alt="Elasticsearch">
  <img src="https://img.shields.io/badge/Logstash-005571?style=flat-square&logo=logstash&logoColor=white" alt="Logstash">
  <img src="https://img.shields.io/badge/Kibana-005571?style=flat-square&logo=kibana&logoColor=white" alt="Kibana">
  <img src="https://img.shields.io/badge/Linux-FCC624?style=flat-square&logo=linux&logoColor=black" alt="Linux">
</p>

## Overview

This project documents the development of a practical Security Information and Event Management (SIEM) and Security Operations Center (SOC) home lab.

The current implementation focuses on building a centralized security monitoring platform using the Elastic Stack. Logs from endpoints, servers, applications, firewalls, and other infrastructure components can be collected, processed, indexed, visualized, and analyzed from a centralized platform.

The long-term objective is to expand the environment into an integrated SOC ecosystem that includes detection engineering, incident response, security orchestration, case management, threat intelligence, and automated notification capabilities.

> **Current implementation status:** The SIEM platform based on the Elastic Stack has been implemented. SOAR, incident response, and threat intelligence integrations remain part of the project roadmap.

---

## Architecture

The following diagram represents the target architecture of the SIEM and SOC home lab.

./architecture/soc-architecture-diagram.png

> **Implementation note:** The Elastic Stack SIEM components have been implemented. Shuffle, TheHive, MISP, automated notifications, and other SOAR capabilities remain part of the project roadmap.

The architecture is divided into two primary areas:

### SIEM

The SIEM layer is responsible for:

- Collecting logs from multiple security and infrastructure sources
- Parsing and normalizing security events
- Storing and indexing centralized log data
- Creating security dashboards and visualizations
- Supporting threat detection and investigation
- Providing information to security analysts

### SOAR

The SOAR layer is the planned development phase responsible for:

- Security workflow orchestration
- Automated incident response
- Case handling and investigation
- Threat intelligence enrichment
- Real-time security notification
- Analyst collaboration

The SOAR components shown in the architecture are part of the target design and have not yet been fully implemented.

---

## Current Implementation

The following components are currently included in the SIEM implementation:

| Component | Function | Status |
|---|---|---|
| Elasticsearch | Centralized log storage and indexing | Implemented |
| Logstash | Log ingestion, parsing, and normalization | Implemented |
| Kibana | Dashboard, visualization, and log analysis | Implemented |
| Windows Log Source | Windows security and system event collection | In Progress |
| Linux Log Source | Linux authentication and system log collection | In Progress |
| Firewall Log Source | Network security event collection | In Progress |
| Detection Rules | Identification of suspicious security events | In Development |
| SOAR Integration | Security workflow automation | Planned |
| TheHive Integration | Incident and case management | Planned |
| MISP Integration | Threat intelligence management | Planned |
| Automated Notification | Alert notification using email or messaging | Planned |

---

## Project Objectives

The main objectives of this project are:

1. Build a centralized platform for security log monitoring.
2. Understand the end-to-end SIEM data pipeline.
3. Practice log collection, parsing, enrichment, and normalization.
4. Develop security monitoring dashboards in Kibana.
5. Create detection use cases based on common security threats.
6. Simulate security events and validate detection capabilities.
7. Document investigation and incident response procedures.
8. Integrate automated SOC workflows in future development phases.
9. Map detection rules to the MITRE ATT&CK framework.
10. Build a practical cybersecurity portfolio project.

---

## SIEM Data Flow

The current SIEM data flow follows this general process:

```text
Log Sources
    |
    v
Agents / Log Forwarders
    |
    v
Logstash
    |
    v
Elasticsearch
    |
    v
Kibana
    |
    v
Security Monitoring and Investigation
