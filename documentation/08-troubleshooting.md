# Troubleshooting Guide

## Filebeat Cannot Connect to Logstash

### Symptoms

```text
connection refused
```

```text
dial tcp 192.168.56.121:5044
```

### Checks

From the Ubuntu Test Server:

```bash
ping -c 4 192.168.56.121
```

```bash
nc -vz 192.168.56.121 5044
```

```bash
sudo filebeat test output -e
```

On the ELK Server:

```bash
sudo systemctl status logstash
```

```bash
sudo ss -lntp | grep ':5044'
```

Potential causes:

- Logstash is stopped
- Beats input is missing
- Logstash configuration is invalid
- UFW blocks TCP/5044
- Wrong ELK Server address
- VirtualBox adapters are on different networks

---

## Filebeat Configuration Error

Test configuration:

```bash
sudo filebeat test config -e
```

Review indentation in:

```text
/etc/filebeat/filebeat.yml
```

YAML uses spaces for indentation. Tabs should not be used.

Check duplicate outputs:

```bash
grep -nE "^output\.elasticsearch:|^output\.logstash:" \
/etc/filebeat/filebeat.yml
```

Only one output should be enabled.

---

## Filebeat Service Is Running but No Events Appear

Check enabled modules:

```bash
sudo filebeat modules list
```

Check local log files:

```bash
ls -l /var/log/syslog /var/log/auth.log
```

Generate a test event:

```bash
logger "SOC-HOMELAB TEST: Troubleshooting validation"
```

Check Filebeat logs:

```bash
sudo journalctl -u filebeat -n 200 --no-pager
```

---

## Logstash Configuration Error

Test Logstash:

```bash
sudo -u logstash /usr/share/logstash/bin/logstash \
--path.settings /etc/logstash \
--config.test_and_exit
```

List pipeline files:

```bash
find /etc/logstash/conf.d \
-maxdepth 1 \
-type f \
-name "*.conf" \
-print
```

Review Logstash logs:

```bash
sudo journalctl -u logstash -n 200 --no-pager
```

---

## Logstash Bulk Request Errors

Search for detailed errors:

```bash
sudo journalctl -u logstash -n 500 --no-pager -o cat | \
grep -iE "bulk|mapping|parsing|security_exception|timeout|rejected"
```

Common causes:

- Elasticsearch temporarily unavailable
- Invalid Elasticsearch credentials
- TLS certificate issue
- Mapping conflict
- Elasticsearch memory pressure
- Rejected indexing request

Check Elasticsearch health:

```bash
curl --cacert /etc/elasticsearch/certs/http_ca.crt \
-u elastic \
"https://localhost:9200/_cluster/health?pretty"
```

Check indices:

```bash
curl --cacert /etc/elasticsearch/certs/http_ca.crt \
-u elastic \
"https://localhost:9200/_cat/indices?v&s=index"
```

---

## Elasticsearch Is Not Found

Incorrect service name:

```bash
sudo systemctl status elastic
```

Correct service name:

```bash
sudo systemctl status elasticsearch
```

---

## Elasticsearch Authentication Error

Possible output:

```text
401 Unauthorized
```

Verify the username and password.

Use interactive password entry:

```bash
curl --cacert /etc/elasticsearch/certs/http_ca.crt \
-u elastic \
https://localhost:9200/
```

Do not place the password directly in screenshots or GitHub files.

---

## Kibana Is Not Accessible

Check the service:

```bash
sudo systemctl status kibana
```

Check port `5601`:

```bash
sudo ss -lntp | grep ':5601'
```

Check local endpoint:

```bash
curl -I http://localhost:5601
```

Check firewall:

```bash
sudo ufw status numbered
```

Check Kibana logs:

```bash
sudo journalctl -u kibana -n 200 --no-pager
```

---

## Kibana Data View Does Not Find an Index

List indices:

```bash
curl --cacert /etc/elasticsearch/certs/http_ca.crt \
-u elastic \
"https://localhost:9200/_cat/indices?v&s=index"
```

Confirm the data view matches the actual index.

Example:

```text
Index: linux-logs-2026.10.25
Data view: linux-logs-*
```

Check the selected time range in Kibana Discover.

Recommended temporary range:

```text
Last 24 hours
```

---

## High Memory Usage

Check memory:

```bash
free -h
```

Check swap:

```bash
swapon --show
```

Check top processes:

```bash
ps aux --sort=-%mem | head -n 15
```

Check disk space:

```bash
df -h
```

The ELK Stack requires significant memory. Avoid running unnecessary virtual machines during initial troubleshooting.

---

## VirtualBox Connectivity Issue

Check interfaces:

```bash
ip -br address
```

Check routes:

```bash
ip route
```

Verify all virtual machines have the Host-Only interface in:

```text
192.168.56.0/24
```

Expected addressing:

```text
ELK Server:         192.168.56.121
Kali Linux:         192.168.56.122
Ubuntu Test Server: 192.168.56.123
```

Verify the VirtualBox adapter is attached to the same Host-Only network.

---

## Useful Diagnostic Collection

### ELK Server

```bash
systemctl is-active elasticsearch logstash kibana
sudo ss -lntp | grep -E ':5044|:5601|:9200|:9600'
free -h
df -h
sudo journalctl -u logstash -n 100 --no-pager
```

### Ubuntu Test Server

```bash
ip -br address
ping -c 4 192.168.56.121
nc -vz 192.168.56.121 5044
sudo filebeat test config -e
sudo filebeat test output -e
sudo systemctl status filebeat
```

## Security Reminder

Before sharing troubleshooting output:

- Remove passwords
- Remove API keys
- Remove tokens
- Remove private keys
- Review usernames
- Review internal organization information
- Never upload Logstash keystore files
