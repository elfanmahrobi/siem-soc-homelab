# Network Topology

## Overview

The SIEM and SOC home lab is deployed using Oracle VirtualBox and consists of three virtual machines connected through an isolated Host-Only network.

The lab uses the following private network:

```text
192.168.56.0/24
```

The environment separates centralized security monitoring, monitored infrastructure, and authorized security testing into different virtual machines.

The current lab consists of:

- One Ubuntu server running the Elastic Stack
- One Ubuntu server acting as the monitored endpoint
- One Kali Linux machine used for authorized attack simulation

> [!IMPORTANT]
> Kali Linux is used only for controlled security testing against systems inside the isolated home lab. All testing must be performed only against systems owned by the lab operator or systems where explicit authorization has been granted.

---

## Network Architecture

```text
                        VirtualBox Host-Only Network
                              192.168.56.0/24

┌────────────────────────────┐
│ Kali Linux                 │
│ Authorized Attack Simulator│
│                            │
│ IP: 192.168.56.122         │
│                            │
│ Functions:                 │
│ - Security testing         │
│ - SSH login simulation     │
│ - Port scanning simulation │
│ - Detection validation     │
└──────────────┬─────────────┘
               │
               │ Authorized security testing
               │
               ▼
┌────────────────────────────┐
│ Ubuntu Test Server         │
│ Monitored Linux Endpoint   │
│                            │
│ IP: 192.168.56.123         │
│                            │
│ Components:                │
│ - Filebeat                 │
│ - Linux system logs        │
│ - Authentication logs      │
│ - SSH service              │
│ - Sudo activity            │
└──────────────┬─────────────┘
               │
               │ Filebeat / Beats Protocol
               │ TCP/5044
               │
               ▼
┌────────────────────────────┐
│ Ubuntu ELK Server          │
│ Centralized SIEM Platform  │
│                            │
│ IP: 192.168.56.121         │
│                            │
│ Components:                │
│ - Elasticsearch            │
│ - Logstash                 │
│ - Kibana                   │
│ - Filebeat                 │
└──────────────┬─────────────┘
               │
               ▼
┌────────────────────────────┐
│ Security Monitoring        │
│                            │
│ - Kibana Discover          │
│ - Security dashboards      │
│ - Detection rules          │
│ - Event investigation      │
└────────────────────────────┘
```

---

## Virtual Machine Inventory

| Hostname | IP Address | Operating System | Role |
|---|---|---|---|
| ELK Server | `192.168.56.121` | Ubuntu Server | Centralized SIEM platform |
| Kali Linux | `192.168.56.122` | Kali Linux | Authorized attack simulation |
| Ubuntu Test Server | `192.168.56.123` | Ubuntu Server | Monitored Linux endpoint |

---

## Network Configuration

| Parameter | Configuration |
|---|---|
| Virtualization platform | Oracle VirtualBox |
| Network type | Host-Only Adapter |
| Network address | `192.168.56.0/24` |
| Subnet mask | `255.255.255.0` |
| Usable host range | `192.168.56.1` to `192.168.56.254` |
| Broadcast address | `192.168.56.255` |
| Centralized SIEM server | `192.168.56.121` |
| Authorized attack simulator | `192.168.56.122` |
| Monitored Ubuntu server | `192.168.56.123` |

The Host-Only network allows communication between the host computer and the virtual machines while keeping the security testing environment separated from the primary physical network.

If internet access is required for package installation and system updates, each virtual machine can use an additional NAT adapter.

### Recommended VirtualBox Adapters

#### Adapter 1: NAT

Used for:

- Operating system updates
- Package installation
- Elastic repository access
- Internet connectivity

Example interface:

```text
10.0.2.0/24
```

#### Adapter 2: Host-Only Adapter

Used for:

- Communication between lab virtual machines
- Filebeat log forwarding
- Kibana administration
- Authorized security testing
- Security event monitoring

Lab network:

```text
192.168.56.0/24
```

> [!NOTE]
> All SIEM communication and security testing should use the Host-Only IP addresses in the `192.168.56.0/24` network rather than the NAT interface addresses.

---

## Server Roles

### Ubuntu ELK Server

```text
IP address: 192.168.56.121
```

The Ubuntu ELK Server provides centralized security event collection, processing, storage, and visualization.

Installed components:

- Elasticsearch
- Logstash
- Kibana
- Filebeat for optional local server monitoring

Primary responsibilities:

- Receive events from Filebeat
- Parse and normalize Linux logs
- Store security events in Elasticsearch
- Provide dashboards and event analysis through Kibana
- Support security investigation
- Support future security detection rules
- Support future SOAR integration

### Ubuntu Test Server

```text
IP address: 192.168.56.123
```

The Ubuntu Test Server represents a monitored Linux endpoint.

Installed components:

