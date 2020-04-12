---
title: 数据导出及相关集成
---

# 数据导出及相关整合

有许多库和服务可帮助将第三方系统中的现有数据指标导出为 Prometheus 数据指标。这对于无法直接使用 Prometheus 指标\(例如 HAProxy 或 Linux系统统计信息\)来检测给定系统的状态很有用。

## 第三方数据导出器 <a id="third-party-exporters"></a>

其中一些导出器是 [Prometheus GitHub 官方组织](https://github.com/prometheus)的一部分，其中部分导出器被标记为_官方的_，其他到初七则由外部贡献和维护。

我们鼓励创建更多的数据指标导出器，但不能审查所有数据导出器的[最佳实践](https://prometheus.io/docs/instrumenting/writing_exporters/)。通常，这些导出器托管在 Prometheus GitHub 组织之外。

[数据导出器默认端口](https://github.com/prometheus/prometheus/wiki/Default-port-allocations) Wiki 页面已成为导出器的概览目录，并且可能包括由于功能重叠或仍在开发中而未在此处列出的数据导出器。

[JMX 导出器](https://github.com/prometheus/jmx_exporter)可以从各种基于 JVM 的应用程序中导出数据，例如 [Kafka](https://kafka.apache.org/)和 [Cassandra](https://cassandra.apache.org/)。

### 数据库 <a id="databases"></a>

* [Aerospike exporter](https://github.com/alicebob/asprom)
* [ClickHouse exporter](https://github.com/f1yegor/clickhouse_exporter)
* [Consul exporter](https://github.com/prometheus/consul_exporter) \(**官方**\)
* [Couchbase exporter](https://github.com/blakelead/couchbase_exporter)
* [CouchDB exporter](https://github.com/gesellix/couchdb-exporter)
* [ElasticSearch exporter](https://github.com/justwatchcom/elasticsearch_exporter)
* [EventStore exporter](https://github.com/marcinbudny/eventstore_exporter)
* [Memcached exporter](https://github.com/prometheus/memcached_exporter) \(**官方**\)
* [MongoDB exporter](https://github.com/dcu/mongodb_exporter)
* [MSSQL server exporter](https://github.com/awaragi/prometheus-mssql-exporter)
* [MySQL router exporter](https://github.com/rluisr/mysqlrouter_exporter)
* [MySQL server exporter](https://github.com/prometheus/mysqld_exporter) \(**官方**\)
* [OpenTSDB Exporter](https://github.com/cloudflare/opentsdb_exporter)
* [Oracle DB Exporter](https://github.com/iamseth/oracledb_exporter)
* [PgBouncer exporter](http://git.cbaines.net/prometheus-pgbouncer-exporter/about)
* [PostgreSQL exporter](https://github.com/wrouesnel/postgres_exporter)
* [Presto exporter](https://github.com/yahoojapan/presto_exporter)
* [ProxySQL exporter](https://github.com/percona/proxysql_exporter)
* [RavenDB exporter](https://github.com/marcinbudny/ravendb_exporter)
* [Redis exporter](https://github.com/oliver006/redis_exporter)
* [RethinkDB exporter](https://github.com/oliver006/rethinkdb_exporter)
* [SQL exporter](https://github.com/free/sql_exporter)
* [Tarantool metric library](https://github.com/tarantool/prometheus)
* [Twemproxy](https://github.com/stuartnelson3/twemproxy_exporter)

### 硬件相关 <a id="hardware-related"></a>

* [apcupsd exporter](https://github.com/mdlayher/apcupsd_exporter)
* [BIG-IP exporter](https://github.com/ExpressenAB/bigip_exporter)
* [Collins exporter](https://github.com/soundcloud/collins_exporter)
* [Dell Hardware OMSA exporter](https://github.com/galexrt/dellhw_exporter)
* [IBM Z HMC exporter](https://github.com/zhmcclient/zhmc-prometheus-exporter)
* [IoT Edison exporter](https://github.com/roman-vynar/edison_exporter)
* [IPMI exporter](https://github.com/soundcloud/ipmi_exporter)
* [knxd exporter](https://github.com/RichiH/knxd_exporter)
* [Modbus exporter](https://github.com/RichiH/modbus_exporter)
* [Netgear Cable Modem Exporter](https://github.com/ickymettle/netgear_cm_exporter)
* [Netgear Router exporter](https://github.com/DRuggeri/netgear_exporter)
* [Node/system metrics exporter](https://github.com/prometheus/node_exporter) \(**官方**\)
* [NVIDIA GPU exporter](https://github.com/mindprince/nvidia_gpu_prometheus_exporter)
* [ProSAFE exporter](https://github.com/dalance/prosafe_exporter)
* [Ubiquiti UniFi exporter](https://github.com/mdlayher/unifi_exporter)

### 问题跟踪和持续集成 <a id="issue-trackers-and-continuous-integration"></a>

* [Bamboo exporter](https://github.com/AndreyVMarkelov/bamboo-prometheus-exporter)
* [Bitbucket exporter](https://github.com/AndreyVMarkelov/prom-bitbucket-exporter)
* [Confluence exporter](https://github.com/AndreyVMarkelov/prom-confluence-exporter)
* [Jenkins exporter](https://github.com/lovoo/jenkins_exporter)
* [JIRA exporter](https://github.com/AndreyVMarkelov/jira-prometheus-exporter)

### 消息系统 <a id="messaging-systems"></a>

* [Beanstalkd exporter](https://github.com/messagebird/beanstalkd_exporter)
* [EMQ exporter](https://github.com/nuvo/emq_exporter)
* [Gearman exporter](https://github.com/bakins/gearman-exporter)
* [IBM MQ exporter](https://github.com/ibm-messaging/mq-metric-samples/tree/master/cmd/mq_prometheus)
* [Kafka exporter](https://github.com/danielqsj/kafka_exporter)
* [NATS exporter](https://github.com/nats-io/prometheus-nats-exporter)
* [NSQ exporter](https://github.com/lovoo/nsq_exporter)
* [Mirth Connect exporter](https://github.com/vynca/mirth_exporter)
* [MQTT blackbox exporter](https://github.com/inovex/mqtt_blackbox_exporter)
* [RabbitMQ exporter](https://github.com/kbudde/rabbitmq_exporter)
* [RabbitMQ Management Plugin exporter](https://github.com/deadtrickster/prometheus_rabbitmq_exporter)
* [RocketMQ exporter](https://github.com/apache/rocketmq-exporter)
* [Solace exporter](https://github.com/dabgmx/solace_exporter)

### 存储 <a id="storage"></a>

* [Ceph exporter](https://github.com/digitalocean/ceph_exporter)
* [Ceph RADOSGW exporter](https://github.com/blemmenes/radosgw_usage_exporter)
* [Gluster exporter](https://github.com/ofesseler/gluster_exporter)
* [Hadoop HDFS FSImage exporter](https://github.com/marcelmay/hadoop-hdfs-fsimage-exporter)
* [Lustre exporter](https://github.com/HewlettPackard/lustre_exporter)
* [ScaleIO exporter](https://github.com/syepes/sio2prom)

### HTTP

* [Apache exporter](https://github.com/Lusitaniae/apache_exporter)
* [HAProxy exporter](https://github.com/prometheus/haproxy_exporter) \(**官方**\)
* [Nginx metric library](https://github.com/knyar/nginx-lua-prometheus)
* [Nginx VTS exporter](https://github.com/hnlq715/nginx-vts-exporter)
* [Passenger exporter](https://github.com/stuartnelson3/passenger_exporter)
* [Squid exporter](https://github.com/boynux/squid-exporter)
* [Tinyproxy exporter](https://github.com/igzivkov/tinyproxy_exporter)
* [Varnish exporter](https://github.com/jonnenauha/prometheus_varnish_exporter)
* [WebDriver exporter](https://github.com/mattbostock/webdriver_exporter)

### APIs

* [AWS ECS exporter](https://github.com/slok/ecs-exporter)
* [AWS Health exporter](https://github.com/Jimdo/aws-health-exporter)
* [AWS SQS exporter](https://github.com/jmal98/sqs_exporter)
* [Azure Health exporter](https://github.com/FXinnovation/azure-health-exporter)
* [BigBlueButton](https://github.com/greenstatic/bigbluebutton-exporter)
* [Cloudflare exporter](https://github.com/wehkamp/docker-prometheus-cloudflare-exporter)
* [DigitalOcean exporter](https://github.com/metalmatze/digitalocean_exporter)
* [Docker Cloud exporter](https://github.com/infinityworksltd/docker-cloud-exporter)
* [Docker Hub exporter](https://github.com/infinityworksltd/docker-hub-exporter)
* [GitHub exporter](https://github.com/infinityworksltd/github-exporter)
* [InstaClustr exporter](https://github.com/fcgravalos/instaclustr_exporter)
* [Mozilla Observatory exporter](https://github.com/Jimdo/observatory-exporter)
* [OpenWeatherMap exporter](https://github.com/RichiH/openweathermap_exporter)
* [Pagespeed exporter](https://github.com/foomo/pagespeed_exporter)
* [Rancher exporter](https://github.com/infinityworksltd/prometheus-rancher-exporter)
* [Speedtest exporter](https://github.com/nlamirault/speedtest_exporter)
* [Tankerkönig API Exporter](https://github.com/lukasmalkmus/tankerkoenig_exporter)

### 日志 <a id="logging"></a>

* [Fluentd exporter](https://github.com/V3ckt0r/fluentd_exporter)
* [Google's mtail log data extractor](https://github.com/google/mtail)
* [Grok exporter](https://github.com/fstab/grok_exporter)

### 其它监控系统 <a id="other-monitoring-systems"></a>

* [Akamai Cloudmonitor exporter](https://github.com/ExpressenAB/cloudmonitor_exporter)
* [Alibaba Cloudmonitor exporter](https://github.com/aylei/aliyun-exporter)
* [AWS CloudWatch exporter](https://github.com/prometheus/cloudwatch_exporter) \(**官方**\)
* [Azure Monitor exporter](https://github.com/RobustPerception/azure_metrics_exporter)
* [Cloud Foundry Firehose exporter](https://github.com/cloudfoundry-community/firehose_exporter)
* [Collectd exporter](https://github.com/prometheus/collectd_exporter) \(**官方**\)
* [Google Stackdriver exporter](https://github.com/frodenas/stackdriver_exporter)
* [Graphite exporter](https://github.com/prometheus/graphite_exporter) \(**官方**\)
* [Heka dashboard exporter](https://github.com/docker-infra/heka_exporter)
* [Heka exporter](https://github.com/imgix/heka_exporter)
* [Huawei Cloudeye exporter](https://github.com/huaweicloud/cloudeye-exporter)
* [InfluxDB exporter](https://github.com/prometheus/influxdb_exporter) \(**官方**\)
* [JavaMelody exporter](https://github.com/fschlag/javamelody-prometheus-exporter)
* [JMX exporter](https://github.com/prometheus/jmx_exporter) \(**官方**\)
* [Munin exporter](https://github.com/pvdh/munin_exporter)
* [Nagios / Naemon exporter](https://github.com/Griesbacher/Iapetos)
* [New Relic exporter](https://github.com/mrf/newrelic_exporter)
* [NRPE exporter](https://github.com/robustperception/nrpe_exporter)
* [Osquery exporter](https://github.com/zwopir/osquery_exporter)
* [OTC CloudEye exporter](https://github.com/tiagoReichert/otc-cloudeye-prometheus-exporter)
* [Pingdom exporter](https://github.com/giantswarm/prometheus-pingdom-exporter)
* [scollector exporter](https://github.com/tgulacsi/prometheus_scollector)
* [Sensu exporter](https://github.com/reachlin/sensu_exporter)
* [SNMP exporter](https://github.com/prometheus/snmp_exporter) \(**官方**\)
* [StatsD exporter](https://github.com/prometheus/statsd_exporter) \(**官方**\)
* [TencentCloud monitor exporter](https://github.com/tencentyun/tencentcloud-exporter)
* [ThousandEyes exporter](https://github.com/sapcc/1000eyes_exporter)

### 杂项 <a id="miscellaneous"></a>

* [ACT Fibernet Exporter](https://git.captnemo.in/nemo/prometheus-act-exporter)
* [BIND exporter](https://github.com/prometheus-community/bind_exporter)
* [Bitcoind exporter](https://github.com/LePetitBloc/bitcoind-exporter)
* [Blackbox exporter](https://github.com/prometheus/blackbox_exporter) \(**官方**\)
* [BOSH exporter](https://github.com/cloudfoundry-community/bosh_exporter)
* [cAdvisor](https://github.com/google/cadvisor)
* [Cachet exporter](https://github.com/ContaAzul/cachet_exporter)
* [ccache exporter](https://github.com/virtualtam/ccache_exporter)
* [Dovecot exporter](https://github.com/kumina/dovecot_exporter)
* [Dnsmasq exporter](https://github.com/google/dnsmasq_exporter)
* [eBPF exporter](https://github.com/cloudflare/ebpf_exporter)
* [Ethereum Client exporter](https://github.com/31z4/ethereum-prometheus-exporter)
* [JFrog Artifactory Exporter](https://github.com/peimanja/artifactory_exporter)
* [Hostapd Exporter](https://bitbucket.i2cat.net/users/miguel_catalan/repos/hostapd_prometheus_exporter)
* [IRCd exporter](https://github.com/dgl/ircd_exporter)
* [Linux HA ClusterLabs exporter](https://github.com/ClusterLabs/ha_cluster_exporter)
* [JMeter plugin](https://github.com/johrstrom/jmeter-prometheus-plugin)
* [Kannel exporter](https://github.com/apostvav/kannel_exporter)
* [Kemp LoadBalancer exporter](https://github.com/giantswarm/prometheus-kemp-exporter)
* [Kibana Exporter](https://github.com/pjhampton/kibana-prometheus-exporter)
* [kube-state-metrics](https://github.com/kubernetes/kube-state-metrics)
* [Locust Exporter](https://github.com/ContainerSolutions/locust_exporter)
* [Meteor JS web framework exporter](https://atmospherejs.com/sevki/prometheus-exporter)
* [Minecraft exporter module](https://github.com/Baughn/PrometheusIntegration)
* [OpenStack exporter](https://github.com/openstack-exporter/openstack-exporter)
* [OpenStack blackbox exporter](https://github.com/infraly/openstack_client_exporter)
* [oVirt exporter](https://github.com/czerwonk/ovirt_exporter)
* [Pact Broker exporter](https://github.com/ContainerSolutions/pactbroker_exporter)
* [PHP-FPM exporter](https://github.com/bakins/php-fpm-exporter)
* [PowerDNS exporter](https://github.com/ledgr/powerdns_exporter)
* [Process exporter](https://github.com/ncabatoff/process-exporter)
* [rTorrent exporter](https://github.com/mdlayher/rtorrent_exporter)
* [SABnzbd exporter](https://github.com/msroest/sabnzbd_exporter)
* [Script exporter](https://github.com/adhocteam/script_exporter)
* [Shield exporter](https://github.com/cloudfoundry-community/shield_exporter)
* [Smokeping prober](https://github.com/SuperQ/smokeping_prober)
* [SMTP/Maildir MDA blackbox prober](https://github.com/cherti/mailexporter)
* [SoftEther exporter](https://github.com/dalance/softether_exporter)
* [Transmission exporter](https://github.com/metalmatze/transmission-exporter)
* [Unbound exporter](https://github.com/kumina/unbound_exporter)
* [WireGuard exporter](https://github.com/MindFlavor/prometheus_wireguard_exporter)
* [Xen exporter](https://github.com/lovoo/xenstats_exporter)

实现新的 Prometheus 数据导出器时，请遵循有关编写[导出器的准则](https://prometheus.io/docs/instrumenting/writing_exporters)。也请考虑咨询[开发邮件列表](https://groups.google.com/forum/#!forum/prometheus-developers)。我们很乐意就如何使您的导出器尽可能有用和一致提供建议

## 公开 Prometheus 数据指标的软件 <a id="software-exposing-prometheus-metrics"></a>

一些第三方软件以 Prometheus 格式公开指标，因此不需要单独的导出器:

* [App Connect Enterprise](https://github.com/ot4i/ace-docker)
* [Ballerina](https://ballerina.io/)
* [BFE](https://github.com/baidu/bfe)
* [Ceph](http://docs.ceph.com/docs/master/mgr/prometheus/)
* [CockroachDB](https://www.cockroachlabs.com/docs/stable/monitoring-and-alerting.html#prometheus-endpoint)
* [Collectd](https://collectd.org/wiki/index.php/Plugin:Write_Prometheus)
* [Concourse](https://concourse-ci.org/)
* [CRG Roller Derby Scoreboard](https://github.com/rollerderby/scoreboard) \(**direct**\)
* [Diffusion](https://docs.pushtechnology.com/docs/latest/manual/html/administratorguide/systemmanagement/r_statistics.html)
* [Docker Daemon](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-metrics)
* [Doorman](https://github.com/youtube/doorman) \(**direct**\)
* [Envoy](https://www.envoyproxy.io/docs/envoy/latest/operations/admin.html#get--stats?format=prometheus)
* [Etcd](https://github.com/coreos/etcd) \(**direct**\)
* [Flink](https://github.com/apache/flink)
* [FreeBSD Kernel](https://www.freebsd.org/cgi/man.cgi?query=prometheus_sysctl_exporter&apropos=0&sektion=8&manpath=FreeBSD+12-current&arch=default&format=html)
* [Grafana](http://docs.grafana.org/administration/metrics/)
* [JavaMelody](https://github.com/javamelody/javamelody/wiki/UserGuideAdvanced#exposing-metrics-to-prometheus)
* [Kong](https://github.com/Kong/kong-plugin-prometheus)
* [Kubernetes](https://github.com/kubernetes/kubernetes) \(**direct**\)
* [Linkerd](https://github.com/BuoyantIO/linkerd)
* [mgmt](https://github.com/purpleidea/mgmt/blob/master/docs/prometheus.md)
* [MidoNet](https://github.com/midonet/midonet)
* [midonet-kubernetes](https://github.com/midonet/midonet-kubernetes) \(**direct**\)
* [Minio](https://github.com/minio/minio)
* [Netdata](https://github.com/firehol/netdata)
* [Pretix](https://pretix.eu/)
* [Quobyte](https://www.quobyte.com/) \(**direct**\)
* [RabbitMQ](https://rabbitmq.com/prometheus.html)
* [RobustIRC](https://robustirc.net/)
* [ScyllaDB](https://github.com/scylladb/scylla)
* [Skipper](https://github.com/zalando/skipper)
* [SkyDNS](https://github.com/skynetservices/skydns) \(**direct**\)
* [Telegraf](https://github.com/influxdata/telegraf/tree/master/plugins/outputs/prometheus_client)
* [Traefik](https://github.com/containous/traefik)
* [VerneMQ](https://github.com/vernemq/vernemq)
* [Weave Flux](https://github.com/weaveworks/flux)
* [Xandikos](https://www.xandikos.org/) \(**direct**\)
* [Zipkin](https://github.com/openzipkin/zipkin/tree/master/zipkin-server#metrics)

标有 _direct_ 的软件也可以直接用 Prometheus 客户端库进行检测。

## 其它第三方工具 <a id="other-third-party-utilities"></a>

本部分列出了可帮助您使用某种语言来编写代码的库和其他实用程序。它们本身不是 Prometheus 客户端库，而是使用内部的常规 Prometheus 客户端库之一。对于所有独立维护的软件，我们无法审查所有软件以获得最佳实践。

* Clojure: [iapetos](https://github.com/clj-commons/iapetos)
* Go: [go-metrics instrumentation library](https://github.com/armon/go-metrics)
* Go: [gokit](https://github.com/peterbourgon/gokit)
* Go: [prombolt](https://github.com/mdlayher/prombolt)
* Java/JVM: [EclipseLink metrics collector](https://github.com/VitaNuova/eclipselinkexporter)
* Java/JVM: [Hystrix metrics publisher](https://github.com/ahus1/prometheus-hystrix)
* Java/JVM: [Jersey metrics collector](https://github.com/VitaNuova/jerseyexporter)
* Java/JVM: [Micrometer Prometheus Registry](https://micrometer.io/docs/registry/prometheus)
* Python-Django: [django-prometheus](https://github.com/korfuri/django-prometheus)
* Node.js: [swagger-stats](https://github.com/slanatech/swagger-stats)

