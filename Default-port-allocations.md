## Core components starting at 9090
* 9090 - Prometheus server
* 9091 - Pushgateway
* 9092 - UNALLOCATED (to avoid collision with Kafka)
* 9093 - Alertmanager

## Exporters starting at 9100
* 9100 - [Node exporter](http://github.com/prometheus/node_exporter)
* 9101 - [HAProxy exporter](http://github.com/prometheus/haproxy_exporter)
* 9102 - [StatsD bridge](http://github.com/prometheus/statsd_bridge): Metrics
* 9103 - [Collectd exporter](http://github.com/prometheus/collectd_exporter)
* 9104 - [MySQLd exporter](http://github.com/prometheus/mysqld_exporter)
* 9105 - [Mesos exporter](http://github.com/prometheus/mesos_exporter)
* 9106 - [CloudWatch exporter](http://github.com/prometheus/cloudwatch_exporter)
* 9107 - [Consul exporter](http://github.com/prometheus/consul_exporter)
* 9108 - [Graphite exporter](http://github.com/prometheus/graphite_exporter): Metrics
* 9109 - [Graphite exporter](http://github.com/prometheus/graphite_exporter): Ingestion
* 9110 - [Blackbox prober](/discordianfish/blackbox_prober)
* 9111 - [Expvar exporter](/docker-infra/expvar_exporter)
* 9112 - [Heka Output Plugin](/docker-infra/heka-prometheus-output)
* 9113 - [Nginx exporter](/discordianfish/nginx_exporter)
* 9125 - [StatsD bridge](http://github.com/prometheus/statsd_bridge): Ingestion
