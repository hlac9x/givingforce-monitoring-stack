# Prerequisites

- Asible installed
- Update the Ansible Inventory


# Deploy The Monitoring Stack first

Update the monitoring server IP address in file inventory.yaml
Comment "#" the monitoring clients in file inventory.yaml
Run command:

```
ansible-playbook -i inventory.yml playbook.yml  
```

# Deploy The Monitoring Stack

Comment monitoring server in file inventory.yaml
Uncomment "#" and the monitoring clients IP address in file inventory.yaml
Run command:

```
ansible-playbook -i inventory.yml playbook.yml  
```

# Dockerfile

There are 2 Dockerfile to deploy mornitoring stack and monitoring clients

# Docker containers name showing in Loki

In the official doc : https://docs.docker.com/config/containers/logging/json-file/ , "tag" isn't on the drivers opts list.

So, as mentionned upper, for docker run cli use :
--log-opt tag="{{.ImageName}}|{{.Name}}|{{.ImageFullID}}|{{.FullID}}"

For docker compose :

```
    logging:
      driver: "json-file"
      options: 
        tag: "{{.ImageName}}|{{.Name}}|{{.ImageFullID}}|{{.FullID}}"
```

And for directly add this to the /etc/docker/daemon.json :

```
{
   "experimental":true, 
   "metrics-addr":"0.0.0.0:9323", 
   "insecure-registries":[
      "vip-registry-cache.domain.tld:5000", 
      "vip-registry-build.domain.tld:5001" 
   ],
   "hosts":[
      "tcp://0.0.0.0:2375", 
      "unix:///var/run/docker.sock" 
   ], 
   "log-driver": "json-file",
   "log-opts": {
      "tag": "{{.ImageName}}|{{.Name}}|{{.ImageFullID}}|{{.FullID}}"
   }
}

```
# Configure datasource in Grafana
```
Loki: http://loki:3100
Prometheus: http://prometheus:9090
```

# Endpoints
```
Grafana: http://[monitoringserver'IP]:3000
Loki: http://[monitoringserver'IP]:3100
Prometheus: http://[monitoringserver'IP]:9090
```

# Deploy mysql exporter on MySQL instance
```
Step 1. Create the export user

CREATE USER 'exporter'@'%' IDENTIFIED BY 'TxTCvv8dwp' WITH MAX_USER_CONNECTIONS 3;
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'%';

Step 2. Docker command with flags for mysql exporter

docker run -d \
  --name mysqld-exporter \
  -p 9104:9104 \
  -e DATA_SOURCE_NAME="exporter:TxTCvv8dwp@(10.110.1.107:3306)/" \
  prom/mysqld-exporter \
  --collect.auto_increment.columns	\
  --collect.binlog_size	\
  --collect.engine_innodb_status \
  --collect.global_status	\
  --collect.global_variables \
  --collect.info_schema.clientstats	\
  --collect.info_schema.innodb_metrics \
  --collect.info_schema.innodb_tablespaces \
  --collect.info_schema.innodb_cmp \
  --collect.info_schema.innodb_cmpmem \
  --collect.info_schema.query_response_time \
  --collect.info_schema.replica_host \
  --collect.info_schema.tablestats \
  --collect.info_schema.schemastats \
  --collect.info_schema.userstats \
  --collect.mysql.user \
  --collect.perf_schema.eventsstatements \
  --collect.perf_schema.eventsstatementssum \
  --collect.perf_schema.eventswaits	\
  --collect.perf_schema.file_events	\
  --collect.perf_schema.file_instances \
  --collect.perf_schema.indexiowaits \
  --collect.perf_schema.memory_events	\
  --collect.perf_schema.tableiowaits \
  --collect.perf_schema.tablelocks \
  --collect.perf_schema.replication_group_members \
  --collect.perf_schema.replication_group_member_stats \
  --log-opt tag="{{.ImageName}}|{{.Name}}|{{.ImageFullID}}|{{.FullID}}"
```

# Deploy nginx exporter on Web instance
```
Step 1: Run Nginx container

For Nginx to expose it metrics we have to add stub_status to /metrics in Nginx conf. Example Nginx conf:
Create stub_status.conf in /etc/nginx/conf.d/
---------
server {
    listen       8080;
    server_name  localhost;

    location /metrics {
        stub_status;
    }
}
---------

Step 2: Run Nginx exporter for exporting Nginx metrics to Prometheus metrics format

docker run -d \
--name nginx-exporter \
-p 9113:9113 \
nginx/nginx-prometheus-exporter:0.10.0 \
-nginx.scrape-uri=http://<nginx>:8080/metrics \
--log-opt tag="{{.ImageName}}|{{.Name}}|{{.ImageFullID}}|{{.FullID}}"
```