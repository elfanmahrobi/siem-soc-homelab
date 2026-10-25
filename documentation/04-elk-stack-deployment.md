# ELK Stack Deployment

## Overview

The Elastic Stack is deployed on the Ubuntu ELK Server:

```text
IP address: 192.168.56.121
```

The server runs:

- Elasticsearch
- Logstash
- Kibana
- Filebeat

## Component Responsibilities

### Elasticsearch

Elasticsearch stores and indexes security events.

### Logstash

Logstash receives events from Filebeat, processes the events, and sends them to Elasticsearch.

### Kibana

Kibana provides event searching, dashboards, visualization, and security investigation.

### Filebeat

Filebeat on the ELK Server can optionally collect local Linux events.

Remote Linux events are collected by Filebeat installed directly on the Ubuntu Test Server.

## Service Validation

Check all ELK services:

```bash
sudo systemctl is-active elasticsearch logstash kibana filebeat
```

Expected output:

```text
active
active
active
active
```

Check whether services start during boot:

```bash
sudo systemctl is-enabled elasticsearch logstash kibana filebeat
```

Check detailed status:

```bash
sudo systemctl --no-pager --full status elasticsearch
sudo systemctl --no-pager --full status logstash
sudo systemctl --no-pager --full status kibana
sudo systemctl --no-pager --full status filebeat
```

## Listening Ports

```bash
sudo ss -lntp | grep -E ':5044|:5601|:9200|:9600'
```

Expected services:

```text
5044 - Logstash Beats input
5601 - Kibana
9200 - Elasticsearch
9600 - Logstash monitoring API
```

## Elasticsearch Validation

Check the Elasticsearch endpoint:

```bash
curl --cacert /etc/elasticsearch/certs/http_ca.crt \
-u elastic \
https://localhost:9200/
```

Check cluster health:

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

> [!NOTE]
> A single-node Elasticsearch cluster may show a yellow status when replica shards cannot be assigned to another node.

## Logstash Validation

Test the Logstash configuration:

```bash
sudo -u logstash /usr/share/logstash/bin/logstash \
--path.settings /etc/logstash \
--config.test_and_exit
```

Expected result:

```text
Configuration OK
```

Check recent Logstash logs:

```bash
sudo journalctl -u logstash -n 100 --no-pager
```

Check only warnings and errors:

```bash
sudo journalctl -u logstash -n 300 --no-pager -o cat | \
grep -iE "error|warn|failed|bulk|mapping|timeout|exception"
```

## Kibana Validation

Check Kibana status:

```bash
curl -s http://localhost:5601/api/status | python3 -m json.tool
```

If HTTPS is enabled:

```bash
curl -sk https://localhost:5601/api/status | python3 -m json.tool
```

Access Kibana from the host workstation:

```text
http://192.168.56.121:5601
```

## Resource Monitoring

Check memory usage:

```bash
free -h
```

Check disk usage:

```bash
df -h
```

Check swap:

```bash
swapon --show
```

Check high-memory processes:

```bash
ps aux --sort=-%mem | head -n 15
```

## Service Management

Restart Elasticsearch:

```bash
sudo systemctl restart elasticsearch
```

Restart Logstash:

```bash
sudo systemctl restart logstash
```

Restart Kibana:

```bash
sudo systemctl restart kibana
```

Enable services during startup:

```bash
sudo systemctl enable elasticsearch logstash kibana
```

## Security Notes

Never commit the following files directly to GitHub:

```text
/etc/elasticsearch/elasticsearch.yml
/etc/elasticsearch/certs/
/etc/kibana/kibana.yml
/etc/logstash/logstash.keystore
```

Only sanitized example configurations should be stored in the public repository.
