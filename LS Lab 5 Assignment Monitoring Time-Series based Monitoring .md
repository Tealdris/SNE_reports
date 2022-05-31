###### tags: `LS`
:::success
# LS Lab 5 Assignment: Monitoring: Time-Series based Monitoring:
Prometheus + TSDB + Grafana
**Name: Radomskiy Ilia, Fige Polina**
:::


---

## Task 0: Choose subtopick

My id is `st 10` and the id of my partner is `st 7` so, according to the assignment subtopic is the second one.
```
10 mod 3 = 1, or subtopic 2
7 mod 3 = 1, or subtopic 2
```

**Bonus 1**: wrap your deployment in tasks 1-2 as `ansible` roles.

`Ansible` playbooks are stored in the [github](https://github.com/Tealdris/ls5_monitoring.git) repository. Rules are stored in the `ansible/playbooks/templates`. The main playbook for the deploying monitoring environment is stored in the `monitoring.yml` playbook.

**Bonus 2**: organize a process of cross-systems notifications about your alerts.

---

## Task 1: Prepare monitoring environment
:::info
Choose and deploy your monitoring stack:
* **Prometheus** as core monitoring system for metrics processing
* **InfluxDB** as TSDB for time series metrics storing
* **Grafana** as software for metrics visualization and reports generation
:::

For this lab, I will choose a combination of the following instruments: **Prometheus**, **InfluxDB** and **Grafana**
1. [Prometheus](https://prometheus.io/) is as tool for systems monitoring. It can be represented as a data base of the time series. It also scalable due to ability of expancion with the help of modules.
One of the advantages of `Prometheus` a centralized method of data aggregation.
Installation
For the installation we need `docker`

```bash=
sudo docker pull quay.io/prometheus/prometheus
```

This configuration file is used by `prometheus` in docker

```bash=
global:
  scrape_interval:     15s
  evaluation_interval: 15s
 
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
    - targets: ['localhost:9090']
  - job_name: 'node'
    static_configs:
    - targets:
      - localhost:9100
```

docker file created for the `Prometheus`

```dockerfile=
sudo docker run \
--detach \
--interactive \
--tty \
--network host \
--name prometheus \
--restart always \
--volume prometheus:/prometheus \
--volume /etc/prometheus.d:/etc/prometheus.d:Z \
--volume /etc/pki/tls/certs/ca-bundle.crt:/etc/ssl/certs/ca-certificates.crt:z \
quay.io/prometheus/prometheus \
--config.file=/etc/prometheus.d/prometheus.yml \
--web.external-url=http://$(hostname -f):9090 \
--web.enable-lifecycle \
--web.enable-admin-api \
--web.listen-address=:9090
```

After that, we might need to install additional modules for the `Prometheus` like `node exporter` or `Push gateway`. We will do this in this particular section.
In the picture below you can see working `prometheus`

<center>

![](https://i.imgur.com/HViYGaq.png)
![](https://i.imgur.com/TTz04E4.png)

Figure 1, 2 - Prometheus
</center>
    
:::spoiler

prometeus
`https://www.digitalocean.com/community/tutorials/how-to-install-prometheus-on-ubuntu-16-04`

app
9090
need user

`https://blog.christophersmart.com/2019/09/08/setting-up-a-monitoring-host-with-prometheus-influxdb-and-grafana/`
```
sudo docker run \
--detach \
--interactive \
--tty \
--network host \
--name prometheus \
--restart always \
--volume prometheus:/prometheus \
--volume /etc/prometheus.d:/etc/prometheus.d:Z \
quay.io/prometheus/prometheus \
--config.file=/etc/prometheus.d/prometheus.yml \
--web.external-url=http://$(hostname -f):9090 \
--web.enable-lifecycle \
--web.enable-admin-api \
--web.listen-address=:9090
```
:::


2. [InfluxDB](https://docs.influxdata.com/influxdb/v2.0/) is a database. It is used to store time series, metrics, and information about events. In the same way to the `prometheus` data in `InfluxDB` stored as `key-value`

It can be installed with

```bash=
sudo apt install influxdb
sudo apt install influxdb-client
```

Database requares this change configurataion of `/etc/influxdb/influxdb.conf` in order to work

```bash=
sudo nano /etc/influxdb/influxdb.conf
[[collectd]]
  enabled = true
  bind-address = ":25826"
  database = "collectd"
  retention-policy = ""
   typesdb = "/usr/local/share/collectd"
   security-level = "none"
```

After that, we are ready to work with databases. We can create one with the command below.

```bash=
influx -execute 'CREATE DATABASE collectd'
```

:::spoiler
```
sudo apt install influxdb
sudo apt install influxdb-client

sudo sed-i 's/^\[\[collectd\]\]/#\[\[collectd\]\]/' /etc/influxdb/influxdb.conf
cat << EOF | sudo tee -a /etc/influxdb/influxdb.conf
[[collectd]]
  enabled = true
  bind-address = ":25826"
  database = "collectd"
  retention-policy = ""
   typesdb = "/usr/local/share/collectd/types.db"
   security-level = "none"
EOF
```
:::

The easiest to install is `Grafana`.
[Grafana](http://grafana.com/docs/) is a visualization agent and it is also used in order to analyze data.
And it easily can be done with the help of official documentation

```bash=
apt-get install -y apt-transport-https
apt-get install -y software-properties-common wget
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/enterprise/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
echo "deb https://packages.grafana.com/enterprise/deb beta main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
apt-get update
apt-get install grafana-enterprise -y
systemctl enable grafana-server
systemctl start grafana-server
```

:::spoiler

https://grafana.com/docs/grafana/latest/installation/debian/

graphana
4m7AAyW48AsstEB
:::

Application, that is used in this lab, is stored in the [repository](https://github.com/Tealdris/webnotes.git) and was used in one of the previous labs
On the pictures below you can see the working applications

<center>

![](https://i.imgur.com/oKMl8pZ.png)
    
![](https://i.imgur.com/PAvkzUh.png)
Figures 3, 4 - Apps
</center>


---

## Task 2: Configure monitoring agents

:::info
You have to deploy and configure metrics exporters on your monitoring agents (targets instances). Targets might be your VMs and applications/services that you have used in previous labs on this course. Try to set up different types of exporters and as much as possible, but usually core exporters are:
* exporter for instance metrics (`nodexporter`)
* exporter for containers metrics (`cadvisor` is one of the most popular solution)
* exporter for containers states (`dockerstatesexporter`)
* **bonus**: integrated exporter of your application/service metrics (here you need to write a logic to expose your service metrics on particular port and enable Service discovery Prometheus feature)
:::

`nodexporter` is used to monitor linux machines. This plugin accrues variety of the metrics, such as memory, CPU load, etc

installation commands are listed below

```bash=
curl -LO https://github.com/prometheus/node_exporter/releases/download/v0.15.1/node_exporter-0.15.1.linux-amd64.tar.gz
tar xvf node_exporter-0.15.1.linux-amd64.tar.gz
    sudo cp node_exporter-0.15.1.linux-amd64/node_exporter /usr/local/bin
    sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
rm -rf node_exporter-0.15.1.linux-amd64.tar.gz node_exporter-0.15.1.linux-amd64
```

With the following commands we can make `nodexporter`  service

```bash=
sudo nano /etc/systemd/system/node_exporter.service
[Unit]

Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target


sudo nano /etc/systemd/system/node_exporter.service

ExecStart=/usr/local/bin/node_exporter --collectors.enabled meminfo,loadavg,filesystem
```

Config below are exposing `nodexporter` to the `Prometheus` 
The configuration file is stored in the `/etc/prometheus/prometheus.yml` folder

```bash=
global:
  scrape_interval:     15s
  evaluation_interval: 15s
rule_files:
  - rules.yml
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
    - targets: ['3.134.83.81:9090']
  - job_name: 'node'
    static_configs:
    - targets:
      - 3.134.83.81:9100
```

:::spoiler

**Other config file from st7:**
global:
  scrape_interval:     15s
  evaluation_interval: 15s
rule_files:
  - rules.yml
alerting:
  alertmanagers:
  - static_configs:
    - targets: ['18.219.8.195:9093']
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
    - targets: ['18.219.8.195:9090']
  - job_name: 'node'
    static_configs:
    - targets: ['18.219.8.195:9100']


node from digital ocean with user from step 1
https://www.digitalocean.com/community/tutorials/how-to-install-prometheus-on-ubuntu-16-04
:::

After installation `node exporter` is available on the port `9100`

<center>

![](https://i.imgur.com/Nn1GC25.png)
![](https://i.imgur.com/raCuX5C.png)

Figures 5, 6 - Metrics
</center>

---


## Task 3: Processing with Prometheus

:::info
1. Since your monitoring infrastructure and agents metrics exporters are configured, log in Prometheus console, and make sure that you are getting metrics:
* go to `Targets` and confirm that you can see your target instances
* go to `Graph` and confirm that you can see coming metrics via PromQL
:::

By entering `Prometheus/Status/Targets` we can see all the metrics ports that are used by the `Prometheus`. In the example below there are two ports available to work with, `node exporter` and `Prometheus` native one

<center>

![](https://i.imgur.com/gdKWgWY.png)
Figure 7 - Targets
</center>

We can see this metric as graphs if we enter the specified link `Graph` in the `Prometheus` web page.
 
<center>
    
![](https://i.imgur.com/ZsmVmCk.png)    
![](https://i.imgur.com/vcbL0BW.png)

Figures 8, 9 - Checking memory statistics
    
</center> 

But I will not stop on this stage because we will use `grafana` to show graphics based on metrics.

:::info
2. Create and write your alerts rules in prometheus.rules.yml file. Try to play with different alerts types and test and as much as possible. Some common alerts are:
* high instance memory usage
* high instance cpu usage
* high instance disk usage
* docker container restarting
* docker container exited
* docker container dead
* exporter down
* kubernetes pod crashed
* specific alerts that related to your app/service: high heap usage, high cache hit rate usage
:::

One of the features of `Prometheus` it is to create alert rules for the jobs, metrics, or services.
In the picture below you can see rule that monitors the `node_exporter` job. If it `node_exporter`  is not responding for 1 minute, alert warns about the down

<center>

![](https://i.imgur.com/1CjNxLG.png)
Figure 10 - Example of rule
</center>

If the `Prometheus` is configured correct to work with the rules we can see that rule in the web page of `Prometheus`.

<center>

![](https://i.imgur.com/FL9PMJ9.png)
Figure 11 - Visible rule
</center>

You need to change two things in order to make them work
File with rules itself `/etc/prometheus/rules.yml`. In this field all the rules are stored in the yml format.
Main configuration file of the prometheus `/etc/prometheus/prometheus.yml`
All 4 rules, implemented in this task are shown on the picture below.

<center>

![](https://i.imgur.com/rm2IEel.png)
Figure 12 - List of rules
</center>

<center>

![](https://i.imgur.com/rDWqmCQ.png)
Figure 13 - Example of how it works
</center>

:::info
3. Apply your alerts rules and provoke some alerts conditions. Confirm that you can see alerts on Prometheus dashboard.
**Bonus**: configure Alertmanager for alerts visualization. Group alerts by instances, types, environments... Provoke false-positive alerts and silence them.
:::

`Alertmanager` can help us to send notifications. So, to this end it will be configured.

Appliaction can be downloaded from this web [page](https://prometheus.io/download/#alertmanager)

After that we can copy configuration of the `Alertmanager` to `/etc/alertmanager`, set user permissions on the folders and make it a service

```bash=
[Unit]
Description=Alertmanager Service
After=network.target

[Service]
EnvironmentFile=-/etc/default/alertmanager
User=alertmanager
Group=alertmanager
Type=simple
ExecStart=/usr/local/bin/alertmanager \
          --config.file=/etc/alertmanager/alertmanager.yml \
          --storage.path=/var/lib/prometheus/alertmanager \
          $ALERTMANAGER_OPTS
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

In the example below we can see how the rule was triggered by the `node_exporter`'s down

<center>

![](https://i.imgur.com/GvseYwM.png)
Figure 14 - Alert manager

</center>

<center>

![](https://i.imgur.com/BEBHczz.png)
Figure 15 - Example of how rule works
</center>

At this stage, notification is sent to us about condition of the application.

If we turn back to the`node_exporter`  we will receive this message below:

<center>

![](https://i.imgur.com/9RnXbRy.png)
Figure 16 - New message
</center>

## Task 4: Processing with Grafana

:::info
After you set up metrics catching and alerts rules, it it time to visualize your metrics and deploy
elegant charts.
1. Log in Grafana console and create several dashboards that are separated by metrics types
and logic.
2. Deploy some panels in dashboards. Write expressions (metrics query) for them and name
them according to panels purposes.
3. Make sure that your dashboards are ready and work properly.
**Bonus**: use dynamic variables within Dashboards.
:::

If the `Grafana` is configured correctly, then we can enter its web page. Default login\password = admin\admin

<center>

![](https://i.imgur.com/z3GcbLc.png)
Figure 17 - Grafana's interface
</center>

Now, in order to make the `Grafana` usefull, we need to add Data sources to it. On the two pictures you can see, how `Prometheus` and `InfluxDB` was added as data sources

<center>
    
![](https://i.imgur.com/taGkg0z.png)
Figure 18 - Add Promethus
</center>

<center>
    
![](https://i.imgur.com/pqP2tPd.png)
Figure 19 - Add InfluxDB
</center>

After successful addition we can see, that it is possible for us to get metrics from them, and make pretty beautiful graphics

<center>
        
![](https://i.imgur.com/MJlmsWJ.png)
   
    
![](https://i.imgur.com/uI4xnPQ.png)
Figures 20, 21 - Graphics    
</center>

---


:::spoiler

aws vars
export AWS_ACCESS_KEY_ID=
export AWS_SECRET_ACCESS_KEY=

sudo docker restart prometheus

sudo docker exec -it $(sudo docker ps -aq) promtool check rules alert.rules.yml
/etc/prometheus/rules.yml

sudo docker exec -it $(sudo docker ps -aq) ls /etc/alertmanager


sudo docker rm $(sudo docker ps -aq) --force
docker-compose up -d


sudo docker run --name alertmanager -d -p 3.134.83.81:9093:9093 quay.io/prometheus/alertmanager

./alertmanager --config.file="alertmanager.yml"

or 

/usr/share/prometheus/alertmanager/generate-ui.sh


https://www.dmosk.ru/instruktions.php?object=prometheus-linux
/etc/prometheus
/var/lib/prometheus


/etc/alertmanager /var/lib/prometheus/alertmanager

systemctl start prometheus
systemctl start alertmanager
systemctl start node_exporter

systemctl status prometheus
:::

## References:
1. [How To Install Prometheus on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-prometheus-on-ubuntu-16-04)
2. [Prometheus Configuration](https://prometheus.io/docs/prometheus/latest/configuration/configuration/)
3. [List of Prometheus rules](https://awesome-prometheus-alerts.grep.to/rules.html)