# Authorized Security Testing

## Purpose

Kali Linux is used to generate controlled security events against the Ubuntu Test Server.

```text
Kali Linux:         192.168.56.122
Ubuntu Test Server: 192.168.56.123
ELK Server:         192.168.56.121
```

Testing is restricted to the isolated VirtualBox Host-Only network.

## Rules of Engagement

- Only test lab-owned systems.
- Keep testing inside `192.168.56.0/24`.
- Do not target external systems.
- Do not perform destructive testing.
- Limit repeated authentication attempts.
- Record the test date and expected result.
- Stop the test if system stability is affected.

## Test Scenario 1: Connectivity

From Kali Linux:

```bash
ping -c 4 192.168.56.123
```

Expected result:

```text
Ubuntu Test Server responds to ICMP.
```

## Test Scenario 2: SSH Port Validation

```bash
nc -vz 192.168.56.123 22
```

Expected result:

```text
TCP/22 is reachable.
```

## Test Scenario 3: Successful SSH Login

Use an authorized account:

```bash
ssh <AUTHORIZED_USER>@192.168.56.123
```

After the login, search in Kibana:

```text
host.ip : "192.168.56.123" and message : "Accepted"
```

## Test Scenario 4: Failed SSH Login

Perform one or two failed login attempts using a lab account.

Search in Kibana:

```text
host.ip : "192.168.56.123" and message : "Failed password"
```

Do not perform excessive attempts.

## Test Scenario 5: Sudo Activity

On the Ubuntu Test Server:

```bash
sudo whoami
```

Search in Kibana:

```text
host.name : "ubuntu-test-server" and message : "sudo"
```

## Test Scenario 6: Controlled Port Discovery

A limited scan may be performed against the authorized Ubuntu Test Server:

```bash
nmap -sT -Pn -p 22,80,443 192.168.56.123
```

This scan is restricted to three selected ports and the authorized lab target.

## Test Result Template

```text
Test ID:
Test name:
Date:
Source:
Destination:
Expected log:
Observed log:
Kibana query:
Result:
Evidence:
Notes:
```

## Validation Checklist

- [ ] Testing remained within the lab network
- [ ] Target authorization was confirmed
- [ ] Ubuntu logs captured the activity
- [ ] Filebeat forwarded the event
- [ ] Elasticsearch indexed the event
- [ ] Kibana displayed the event
- [ ] Evidence was documented
- [ ] No credentials were exposed
