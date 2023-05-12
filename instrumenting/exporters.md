# 数据导出及相关集成

有许多库和服务可帮助将第三方系统中的现有数据指标导出为 Prometheus 数据指标。这对于无法直接使用 Prometheus 指标(例如 HAProxy 或 Linux系统统计信息)来检测给定系统的状态很有用。

## 第三方数据导出器 <a href="#third-party-exporters" id="third-party-exporters"></a>

其中一些导出器是 [Prometheus GitHub 官方组织](https://github.com/prometheus)的一部分，其中部分导出器被标记为\_官方的\_，其他到初七则由外部贡献和维护。

我们鼓励创建更多的数据指标导出器，但不能审查所有数据导出器的[最佳实践](writing\_exporters.md)。通常，这些导出器托管在 Prometheus GitHub 组织之外。

[数据导出器默认端口](https://github.com/prometheus/prometheus/wiki/Default-port-allocations) Wiki 页面已成为导出器的概览目录，并且可能包括由于功能重叠或仍在开发中而未在此处列出的数据导出器。

[JMX 导出器](https://github.com/prometheus/jmx\_exporter)可以从各种基于 JVM 的应用程序中导出数据，例如 [Kafka](https://kafka.apache.org/)和 [Cassandra](https://cassandra.apache.org/)。

### 数据库 <a href="#databases" id="databases"></a>

* [Aerospike exporter](https://github.com/alicebob/asprom)
* [ClickHouse exporter](https://github.com/f1yegor/clickhouse\_exporter)
* [Consul exporter](https://github.com/prometheus/consul\_exporter) (**官方**)
* [Couchbase exporter](https://github.com/blakelead/couchbase\_exporter)
* [CouchDB exporter](https://github.com/gesellix/couchdb-exporter)
* [ElasticSearch exporter](https://github.com/justwatchcom/elasticsearch\_exporter)
* [EventStore exporter](https://github.com/marcinbudny/eventstore\_exporter)
* [Memcached exporter](https://github.com/prometheus/memcached\_exporter) (**官方**)
* [MongoDB exporter](https://github.com/dcu/mongodb\_exporter)
* [MSSQL server exporter](https://github.com/awaragi/prometheus-mssql-exporter)
* [MySQL router exporter](https://github.com/rluisr/mysqlrouter\_exporter)
* [MySQL server exporter](https://github.com/prometheus/mysqld\_exporter) (**官方**)
* [OpenTSDB Exporter](https://github.com/cloudflare/opentsdb\_exporter)
* [Oracle DB Exporter](https://github.com/iamseth/oracledb\_exporter)
* [PgBouncer exporter](http://git.cbaines.net/prometheus-pgbouncer-exporter/about)
* [PostgreSQL exporter](https://github.com/wrouesnel/postgres\_exporter)
* [Presto exporter](https://github.com/yahoojapan/presto\_exporter)
* [ProxySQL exporter](https://github.com/percona/proxysql\_exporter)
* [RavenDB exporter](https://github.com/marcinbudny/ravendb\_exporter)
* [Redis exporter](https://github.com/oliver006/redis\_exporter)
* [RethinkDB exporter](https://github.com/oliver006/rethinkdb\_exporter)
* [SQL exporter](https://github.com/free/sql\_exporter)
* [Tarantool metric library](https://github.com/tarantool/prometheus)
* [Twemproxy](https://github.com/stuartnelson3/twemproxy\_exporter)

### 硬件相关 <a href="#hardware-related" id="hardware-related"></a>

* [apcupsd exporter](https://github.com/mdlayher/apcupsd\_exporter)
* [BIG-IP exporter](https://github.com/ExpressenAB/bigip\_exporter)
* [Collins exporter](https://github.com/soundcloud/collins\_exporter)
* [Dell Hardware OMSA exporter](https://github.com/galexrt/dellhw\_exporter)
* [IBM Z HMC exporter](https://github.com/zhmcclient/zhmc-prometheus-exporter)
* [IoT Edison exporter](https://github.com/roman-vynar/edison\_exporter)
* [IPMI exporter](https://github.com/soundcloud/ipmi\_exporter)
* [knxd exporter](https://github.com/RichiH/knxd\_exporter)
* [Modbus exporter](https://github.com/RichiH/modbus\_exporter)
* [Netgear Cable Modem Exporter](https://github.com/ickymettle/netgear\_cm\_exporter)
* [Netgear Router exporter](https://github.com/DRuggeri/netgear\_exporter)
* [Node/system metrics exporter](https://github.com/prometheus/node\_exporter) (**官方**)
* [NVIDIA GPU exporter](https://github.com/mindprince/nvidia\_gpu\_prometheus\_exporter)
* [ProSAFE exporter](https://github.com/dalance/prosafe\_exporter)
* [Ubiquiti UniFi exporter](https://github.com/mdlayher/unifi\_exporter)

### 问题跟踪和持续集成 <a href="#issue-trackers-and-continuous-integration" id="issue-trackers-and-continuous-integration"></a>

* [Bamboo exporter](https://github.com/AndreyVMarkelov/bamboo-prometheus-exporter)
* [Bitbucket exporter](https://github.com/AndreyVMarkelov/prom-bitbucket-exporter)
* [Confluence exporter](https://github.com/AndreyVMarkelov/prom-confluence-exporter)
* [Jenkins exporter](https://github.com/lovoo/jenkins\_exporter)
* [JIRA exporter](https://github.com/AndreyVMarkelov/jira-prometheus-exporter)

### 消息系统 <a href="#messaging-systems" id="messaging-systems"></a>

* [Beanstalkd exporter](https://github.com/messagebird/beanstalkd\_exporter)
* [EMQ exporter](https://github.com/nuvo/emq\_exporter)
* [Gearman exporter](https://github.com/bakins/gearman-exporter)
* [IBM MQ exporter](https://github.com/ibm-messaging/mq-metric-samples/tree/master/cmd/mq\_prometheus)
* [Kafka exporter](https://github.com/danielqsj/kafka\_exporter)
* [NATS exporter](https://github.com/nats-io/prometheus-nats-exporter)
* [NSQ exporter](https://github.com/lovoo/nsq\_exporter)
* [Mirth Connect exporter](https://github.com/vynca/mirth\_exporter)
* [MQTT blackbox exporter](https://github.com/inovex/mqtt\_blackbox\_exporter)
* [RabbitMQ exporter](https://github.com/kbudde/rabbitmq\_exporter)
* [RabbitMQ Management Plugin exporter](https://github.com/deadtrickster/prometheus\_rabbitmq\_exporter)
* [RocketMQ exporter](https://github.com/apache/rocketmq-exporter)
* [Solace exporter](https://github.com/dabgmx/solace\_exporter)

### 存储 <a href="#storage" id="storage"></a>

* [Ceph exporter](https://github.com/digitalocean/ceph\_exporter)
* [Ceph RADOSGW exporter](https://github.com/blemmenes/radosgw\_usage\_exporter)
* [Gluster exporter](https://github.com/ofesseler/gluster\_exporter)
* [Hadoop HDFS FSImage exporter](https://github.com/marcelmay/hadoop-hdfs-fsimage-exporter)
* [Lustre exporter](https://github.com/HewlettPackard/lustre\_exporter)
* [ScaleIO exporter](https://github.com/syepes/sio2prom)

### HTTP

* [Apache exporter](https://github.com/Lusitaniae/apache\_exporter)
* [HAProxy exporter](https://github.com/prometheus/haproxy\_exporter) (**官方**)
* [Nginx metric library](https://github.com/knyar/nginx-lua-prometheus)
* [Nginx VTS exporter](https://github.com/hnlq715/nginx-vts-exporter)
* [Passenger exporter](https://github.com/stuartnelson3/passenger\_exporter)
* [Squid exporter](https://github.com/boynux/squid-exporter)
* [Tinyproxy exporter](https://github.com/igzivkov/tinyproxy\_exporter)
* [Varnish exporter](https://github.com/jonnenauha/prometheus\_varnish\_exporter)
* [WebDriver exporter](https://github.com/mattbostock/webdriver\_exporter)

### APIs

* [AWS ECS exporter](https://github.com/slok/ecs-exporter)
* [AWS Health exporter](https://github.com/Jimdo/aws-health-exporter)
* [AWS SQS exporter](https://github.com/jmal98/sqs\_exporter)
* [Azure Health exporter](https://github.com/FXinnovation/azure-health-exporter)
* [BigBlueButton](https://github.com/greenstatic/bigbluebutton-exporter)
* [Cloudflare exporter](https://github.com/wehkamp/docker-prometheus-cloudflare-exporter)
* [DigitalOcean exporter](https://github.com/metalmatze/digitalocean\_exporter)
* [Docker Cloud exporter](https://github.com/infinityworksltd/docker-cloud-exporter)
* [Docker Hub exporter](https://github.com/infinityworksltd/docker-hub-exporter)
* [GitHub exporter](https://github.com/infinityworksltd/github-exporter)
* [InstaClustr exporter](https://github.com/fcgravalos/instaclustr\_exporter)
* [Mozilla Observatory exporter](https://github.com/Jimdo/observatory-exporter)
* [OpenWeatherMap exporter](https://github.com/RichiH/openweathermap\_exporter)
* [Pagespeed exporter](https://github.com/foomo/pagespeed\_exporter)
* [Rancher exporter](https://github.com/infinityworksltd/prometheus-rancher-exporter)
* [Speedtest exporter](https://github.com/nlamirault/speedtest\_exporter)
* [Tankerkönig API Exporter](https://github.com/lukasmalkmus/tankerkoenig\_exporter)

### 日志 <a href="#logging" id="logging"></a>

* [Fluentd exporter](https://github.com/V3ckt0r/fluentd\_exporter)
* [Google's mtail log data extractor](https://github.com/google/mtail)
* [Grok exporter](https://github.com/fstab/grok\_exporter)

### 其它监控系统 <a href="#other-monitoring-systems" id="other-monitoring-systems"></a>

* [Akamai Cloudmonitor exporter](https://github.com/ExpressenAB/cloudmonitor\_exporter)
* [Alibaba Cloudmonitor exporter](https://github.com/aylei/aliyun-exporter)
* [AWS CloudWatch exporter](https://github.com/prometheus/cloudwatch\_exporter) (**官方**)
* [Azure Monitor exporter](https://github.com/RobustPerception/azure\_metrics\_exporter)
* [Cloud Foundry Firehose exporter](https://github.com/cloudfoundry-community/firehose\_exporter)
* [Collectd exporter](https://github.com/prometheus/collectd\_exporter) (**官方**)
* [Google Stackdriver exporter](https://github.com/frodenas/stackdriver\_exporter)
* [Graphite exporter](https://github.com/prometheus/graphite\_exporter) (**官方**)
* [Heka dashboard exporter](https://github.com/docker-infra/heka\_exporter)
* [Heka exporter](https://github.com/imgix/heka\_exporter)
* [Huawei Cloudeye exporter](https://github.com/huaweicloud/cloudeye-exporter)
* [InfluxDB exporter](https://github.com/prometheus/influxdb\_exporter) (**官方**)
* [JavaMelody exporter](https://github.com/fschlag/javamelody-prometheus-exporter)
* [JMX exporter](https://github.com/prometheus/jmx\_exporter) (**官方**)
* [Munin exporter](https://github.com/pvdh/munin\_exporter)
* [Nagios / Naemon exporter](https://github.com/Griesbacher/Iapetos)
* [New Relic exporter](https://github.com/mrf/newrelic\_exporter)
* [NRPE exporter](https://github.com/robustperception/nrpe\_exporter)
* [Osquery exporter](https://github.com/zwopir/osquery\_exporter)
* [OTC CloudEye exporter](https://github.com/tiagoReichert/otc-cloudeye-prometheus-exporter)
* [Pingdom exporter](https://github.com/giantswarm/prometheus-pingdom-exporter)
* [scollector exporter](https://github.com/tgulacsi/prometheus\_scollector)
* [Sensu exporter](https://github.com/reachlin/sensu\_exporter)
* [SNMP exporter](https://github.com/prometheus/snmp\_exporter) (**官方**)
* [StatsD exporter](https://github.com/prometheus/statsd\_exporter) (**官方**)
* [TencentCloud monitor exporter](https://github.com/tencentyun/tencentcloud-exporter)
* [ThousandEyes exporter](https://github.com/sapcc/1000eyes\_exporter)

### 杂项 <a href="#miscellaneous" id="miscellaneous"></a>

* [ACT Fibernet Exporter](https://git.captnemo.in/nemo/prometheus-act-exporter)
* [BIND exporter](https://github.com/prometheus-community/bind\_exporter)
* [Bitcoind exporter](https://github.com/LePetitBloc/bitcoind-exporter)
* [Blackbox exporter](https://github.com/prometheus/blackbox\_exporter) (**官方**)
* [BOSH exporter](https://github.com/cloudfoundry-community/bosh\_exporter)
* [cAdvisor](https://github.com/google/cadvisor)
* [Cachet exporter](https://github.com/ContaAzul/cachet\_exporter)
* [ccache exporter](https://github.com/virtualtam/ccache\_exporter)
* [Dovecot exporter](https://github.com/kumina/dovecot\_exporter)
* [Dnsmasq exporter](https://github.com/google/dnsmasq\_exporter)
* [eBPF exporter](https://github.com/cloudflare/ebpf\_exporter)
* [Ethereum Client exporter](https://github.com/31z4/ethereum-prometheus-exporter)
* [JFrog Artifactory Exporter](https://github.com/peimanja/artifactory\_exporter)
* [Hostapd Exporter](https://bitbucket.i2cat.net/users/miguel\_catalan/repos/hostapd\_prometheus\_exporter)
* [IRCd exporter](https://github.com/dgl/ircd\_exporter)
* [Linux HA ClusterLabs exporter](https://github.com/ClusterLabs/ha\_cluster\_exporter)
* [JMeter plugin](https://github.com/johrstrom/jmeter-prometheus-plugin)
* [Kannel exporter](https://github.com/apostvav/kannel\_exporter)
* [Kemp LoadBalancer exporter](https://github.com/giantswarm/prometheus-kemp-exporter)
* [Kibana Exporter](https://github.com/pjhampton/kibana-prometheus-exporter)
* [kube-state-metrics](https://github.com/kubernetes/kube-state-metrics)
* [Locust Exporter](https://github.com/ContainerSolutions/locust\_exporter)
* [Meteor JS web framework exporter](https://atmospherejs.com/sevki/prometheus-exporter)
* [Minecraft exporter module](https://github.com/Baughn/PrometheusIntegration)
* [OpenStack exporter](https://github.com/openstack-exporter/openstack-exporter)
* [OpenStack blackbox exporter](https://github.com/infraly/openstack\_client\_exporter)
* [oVirt exporter](https://github.com/czerwonk/ovirt\_exporter)
* [Pact Broker exporter](https://github.com/ContainerSolutions/pactbroker\_exporter)
* [PHP-FPM exporter](https://github.com/bakins/php-fpm-exporter)
* [PowerDNS exporter](https://github.com/ledgr/powerdns\_exporter)
* [Process exporter](https://github.com/ncabatoff/process-exporter)
* [rTorrent exporter](https://github.com/mdlayher/rtorrent\_exporter)
* [SABnzbd exporter](https://github.com/msroest/sabnzbd\_exporter)
* [Script exporter](https://github.com/adhocteam/script\_exporter)
* [Shield exporter](https://github.com/cloudfoundry-community/shield\_exporter)
* [Smokeping prober](https://github.com/SuperQ/smokeping\_prober)
* [SMTP/Maildir MDA blackbox prober](https://github.com/cherti/mailexporter)
* [SoftEther exporter](https://github.com/dalance/softether\_exporter)
* [Transmission exporter](https://github.com/metalmatze/transmission-exporter)
* [Unbound exporter](https://github.com/kumina/unbound\_exporter)
* [WireGuard exporter](https://github.com/MindFlavor/prometheus\_wireguard\_exporter)
* [Xen exporter](https://github.com/lovoo/xenstats\_exporter)

实现新的 Prometheus 数据导出器时，请遵循有关编写[导出器的准则](https://prometheus.io/docs/instrumenting/writing\_exporters)。也请考虑咨询[开发邮件列表](https://groups.google.com/forum/#!forum/prometheus-developers)。我们很乐意就如何使您的导出器尽可能有用和一致提供建议

## 公开 Prometheus 数据指标的软件 <a href="#software-exposing-prometheus-metrics" id="software-exposing-prometheus-metrics"></a>

一些第三方软件以 Prometheus 格式公开指标，因此不需要单独的导出器:

* [App Connect Enterprise](https://github.com/ot4i/ace-docker)
* [Ballerina](https://ballerina.io/)
* [BFE](https://github.com/baidu/bfe)
* [Ceph](http://docs.ceph.com/docs/master/mgr/prometheus/)
* [CockroachDB](https://www.cockroachlabs.com/docs/stable/monitoring-and-alerting.html#prometheus-endpoint)
* [Collectd](https://collectd.org/wiki/index.php/Plugin:Write\_Prometheus)
* [Concourse](https://concourse-ci.org/)
* [CRG Roller Derby Scoreboard](https://github.com/rollerderby/scoreboard) (**direct**)
* [Diffusion](https://docs.pushtechnology.com/docs/latest/manual/html/administratorguide/systemmanagement/r\_statistics.html)
* [Docker Daemon](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-metrics)
* [Doorman](https://github.com/youtube/doorman) (**direct**)
* [Envoy](https://www.envoyproxy.io/docs/envoy/latest/operations/admin.html#get--stats?format=prometheus)
* [Etcd](https://github.com/coreos/etcd) (**direct**)
* [Flink](https://github.com/apache/flink)
* [FreeBSD Kernel](https://www.freebsd.org/cgi/man.cgi?query=prometheus\_sysctl\_exporter\&apropos=0\&sektion=8\&manpath=FreeBSD+12-current\&arch=default\&format=html)
* [Grafana](http://docs.grafana.org/administration/metrics/)
* [JavaMelody](https://github.com/javamelody/javamelody/wiki/UserGuideAdvanced#exposing-metrics-to-prometheus)
* [Kong](https://github.com/Kong/kong-plugin-prometheus)
* [Kubernetes](https://github.com/kubernetes/kubernetes) (**direct**)
* [Linkerd](https://github.com/BuoyantIO/linkerd)
* [mgmt](https://github.com/purpleidea/mgmt/blob/master/docs/prometheus.md)
* [MidoNet](https://github.com/midonet/midonet)
* [midonet-kubernetes](https://github.com/midonet/midonet-kubernetes) (**direct**)
* [Minio](https://github.com/minio/minio)
* [Netdata](https://github.com/firehol/netdata)
* [Pretix](https://pretix.eu/)
* [Quobyte](https://www.quobyte.com/) (**direct**)
* [RabbitMQ](https://rabbitmq.com/prometheus.html)
* [RobustIRC](https://robustirc.net/)
* [ScyllaDB](https://github.com/scylladb/scylla)
* [Skipper](https://github.com/zalando/skipper)
* [SkyDNS](https://github.com/skynetservices/skydns) (**direct**)
* [Telegraf](https://github.com/influxdata/telegraf/tree/master/plugins/outputs/prometheus\_client)
* [Traefik](https://github.com/containous/traefik)
* [VerneMQ](https://github.com/vernemq/vernemq)
* [Weave Flux](https://github.com/weaveworks/flux)
* [Xandikos](https://www.xandikos.org/) (**direct**)
* [Zipkin](https://github.com/openzipkin/zipkin/tree/master/zipkin-server#metrics)

标有 _direct_ 的软件也可以直接用 Prometheus 客户端库进行检测。

## 其它第三方工具 <a href="#other-third-party-utilities" id="other-third-party-utilities"></a>

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
