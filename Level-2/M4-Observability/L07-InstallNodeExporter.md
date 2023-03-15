## Node Exporter 

The Prometheus Node Exporter exposes a wide variety of hardware and kernel-related metrics. Node Exporter is installed on all the nodes, for which we need to scrape Node Metrics such as cpu utilization, ram available, disk used etc 

## Monitoring EC2 instance uisng  NODE Exporter by using the following commands:
```
https://devopscube.com/monitor-linux-servers-prometheus-node-exporter/ 
cd /tmp
curl -LO https://github.com/prometheus/node_exporter/releases/download/v0.18.1/node_exporter-0.18.1.linux-amd64.tar.gz
tar -xvf node_exporter-0.18.1.linux-amd64.tar.gz
sudo mv node_exporter-0.18.1.linux-amd64/node_exporter /usr/local/bin/
sudo useradd -rs /bin/false node_exporter
```
## create a service file
```
sudo vi /etc/systemd/system/node_exporter.service
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```
## start and enable the service 

```
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl status node_exporter
sudo systemctl enable node_exporter
```
- Check metrics http://<server-IP>:9100/metrics

## Now configure in prometheus.yml file sudo vi /etc/prometheus/prometheus.yml
```
- job_name: 'node_exporter_metrics'
  scrape_interval: 5s
  static_configs:
    - targets: ['10.142.0.3:9100']
```
## restart the service and check the port 9090

- sudo systemctl restart prometheus

- Now check  targets at http://<prometheus-IP>:9090/targets

- Import dashboard 11074 and 1860 for NODE METRICS DASHBOARD


## Note : Use will get hands-on upon Observability tools in LEVEL 3 