- Filebeat
- OpenSSH Server
- Linux logging services

Monitored log sources:

- System logs
- Authentication logs
- SSH login events
- Failed SSH login events
- Sudo activity
- User and group changes
- Service events
- Custom syslog test events

Filebeat reads Linux log files and sends the events to Logstash on:

```text
192.168.56.121:5044
```

### Kali Linux

```text
IP address: 192.168.56.122
```

Kali Linux is used as an authorized security testing and attack simulation machine.

Primary responsibilities:

- Generate controlled security events
- Simulate SSH login attempts
- Perform authorized network discovery
- Validate SIEM visibility
- Validate detection rules
- Support blue-team investigation exercises

Kali Linux does not send operational logs to the ELK Server during the initial implementation phase. Kali Linux primarily generates test traffic and authentication events against the Ubuntu Test Server.

---

## Log Collection Flow

```text
Ubuntu Test Server
192.168.56.123
        |
        | Filebeat
        | Beats Protocol
        | TCP/5044
        |
        ▼
Logstash
192.168.56.121
        |
        | Parsing
        | Filtering
        | Normalization
        |
        ▼
Elasticsearch
192.168.56.121
        |
        | Indexing and storage
        |
        ▼
Kibana
192.168.56.121:5601
        |
        ▼
Security Monitoring and Investigation
```

---

## Security Testing Flow

```text
Kali Linux
192.168.56.122
        |
        | Controlled and authorized
        | security simulation
        |
        ▼
Ubuntu Test Server
192.168.56.123
        |
        | Authentication and system events
        |
        ▼
Filebeat
        |
        | TCP/5044
        |
        ▼
Logstash
192.168.56.121
        |
        ▼
Elasticsearch
        |
        ▼
Kibana
        |
        ▼
Detection Validation and Investigation
```

---

## Communication Matrix

| Source | Destination | Port | Protocol | Purpose |
|---|---|---:|---|---|
| Ubuntu Test Server | ELK Server | `5044` | TCP | Filebeat events sent to Logstash |
| Administrator Workstation | ELK Server | `5601` | TCP | Kibana web interface |
| Administrator Workstation | ELK Server | `22` | TCP | SSH administration |
| Administrator Workstation | Ubuntu Test Server | `22` | TCP | SSH administration |
| Logstash | Elasticsearch | `9200` | HTTPS | Event indexing |
| Kibana | Elasticsearch | `9200` | HTTPS | Elasticsearch API communication |
| Kali Linux | Ubuntu Test Server | `22` | TCP | Authorized SSH event simulation |
| Kali Linux | Ubuntu Test Server | Lab-dependent | TCP/UDP | Authorized detection testing |
| Filebeat on ELK Server | Logstash | `5044` | TCP | Optional local ELK server log collection |

---

## Required Connectivity

### Ubuntu Test Server to ELK Server

The Ubuntu Test Server must be able to reach the Logstash Beats input:

```bash
nc -vz 192.168.56.121 5044
```

Expected result:

```text
Connection to 192.168.56.121 5044 port [tcp/*] succeeded!
```

### Administrator to Kibana

Kibana should be accessible from the host computer through:

```text
http://192.168.56.121:5601
```

If HTTPS is enabled:

```text
https://192.168.56.121:5601
```

### Kali Linux to Ubuntu Test Server

Validate connectivity from Kali Linux:

```bash
ping -c 4 192.168.56.123
```

Validate authorized SSH connectivity:

```bash
nc -vz 192.168.56.123 22
```

### Ubuntu Test Server to ELK Server

Validate the network path:

```bash
ping -c 4 192.168.56.121
```

Validate Logstash connectivity:

```bash
nc -vz 192.168.56.121 5044
```

---

## Logstash Beats Input

Logstash listens for Filebeat events on TCP port `5044`.

Example input configuration:

```logstash
input {
  beats {
    host => "0.0.0.0"
    port => 5044
  }
}
```

The listening port can be validated on the ELK Server using:

```bash
sudo ss -lntp | grep ':5044'
```

Expected result:

```text
LISTEN 0 4096 0.0.0.0:5044 0.0.0.0:*
```

---

## Filebeat Output

Filebeat on the Ubuntu Test Server sends events to Logstash running on the ELK Server.

Example Filebeat output configuration:

```yaml
output.logstash:
  hosts:
    - "192.168.56.121:5044"
```

The connection can be validated from the Ubuntu Test Server using:

```bash
sudo filebeat test output -e
```

---

## Firewall Requirements

If UFW is active on the ELK Server, TCP port `5044` should only be allowed from the monitored Ubuntu server.

Run on the ELK Server:

```bash
sudo ufw allow from 192.168.56.123 to any port 5044 proto tcp
```

Allow Kibana access from the VirtualBox Host-Only network:

