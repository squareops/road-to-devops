## Promtail Agent 

Promtail is an agent which ships the contents of local logs to a private Grafana Loki instance or Grafana Cloud. It is usually deployed to every machine that has applications needed to be monitored.

It primarily:

- Discovers targets.
  
- Attaches labels to log streams.
  
- Pushes them to the Loki instance.
  
## Install PROMTAIL AGENT by running the following command on ubuntu server
```
curl -s https://api.github.com/repos/grafana/loki/releases/latest | grep browser_download_url | cut -d '"' -f 4 | grep promtail-linux-amd64.zip | wget -i -
unzip promtail-linux-amd64.zip
sudo mv promtail-linux-amd64 /usr/local/bin/promtail
promtail --version
sudo vim /etc/promtail-local-config.yaml
```
Paste the following contents in promtail-local-config.yaml 
```
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /data/loki/positions.yaml

clients:
  - url: http://localhost:3100/loki/api/v1/push

scrape_configs:
- job_name: system
  static_configs:
  - targets:
      - localhost
    labels:
      job: varlogs
      __path__: /var/log/*log

```
```
sudo tee /etc/systemd/system/promtail.service<<EOF
[Unit]
Description=Promtail service
After=network.target

[Service]
Type=simple
User=root
ExecStart=/usr/local/bin/promtail -config.file /etc/promtail-local-config.yaml

[Install]
WantedBy=multi-user.target
EOF
```
## Start and check the status of the service 

        - sudo systemctl daemon-reload

        - sudo systemctl start promtail.service

        - systemctl status promtail.service


## Note : Use will get hands-on upon Observability tools in LEVEL 3 