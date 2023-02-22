## Install GRAFANA by running the following command on ubuntu server
```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install -y gnupg2 curl
curl https://packages.grafana.com/gpg.key | sudo apt-key add -
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
sudo apt-get update
sudo apt-get -y install grafana
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
now check grafana is accessible on (http://server_IP:3000)
make sure you have allowed 3000 port in sg

```

![](Images/g1.png)