```bash
sudo ufw allow from 192.168.56.0/24 to any port 5601 proto tcp
```

Allow SSH administration from the Host-Only network if required:

```bash
sudo ufw allow from 192.168.56.0/24 to any port 22 proto tcp
```

Check the firewall rules:

```bash
sudo ufw status numbered
```

> [!WARNING]
> Elasticsearch TCP port `9200` should not be exposed to Kali Linux or other unnecessary systems. Log sources should send events through Logstash on TCP port `5044`.

---

## Authorized Testing Scenarios

The current network topology supports the following controlled validation scenarios:

### Authentication Monitoring

- Successful SSH login
- Failed SSH login
- Invalid username attempt
- Sudo command execution
- Repeated authentication failure within a controlled threshold

### Linux Monitoring

- Custom syslog event
- User account creation
- User account modification
- Group membership change
- Service start and stop
- Authentication log event
- System log event

### Network Monitoring

- Authorized host discovery
- Controlled TCP connection attempts
- Controlled port-scan validation
- Connection to closed ports
- SSH service access validation

> [!CAUTION]
> Testing must remain inside the isolated `192.168.56.0/24` VirtualBox network. Do not perform scanning, authentication testing, or any other security testing against external systems without explicit authorization.

---

## Example Validation Scenario

### Step 1: Generate an event

Generate a custom event on the Ubuntu Test Server:

```bash
logger "SOC-HOMELAB TEST: Linux log forwarding validation"
```

### Step 2: Verify the local event

```bash
grep "SOC-HOMELAB" /var/log/syslog
```

If the system uses journald:

```bash
journalctl | grep "SOC-HOMELAB"
```

### Step 3: Verify Filebeat

```bash
sudo systemctl status filebeat
```

```bash
sudo journalctl -u filebeat -n 100 --no-pager
```

### Step 4: Verify Logstash

Run on the ELK Server:

```bash
sudo journalctl -u logstash -n 100 --no-pager
```

### Step 5: Verify Elasticsearch

```bash
curl --cacert /etc/elasticsearch/certs/http_ca.crt \
-u elastic \
"https://localhost:9200/_cat/indices?v"
```

### Step 6: Search in Kibana

Open Kibana:

```text
http://192.168.56.121:5601
```

Search in Kibana Discover:

```text
message : "SOC-HOMELAB"
```

Filter by the monitored server:

```text
host.ip : "192.168.56.123"
```

Depending on the Filebeat event structure, the server can also be identified using:

```text
host.name : "ubuntu-test-server"
```

---

## Security Considerations

The following security controls apply to the home lab:

- The security testing network uses a VirtualBox Host-Only adapter.
- Kali Linux testing is restricted to authorized lab targets.
- Filebeat sends events only to the designated ELK Server.
- Logstash TCP port `5044` should only accept authorized log sources.
- Elasticsearch TCP port `9200` should not be publicly accessible.
- Kibana access should be limited to the host or trusted lab network.
- SSH access should be restricted to trusted management systems.
- Elasticsearch credentials must not be stored in public repository files.
- Private keys and certificates must not be committed to GitHub.
- Screenshots should be reviewed before being uploaded.
- Production systems must not be connected to this attack simulation network.
- NAT should only be used for updates and software installation.

---

## Current Implementation Status

| Component | IP Address | Status |
|---|---|---|
| Ubuntu ELK Server | `192.168.56.121` | Implemented |
| Elasticsearch | `192.168.56.121:9200` | Running |
| Logstash | `192.168.56.121:5044` | Running |
| Kibana | `192.168.56.121:5601` | Running |
| Kali Linux | `192.168.56.122` | Available for authorized testing |
| Ubuntu Test Server | `192.168.56.123` | Available |
| Filebeat on Test Server | `192.168.56.123` | In configuration |
| Linux log forwarding | Test Server to ELK Server | In validation |
| Linux security dashboard | Kibana | Planned |
| Linux detection rules | Kibana | Planned |

---

## Next Steps

- [x] Configure the VirtualBox Host-Only network
- [x] Assign the ELK Server IP address
- [x] Assign the Kali Linux IP address
- [x] Assign the Ubuntu Test Server IP address
- [x] Deploy Elasticsearch
- [x] Deploy Logstash
- [x] Deploy Kibana
- [ ] Confirm Logstash is listening on TCP/5044
- [ ] Install Filebeat on the Ubuntu Test Server
- [ ] Enable the Filebeat System module
- [ ] Configure Filebeat output to `192.168.56.121:5044`
- [ ] Validate Filebeat connectivity
- [ ] Send test system logs
- [ ] Send test authentication logs
- [ ] Create the Kibana data view
- [ ] Validate logs in Kibana Discover
- [ ] Perform authorized detection testing from Kali Linux
- [ ] Create Linux security detection rules
- [ ] Create Linux monitoring dashboards
