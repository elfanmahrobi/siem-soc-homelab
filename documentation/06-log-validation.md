# End-to-End Log Validation

## Objective

This procedure validates the complete log collection pipeline:

```text
Ubuntu Test Server
        |
        v
Filebeat
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

## Step 1: Validate the Test Event Locally

On the Ubuntu Test Server:

```bash
logger "SOC-HOMELAB TEST: End-to-end pipeline validation"
```

Verify:

```bash
journalctl --since "5 minutes ago" | grep "SOC-HOMELAB"
```

Or:

```bash
grep "SOC-HOMELAB" /var/log/syslog
```

## Step 2: Validate Filebeat

```bash
sudo systemctl is-active filebeat
```

Test output:

```bash
sudo filebeat test output -e
```

Review logs:

```bash
sudo journalctl -u filebeat -n 100 --no-pager
```

Check for errors:

```bash
sudo journalctl -u filebeat -n 200 --no-pager -o cat | \
grep -iE "error|failed|refused|timeout"
```

## Step 3: Validate Logstash

On the ELK Server:

```bash
sudo systemctl is-active logstash
```

Check port:

```bash
sudo ss -lntp | grep ':5044'
```

Review logs:

```bash
sudo journalctl -u logstash -n 100 --no-pager
```

Check errors:

```bash
sudo journalctl -u logstash -n 300 --no-pager -o cat | \
grep -iE "error|warn|failed|bulk|mapping|timeout|exception"
```

## Step 4: Validate Elasticsearch

Check cluster health:

```bash
curl --cacert /etc/elasticsearch/certs/http_ca.crt \
-u elastic \
"https://localhost:9200/_cluster/health?pretty"
```

List Linux indices:

```bash
curl --cacert /etc/elasticsearch/certs/http_ca.crt \
-u elastic \
"https://localhost:9200/_cat/indices/linux-logs-*?v"
```

Search for the test event:

```bash
curl --cacert /etc/elasticsearch/certs/http_ca.crt \
-u elastic \
-H "Content-Type: application/json" \
"https://localhost:9200/linux-logs-*/_search?pretty" \
-d '{
  "size": 10,
  "sort": [
    {
      "@timestamp": {
        "order": "desc"
      }
    }
  ],
  "query": {
    "match_phrase": {
      "message": "SOC-HOMELAB"
    }
  }
}'
```

## Step 5: Create a Kibana Data View

Open:

```text
http://192.168.56.121:5601
```

Go to:

```text
Stack Management → Data Views
```

Create:

```text
Name: Linux Logs
Index pattern: linux-logs-*
Timestamp field: @timestamp
```

## Step 6: Search in Kibana Discover

Open:

```text
Analytics → Discover
```

Select:

```text
Linux Logs
```

Search:

```text
message : "SOC-HOMELAB"
```

Useful filters:

```text
host.name : "ubuntu-test-server"
```

```text
event.module : "system"
```

```text
event.dataset : "system.auth"
```

```text
event.dataset : "system.syslog"
```

## Expected Result

The event generated on `192.168.56.123` should appear in Kibana with fields such as:

- `@timestamp`
- `host.name`
- `host.ip`
- `agent.name`
- `agent.type`
- `event.module`
- `event.dataset`
- `message`

## Validation Result Template

```text
Test date:
Test event:
Source host:
Source IP:
Filebeat status:
Logstash status:
Elasticsearch index:
Kibana data view:
Validation result:
Notes:
```

## Evidence Checklist

- [ ] Local event screenshot
- [ ] Filebeat validation screenshot
- [ ] Logstash status screenshot
- [ ] Elasticsearch index screenshot
- [ ] Kibana Discover screenshot
- [ ] Sensitive information reviewed before upload
