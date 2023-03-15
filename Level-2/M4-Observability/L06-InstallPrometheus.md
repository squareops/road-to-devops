## Prometheus 

Prometheus collects and stores its metrics as time series data, i.e. metrics information is stored with the timestamp at which it was recorded, alongside optional key-value pairs called labels.

Prometheus's main features are:

- a multi-dimensional data model with time series data identified by metric name and key/value pairs
- PromQL, a flexible query language to leverage this dimensionality
- no reliance on distributed storage; single server nodes are autonomous
- time series collection happens via a pull model over HTTP
- pushing time series is supported via an intermediary gateway
- targets are discovered via service discovery or static configuration
- multiple modes of graphing and dashboarding support
  
## Install PROMETHEUS by running the following command on ubuntu server
```
sudo groupadd --system prometheus
sudo useradd -s /sbin/nologin --system -g prometheus prometheus
sudo mkdir /var/lib/prometheus
for i in rules rules.d files_sd; do sudo mkdir -p /etc/prometheus/${i}; done
sudo apt update
sudo apt -y install wget curl vim
mkdir -p /tmp/prometheus && cd /tmp/prometheus
curl -s https://api.github.com/repos/prometheus/prometheus/releases/latest | grep browser_download_url | grep linux-amd64 | cut -d '"' -f 4 | wget -qi -
tar xvf prometheus*.tar.gz
cd prometheus*/
sudo mv prometheus promtool /usr/local/bin/
prometheus --version
promtool --version
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
sudo mv consoles/ console_libraries/ /etc/prometheus/
cd $HOME

sudo vim /etc/prometheus/prometheus.yml
```
- Paste the following contents as per your monitoring resources in prometheus.yml file:
```
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
rule_files:
  - alert.rules.yml
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - 'localhost:9093'
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
  - job_name: "mongodb1-exporter"
    static_configs:
      - targets: ["10.0.3.149:9216"]
  - job_name: "mongodb2-exporter"
    static_configs:
      - targets: ["10.0.3.20:9216"]
  - job_name: "mongodb3-exporter"
    static_configs:
      - targets: ["10.0.3.135:9216"]
  - job_name: "EC2-observability-node-exporter"
    static_configs:
      - targets: ["localhost:9100"]
  - job_name: "EC2-mongodb1-node-exporter"
    static_configs:
      - targets: ["10.0.3.149:9100"]
  - job_name: "EC2-mongodb2-node-exporter"
    static_configs:
      - targets: ["10.0.3.20:9100"]
  - job_name: "EC2-mongodb3-node-exporter"
    static_configs:
      - targets: ["10.0.3.135:9100"]
  - job_name: 'blackbox-http-endpoints-status'
    metrics_path: /probe
    params:
      module: [http_2xx] # Look for a HTTP 200 response.
    static_configs:
      - targets:
         - https://abc.com/
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: localhost:9115  # The blackbox exporter's real hostname:port.
  - job_name: "ECS-node-exporter"
    # static_configs:
    #   - targets: ["13.126.58.244:8101"]
    ec2_sd_configs:
      - region: eu-central-1
        port: 8101
        refresh_interval : 5s
        profile: arn:aws:iam::123:instance-profile/ssm-access-role
        # role_arn: arn:aws:iam::123:role/ssm-access-role
        # access_key: 111111111111
        # secret_key: 222222222222222222222222222
    relabel_configs:
      - source_labels: [__meta_ec2_tag_Name]
        regex: ECS Instance - EC2ContainerService
        action: keep
      - source_labels: [__meta_ec2_tag_Name]
        target_label: Name
      - source_labels: [__meta_ec2_private_ip]
        target_label: IP
      - source_labels: [__meta_ec2_private_dns_name]
        target_label: Private_DNS
      - source_labels: [__meta_ec2_instance_state]
        target_label: State
      - source_labels: [__meta_ec2_instance_type]
        target_label: Instance_type
      - source_labels: [__meta_ec2_availability_zone]
        target_label: Zone
      - source_labels: [__meta_ec2_ami]
        target_label: AMI
      - source_labels: [__meta_ec2_primary_subnet_id]
        target_label: Subnet
      - source_labels: [__meta_ec2_instance_id]
        target_label: instance
  - job_name: "ECS-cadvisor"
    ec2_sd_configs:
      - region: eu-central-1
        port: 8202
        refresh_interval : 5s
        profile: arn:aws:iam::12345678:instance-profile/ssm-access-role
        # role_arn: arn:aws:iam::12345678:role/ssm-access-role
    relabel_configs:
      - source_labels: [__meta_ec2_tag_Name]
        regex: ECS Instance - EC2ContainerService
        action: keep
      - source_labels: [__meta_ec2_tag_Name]
        target_label: Name
      - source_labels: [__meta_ec2_private_ip]
        target_label: IP
      - source_labels: [__meta_ec2_private_dns_name]
        target_label: Private_DNS
      - source_labels: [__meta_ec2_instance_state]
        target_label: State
      - source_labels: [__meta_ec2_instance_type]
        target_label: Instance_type
      - source_labels: [__meta_ec2_availability_zone]
        target_label: Zone
      - source_labels: [__meta_ec2_ami]
        target_label: AMI
      - source_labels: [__meta_ec2_primary_subnet_id]
        target_label: Subnet
      - source_labels: [__meta_ec2_instance_id]
        target_label: instance
```
- For creating service, run the following command:
```
sudo tee /etc/systemd/system/prometheus.service<<EOF
[Unit]
Description=Prometheus
Documentation=https://prometheus.io/docs/introduction/overview/
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=prometheus
Group=prometheus
ExecReload=/bin/kill -HUP \$MAINPID
ExecStart=/usr/local/bin/prometheus \
--config.file=/etc/prometheus/prometheus.yml \
--storage.tsdb.path=/var/lib/prometheus \
--web.console.templates=/etc/prometheus/consoles \
--web.console.libraries=/etc/prometheus/console_libraries \
--web.listen-address=0.0.0.0:9090 \
--web.external-url=

SyslogIdentifier=prometheus
Restart=always

[Install]
WantedBy=multi-user.target
EOF
```
- Change owner and file permissions 
```
for i in rules rules.d files_sd; do sudo chown -R prometheus:prometheus /etc/prometheus/${i}; done
for i in rules rules.d files_sd; do sudo chmod -R 775 /etc/prometheus/${i}; done
sudo chown -R prometheus:prometheus /var/lib/prometheus/
```
- Enable and start the service 
```
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus
systemctl status prometheus
```

- allow port 9090 in sg
- To set retention in prometheus edit the prometheus.service file and add the flag storage.tsdb.retention.time

```
[Unit]
Description=Prometheus
Documentation=https://prometheus.io/docs/introduction/overview/
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=prometheus
Group=prometheus
ExecReload=/bin/kill -HUP $MAINPID
ExecStart=/usr/local/bin/prometheus   --config.file=/etc/prometheus/prometheus.yml   --storage.tsdb.path=/var/lib/prometheus   --web.console.templates=/etc/prometheus/consoles   --web.console.libraries=/etc/prometheus/console_libraries   --web.listen-address=0.0.0.0:9090   --storage.tsdb.retention.time=45d   --web.external-url=

SyslogIdentifier=prometheus
Restart=always

[Install]
WantedBy=multi-user.target

```

## Note : Use will get hands-on upon Observability tools in LEVEL 3 