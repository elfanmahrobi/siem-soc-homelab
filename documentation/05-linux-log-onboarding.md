# Ubuntu Linux Log Onboarding

## Objective

This procedure collects system and authentication events from:

```text
Ubuntu Test Server: 192.168.56.123
```

The events are sent to:

```text
Logstash: 192.168.56.121:5044
```

## Data Flow

```text
Ubuntu Test Server
        |
        | Filebeat
        | TCP/5044
        v
Logstash
        |
        v
Elasticsearch
        |
        v
Kibana
```

## Step 1: Verify Connectivity

Run on the Ubuntu Test Server:

```bash
ping -c 4 192.168.56.121
```

Check Logstash:

```bash
nc -vz 192.168.56.121 5044
```

## Step 2: Configure Logstash

On the ELK Server, create:

```text
/etc/logstash/conf.d/ubuntu-filebeat.conf
```

Example:

```logstash
input {
  beats {
    host => "0.0.0.0"
    port => 5044
  }
}

filter {
  if [agent][type] == "filebeat" {
    mutate {
      add_field => {
        "lab_environment" => "siem-soc-homelab"
        "log_source_type" => "linux"
      }
    }
  }
}

output {
  elasticsearch {
    hosts => ["https://localhost:9200"]
    user => "elastic"
    password => "${ELASTIC_PASSWORD}"
    ssl_enabled => true
    ssl_certificate_authorities => [
      "/etc/elasticsearch/certs/http_ca.crt"
    ]
    index => "linux-logs-%{+YYYY.MM.dd}"
  }
}
```

Do not store the actual Elasticsearch password in the pipeline file.

Test the configuration:

```bash
sudo -u logstash /usr/share/logstash/bin/logstash \
--path.settings /etc/logstash \
--config.test_and_exit
```

Restart Logstash:

```bash
sudo systemctl restart logstash
```

Confirm port `5044`:

```bash
sudo ss -lntp | grep ':5044'
```

## Step 3: Install Filebeat

Install Filebeat on the Ubuntu Test Server using the package version compatible with the Elastic Stack deployment.

Confirm installation:

```bash
filebeat version
```

## Step 4: Enable the System Module

```bash
sudo filebeat modules enable system
```

Confirm:

```bash
sudo filebeat modules list
```

## Step 5: Configure the System Module

Edit:

```text
/etc/filebeat/modules.d/system.yml
```

Example:

```yaml
- module: system
  syslog:
    enabled: true
    var.paths:
      - /var/log/syslog*

  auth:
    enabled: true
    var.paths:
      - /var/log/auth.log*
```

Check whether the log files exist:

```bash
ls -l /var/log/syslog /var/log/auth.log
```

## Step 6: Configure Filebeat Output

Edit:

```text
/etc/filebeat/filebeat.yml
```

Disable Elasticsearch output:

```yaml
# output.elasticsearch:
#   hosts:
#     - "localhost:9200"
```

Enable Logstash output:

```yaml
output.logstash:
  hosts:
    - "192.168.56.121:5044"
```

Add server identification:

```yaml
name: "ubuntu-test-server"

tags:
  - "linux"
  - "ubuntu"
  - "soc-homelab"

fields:
  environment: "lab"
  log_source: "ubuntu-test-server"
```

## Step 7: Test Filebeat

Test configuration:

```bash
sudo filebeat test config -e
```

Expected result:

```text
Config OK
```

Test Logstash output:

```bash
sudo filebeat test output -e
```

Expected result should include:

```text
connection... OK
```

## Step 8: Start Filebeat

```bash
sudo systemctl enable --now filebeat
```

Check status:

```bash
sudo systemctl --no-pager --full status filebeat
```

Review recent logs:

```bash
sudo journalctl -u filebeat -n 100 --no-pager
```

## Step 9: Generate a Test Event

```bash
logger "SOC-HOMELAB TEST: Ubuntu log forwarding validation"
```

Generate an authentication facility event:

```bash
logger -p auth.notice \
"SOC-HOMELAB AUTH TEST: Authentication monitoring validation"
```

Verify locally:

```bash
grep "SOC-HOMELAB" /var/log/syslog /var/log/auth.log 2>/dev/null
```

For journald:

```bash
journalctl --since "5 minutes ago" | grep "SOC-HOMELAB"
```

## Completion Checklist

- [ ] Ubuntu Test Server can reach the ELK Server
- [ ] TCP/5044 is reachable
- [ ] Logstash configuration is valid
- [ ] Logstash is listening on TCP/5044
- [ ] Filebeat is installed
- [ ] Filebeat System module is enabled
- [ ] Filebeat configuration is valid
- [ ] Filebeat output test is successful
- [ ] Filebeat service is active
- [ ] Test event is generated
- [ ] Test event is received by Elasticsearch
- [ ] Test event is visible in Kibana
