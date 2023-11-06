# Monitoring with Grafana, InfluxDB, Telegraf

Docker implementation of monitoring server (TIG Stack).

Example instance (metrics.demo.jet.dev) could be found in [separate instance repository](https://github.com/jet-dev-team/metrics-demo-jet-dev).

## Quick start

- Make sure you have Ansible installed on your loca machine
- Clone the repository to your local machine
- Clone [example instance's repository](https://github.com/jet-dev-team/metrics-demo-jet-dev)
- Make sure your instance's code is located in `instances` directory
- Make sure your instance's `ssl` directory contains `cert.crt` and `cert.key` files
- Prepare fresh server. Example cloud-init could be found in `instances/metrics.demo.jet.dev/cloud-init.yml`
- Make sure that `instances/metrics.demo.jet.dev/inventory.ini` contains valid configuration
- Run ansible playbook

```sh
ansible-playbook -i instances/metrics.demo.jet.dev/inventory.ini playbooks/main.yml
```

How to run playbook using specific private key

```sh
ansible-playbook -i instances/metrics.demo.jet.dev/inventory.ini playbooks/main.yml --private-key ~/.ssh/ci_metrics_demo_jet_dev.key
```

## Setup simple Telegraf Agent an the server which should be monitored

Install telegraf: [Telegraf Documentation](https://docs.influxdata.com/telegraf/v1.17/introduction/installation/)

```sh
wget -qO- https://repos.influxdata.com/influxdb.key | sudo apt-key add -
source /etc/lsb-release
echo "deb https://repos.influxdata.com/${DISTRIB_ID,,} ${DISTRIB_CODENAME} stable" | sudo tee /etc/apt/sources.list.d/influxdb.list
sudo apt update
sudo apt install telegraf
sudo mv /etc/telegraf/telegraf.conf{,.bak}
sudo vi /etc/telegraf/telegraf.conf
```

```conf
[global_tags]

[agent]
  interval = "10s"
  round_interval = true
  metric_batch_size = 1000
  metric_buffer_limit = 10000
  collection_jitter = "0s"
  flush_interval = "10s"
  flush_jitter = "0s"
  precision = ""
  hostname = "jet.dev"
  omit_hostname = false

[[outputs.influxdb]]
  urls = ["https://metrics.demo.jet.dev:8443"]
  database = "telegraf"
  username = "secret"
  password = "secret"

[[inputs.cpu]]
[[inputs.disk]]
[[inputs.diskio]]
[[inputs.kernel]]
[[inputs.mem]]
[[inputs.processes]]
[[inputs.swap]]
[[inputs.system]]
[[inputs.net]]
[[inputs.netstat]]
[[inputs.interrupts]]
[[inputs.linux_sysctl_fs]]

[[inputs.syslog]]
  server = "udp://:6514"
  keep_alive_period = "5m"
  max_connections = 0
  read_timeout = "5s"
  framing = "octet-counting"
  trailer = "LF"
  best_effort = false
  sdparam_separator = "_"

# Convert values to another metric value type
[[processors.converter]]
  namepass = ["syslog"]
  order = 2

  ## Fields to convert
  ##
  ## The table key determines the target type, and the array of key-values
  ## select the keys to convert.  The array may contain globs.
  ##   <target-type> = [<field-key>...]
  [processors.converter.fields]
    tag = ["type", "base_url"]

[[processors.regex]]
  namepass = ["syslog"]
  order = 1

  [[processors.regex.fields]]
    key = "message"
    pattern = "^(?P<base_url>.*)\\|(?P<timestamp>\\d+)\\|(?P<type>.*)\\|(?P<ip>\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}\\.\\d{1,3})\\|(?P<request_uri>.*)\\|(?P<referer>.*)\\|(?P<uid>\\d+)\\|(?P<link>.*?)\\|(?P<message>.*)"
    replacement = "${base_url}"
    result_key = "base_url"

  [[processors.regex.fields]]
    key = "message"
    pattern = "^(?P<base_url>.*)\\|(?P<timestamp>\\d+)\\|(?P<type>.*)\\|(?P<ip>\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}\\.\\d{1,3})\\|(?P<request_uri>.*)\\|(?P<referer>.*)\\|(?P<uid>\\d+)\\|(?P<link>.*?)\\|(?P<message>.*)"
    replacement = "${timestamp}"
    result_key = "timestamp_part"

  [[processors.regex.fields]]
    key = "message"
    pattern = "^(?P<base_url>.*)\\|(?P<timestamp>\\d+)\\|(?P<type>.*)\\|(?P<ip>\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}\\.\\d{1,3})\\|(?P<request_uri>.*)\\|(?P<referer>.*)\\|(?P<uid>\\d+)\\|(?P<link>.*?)\\|(?P<message>.*)"
    replacement = "${type}"
    result_key = "type"

  [[processors.regex.fields]]
    key = "message"
    pattern = "^(?P<base_url>.*)\\|(?P<timestamp>\\d+)\\|(?P<type>.*)\\|(?P<ip>\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}\\.\\d{1,3})\\|(?P<request_uri>.*)\\|(?P<referer>.*)\\|(?P<uid>\\d+)\\|(?P<link>.*?)\\|(?P<message>.*)"
    replacement = "${ip}"
    result_key = "ip"

  [[processors.regex.fields]]
    key = "message"
    pattern = "^(?P<base_url>.*)\\|(?P<timestamp>\\d+)\\|(?P<type>.*)\\|(?P<ip>\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}\\.\\d{1,3})\\|(?P<request_uri>.*)\\|(?P<referer>.*)\\|(?P<uid>\\d+)\\|(?P<link>.*?)\\|(?P<message>.*)"
    replacement = "${request_uri}"
    result_key = "request_uri"

  [[processors.regex.fields]]
    key = "message"
    pattern = "^(?P<base_url>.*)\\|(?P<timestamp>\\d+)\\|(?P<type>.*)\\|(?P<ip>\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}\\.\\d{1,3})\\|(?P<request_uri>.*)\\|(?P<referer>.*)\\|(?P<uid>\\d+)\\|(?P<link>.*?)\\|(?P<message>.*)"
    replacement = "${referer}"
    result_key = "referer"

  [[processors.regex.fields]]
    key = "message"
    pattern = "^(?P<base_url>.*)\\|(?P<timestamp>\\d+)\\|(?P<type>.*)\\|(?P<ip>\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}\\.\\d{1,3})\\|(?P<request_uri>.*)\\|(?P<referer>.*)\\|(?P<uid>\\d+)\\|(?P<link>.*?)\\|(?P<message>.*)"
    replacement = "${uid}"
    result_key = "uid"

  [[processors.regex.fields]]
    key = "message"
    pattern = "^(?P<base_url>.*)\\|(?P<timestamp>\\d+)\\|(?P<type>.*)\\|(?P<ip>\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}\\.\\d{1,3})\\|(?P<request_uri>.*)\\|(?P<referer>.*)\\|(?P<uid>\\d+)\\|(?P<link>.*?)\\|(?P<message>.*)"
    replacement = "${link}"
    result_key = "link"

  [[processors.regex.fields]]
    key = "message"
    pattern = "^(?P<base_url>.*)\\|(?P<timestamp>\\d+)\\|(?P<type>.*)\\|(?P<ip>\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}\\.\\d{1,3})\\|(?P<request_uri>.*)\\|(?P<referer>.*)\\|(?P<uid>\\d+)\\|(?P<link>.*?)\\|(?P<message>.*)"
    replacement = "${message}"
    result_key = "message_part"
```

Configure rsyslog.

Edit `/etc/rsyslog.d/90-telegraf.conf`

```conf
$ActionQueueType LinkedList
$ActionQueueFileName srvrfwd
$ActionResumeRetryCount -1
$ActionQueueSaveOnShutdown on

action(
  type="omfwd"
  Protocol="udp"
  Target="127.0.0.1"
  Port="6514"
  Template="RSYSLOG_SyslogProtocol23Format"
)
```

```sh
sudo systemctl restart rsyslog
```

Restart.

```sh
sudo systemctl restart telegraf
```

In order to add to syslog specific log file the next config could be used.
The config below is reading the Apache error log file with severity `err`.

```sh
$ModLoad imfile
$InputFilePollInterval 10
$InputFileName  /var/log/apache2/error.log
$InputFileTag apache2-error
$InputFileStateFile stat-apache-error
$InputFileSeverity err
$InputRunFileMonitor
$InputFileFacility local7
```

## Articles

- [Telegraf / InfluxDB / Grafana as syslog receiver](https://nwmichl.net/2020/03/15/telegraf-influxdb-grafana-as-syslog-receiver/)
- [Grafana Auth Proxy Authentication](https://grafana.com/docs/grafana/latest/auth/auth-proxy/)
- [Installing Telegraf on Ubuntu 20.04](https://bist.be/2020/06/11/installing-telegraf-on-ubuntu-20-04/)
- [Creating default user for influxdb 2.0 in docker-compose](https://stackoverflow.com/a/66398814)
- [Running InfluxDB 2.0 and Telegraf Using Docker](https://www.influxdata.com/blog/running-influxdb-2-0-and-telegraf-using-docker/)
- [How to setup and secure Telegraf, InfluxDB and Grafana on Linux](https://sysadmin.info.pl/en/how-to-setup-and-secure-telegraf-influxdb-and-grafana-on-linux/)
- [Grafana and InfluxDB with Let's Encrypt SSL on Docker](https://www.grzegorowski.com/grafana-with-lets-encrypt-ssl-on-docker)

## HOWTO

- [Export dashboards](https://gist.github.com/crisidev/bd52bdcc7f029be2f295#gistcomment-3593795)
- [Configure logrotate for docker to avoid "no left space on device"](https://medium.com/free-code-camp/how-to-setup-log-rotation-for-a-docker-container-a508093912b2)

### InfluxDB CLI

Basic commands:

```sh
influx -ssl -host monitor.sandbox.jet.dev -port 8086 -username '' -password ''
SHOW DATABASES
USE telegraf
SHOW MEASUREMENTS
SHOW FIELD KEYS
SHOW TAG KEYS
```

Onelinner.

```sql
influx -ssl -host example.com -port 8086 -username '' -password '' -execute "SHOW DATABASES"
```

### Retentionn Policy for InfluxDB

Show retention policies.

```sql
SHOW RETENTION POLICIES ON telegraf;
```

Change retention policy.

```sql
ALTER RETENTION POLICY autogen ON telegraf DURATION 4w;
```
