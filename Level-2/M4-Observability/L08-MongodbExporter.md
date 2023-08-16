## MongoDB Exporter 
MongoDB exporter handles ALL metrics exposed by MongoDB monitoring commands. It loops over all the fields exposed in diagnostic commands and tries to get data from them.


## Scrape MONGODB METRICS from EC2 instances using MONGODB EXPORTER by using the following commands
```
$ mkdir mongodb-exporter
$ cd mongodb-exporter
$ wget https://github.com/percona/mongodb_exporter/releases/download/v0.7.1/mongodb_exporter-0.7.1.linux-amd64.tar.gz
$ tar xvzf mongodb_exporter-0.7.1.linux-amd64.tar.gz
$ sudo useradd -rs /bin/false prometheus
$ sudo mv mongodb_exporter /usr/local/bin/
```
- Enabling authentication (optional)
```
use admin
db.createUser(
  {
    user: "mongodb_exporter",
    pwd: "password",
    roles: [
        { role: "clusterMonitor", db: "admin" },
        { role: "read", db: "local" }
    ]
  }
)

db.createUser({
        user: "mongodb_exporter_admin",
        pwd:  "hagjdh179hskdhuhdkdh",
        roles: [{role: "userAdmin" , db: "admin"}]
});
```

## Create service

```
$ cd /lib/systemd/system/
$ sudo touch mongodb_exporter.service
[Unit]
Description=MongoDB Exporter
User=prometheus

[Service]
Type=simple
Restart=always
ExecStart=/usr/local/bin/mongodb_exporter --mongodb.uri=mongodb://root-admin:Hdoodgb9Z17jgf12xy@localhost:27017 --compatible-mode

[Install]
WantedBy=multi-user.target
```

## Start service 
```
sudo systemctl daemon-reload
sudo systemctl start mongodb_exporter.service
```
## Configure the MongoDB exporter as a Prometheus target

```
- job_name: "mongodb1-exporter"
    static_configs:
      - targets: ["10.0.3.149:9216"]   
```

- Restart prometheus and check prometheus status 
- To create mongodb metrics dashboard , import the following :
https://github.com/percona/grafana-dashboards/blob/pmm-1.x/dashboards/MongoDB_Overview.json 

## For more information:

https://devconnected.com/mongodb-monitoring-with-grafana-prometheus 

## Note : Use will get hands-on upon Observability tools in LEVEL 3 