[TOC]

### Prometheus & grafana & exporter
[Prometheus Docs](https://prometheus.io/docs/introduction/overview/ 'Prometheus Docs')
[Alertmanager Rules](https://awesome-prometheus-alerts.grep.to/ 'Alertmanager Rules')
#### node_exporter
```shell script
docker run -d --restart=always \
  --name node_exporter \
  --net="host" \
  --pid="host" \
  -v "/:/host:ro,rslave" \
  prom/node-exporter \
  --path.rootfs=/host
  --collector.filesystem.ignored-mount-points "^/(sys|proc|dev|host|etc)($|/)"

docker run -d --name node_exporter \
--restart=always \
--net="host" \
--pid="host" \
-v "/proc:/host/proc:ro" \
-v "/sys:/host/sys:ro" \
-v "/:/rootfs:ro" \
prom/node-exporter \
--path.procfs=/host/proc \
--path.rootfs=/rootfs \
--path.sysfs=/host/sys \
--collector.filesystem.ignored-mount-points='^/(sys|proc|dev|host|etc)($$|/)'
```

prometheus
```shell script
docker run -d --restart=always \
--name=prometheus \
-p 9090:9090 \
-v /dbdata/prometheus/config/:/etc/prometheus/ \
-v /dbdata/prometheus/data:/prometheus \
-v /etc/localtime:/etc/localtime:ro \
prom/prometheus \
--config.file=/etc/prometheus/prometheus.yml \
--storage.tsdb.path=/prometheus \
--web.console.libraries=/etc/prometheus/console_libraries \
--web.console.templates=/etc/prometheus/consoles \
--storage.tsdb.retention.time=200h \
--web.enable-lifecycle \
--web.enable-admin-api
```
prometheus.yml
```yml
# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).
  external_labels:
    host: 'linux-jd-master'

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
       - 192.16.0.89:9093
       - 192.16.0.90:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
 - "/etc/prometheus/hoststats-alert.rules"
 - "/etc/prometheus/instancestats-alert.rules"
 - "/etc/prometheus/*.rules"
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
     - targets: ['192.16.0.90:9090']

  - job_name: 'node-exporter'
    static_configs:
     - targets: ['192.16.0.89:9100','192.16.0.90:9100']

  - job_name: 'cadvisor'
    static_configs:
     - targets: ['192.16.0.89:9380','192.16.0.90:9380']

  - job_name: 'mysql-exporter'
    static_configs:
     - targets: ['192.16.0.89:9104','192.16.0.90:9104']

  - job_name: 'redis'
    static_configs:
     - targets: 
       - redis://192.16.0.89:6379
       - redis://192.16.0.90:6379
    metrics_path: /scrape
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 192.16.0.89:9121

  - job_name: 'redis-exporter'
    static_configs:
     - targets: ['192.16.0.89:9121']

  #- job_name: 'rabbitmq-exporter'
  #  static_configs:
  #   - targets: ['192.16.0.89:9419']

  - job_name: 'rabbitmq'
    static_configs:
     - targets: ['172.18.0.2:15692']

  - job_name: 'jenkins'
    bearer_token: 'RHsPYW79iscREXL_SF-Bo9tkEHFOQoSJebjB6wdBMjex6TZPjF1yC9ElXhxsJ93O'
    metrics_path: /prometheus
    scheme: http
    static_configs:
     - targets: ['192.16.0.89:9369']

  - job_name: 'tomcat'
    static_configs:
     - targets: ['172.18.0.4:8180']

  - job_name: 'pony-face'
    metrics_path: '/actuator/prometheus'
    static_configs:
     - targets: ['192.16.0.89:10000','192.16.0.90:10000']

  - job_name: 'pony-notify'
    metrics_path: '/actuator/prometheus'
    static_configs:
     - targets: ['192.16.0.89:9527','192.16.0.90:9527']

  #- job_name: 'alertmanager'
  #  static_configs:
  #   - targets: ['192.16.0.89:10005']
 

  - job_name: 'blackbox-http'
    metrics_path: /probe
    params: 
      module: [http_2xx]
    static_configs: 
     - targets:
       - https://jd.jdgczxc.com:18888/ponysafety3
       - https://jd.jdgczxc.com:443/ponysafety3
       - http://192.16.0.89:8080/ponysafety2
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 192.16.0.89:9115

  - job_name: 'blackbox-tcp'
    metrics_path: /probe
    params:
      module: [tcp_connect_example]
    static_configs:
      - targets:
        - 61.164.59.206:16130
        - 61.164.59.206:12400
        - 61.164.59.206:12401
        - 61.164.59.206:12402
        - 61.164.59.206:12405
        - 61.164.59.206:12500
        - 61.164.59.206:17777
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 192.16.0.89:9115

  - job_name: 'blackbox-icpm'
    metrics_path: /probe
    params:
      module: [icmp]
    static_configs:
      - targets:
        - 192.16.0.90
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 192.16.0.89:9115
```
hoststats-alert.rules
```yml
groups: 
- name: hostStatsAlert
  rules:
  - alert: hostCpuUsageAlert
    expr: sum(avg without (cpu)(irate(node_cpu_seconds_total{mode!='idle'}[5m]))) by (instance) > 0.8
    for: 1m
    labels:
      severity: page
      dingAt: 18892621587
    annotations:
      summary: "Instance {{ $labels.instance }} CPU usgae high"
      description: "{{ $labels.instance }} CPU usage above 80% (current value: {{ $value }})"
  - alert: hostMemUsageAlert
    expr: (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes)/node_memory_MemTotal_bytes > 0.8
    for: 1m
    labels:
      severity: page
      dingAt: 18892621587
    annotations:
      summary: "Instance {{ $labels.instance }} MEM usgae high"
      description: "{{ $labels.instance }} MEM usage above 80% (current value: {{ $value }})"
  - alert: hostDiskUsageAlert
    expr: 1 - (node_filesystem_avail_bytes{fstype=~"ext4|xfs"} / node_filesystem_size_bytes{fstype=~"ext4|xfs"}) > 0.8
    for: 1m
    labels:
      severity: page
      dingAt: 18892621587
    annotations:
      summary: "Instance {{ $labels.instance }} DISK usgae high"
      description: "{{ $labels.instance }} - {{ $labels.device }} - {{ $labels.mountpoint }} DISK usage above 80% (current value: {{ $value }})"
  - alert: hostNetOutAlert
    expr: round(irate(node_network_transmit_bytes_total{device!~'tap.*|veth.*|br.*|docker.*|vir.*|lo.*|vnet.*'}[1m])/1024/1024) > 100
    for: 1m
    labels:
      severity: page
      dingAt: 18892621587
    annotations:
      summary: "Instance {{ $labels.instance }} NET-Out usgae high"
      description: "{{ $labels.instance }}-{{ $labels.device }} NET-Out (current value: {{ $value }}) MB/s"
  - alert: hostNetInAlert
    expr: round(irate(node_network_receive_bytes_total{device!~'tap.*|veth.*|br.*|docker.*|vir.*|lo.*|vnet.*'}[1m])/1024/1024) > 100
    for: 1m
    labels:
      severity: page
      dingAt: 18892621587
    annotations:
      summary: "Instance {{ $labels.instance }} NET-In usgae high"
      description: "{{ $labels.instance }}-{{ $labels.device }} NET-In (current value: {{ $value }}) MB/s"
  - alert: hostDiskWillFillIn4HoursAlert
    expr: predict_linear(node_filesystem_free_bytes{fstype!="tmpfs"}[1h], 4 * 3600) < 0
    for: 1m
    labels:
      severity: page
      dingAt: 18892621587
    annotations:
      summary: "Host disk will fill in 4 hours Instance {{ $labels.instance }}"
      description: "{{ $labels.instance }}-{{ $labels.device }}-{{ $labels.mountpoint }} Disk will fill in 4 hours at current write rate VALUE = {{ $value }}"
```
hoststats-alert.rules
```yml
groups: 
- name: hostStatsAlert
  rules:
  - alert: hostCpuUsageAlert
    expr: sum(avg without (cpu)(irate(node_cpu_seconds_total{mode!='idle'}[5m]))) by (instance) > 0.8
    for: 1m
    labels:
      severity: page
      dingAt: 18892621587
    annotations:
      summary: "Instance {{ $labels.instance }} CPU usgae high"
      description: "{{ $labels.instance }} CPU usage above 80% (current value: {{ $value }})"
  - alert: hostMemUsageAlert
    expr: (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes)/node_memory_MemTotal_bytes > 0.8
    for: 1m
    labels:
      severity: page
      dingAt: 18892621587
    annotations:
      summary: "Instance {{ $labels.instance }} MEM usgae high"
      description: "{{ $labels.instance }} MEM usage above 80% (current value: {{ $value }})"
  - alert: hostDiskUsageAlert
    expr: 1 - (node_filesystem_avail_bytes{fstype=~"ext4|xfs"} / node_filesystem_size_bytes{fstype=~"ext4|xfs"}) > 0.8
    for: 1m
    labels:
      severity: page
      dingAt: 18892621587
    annotations:
      summary: "Instance {{ $labels.instance }} DISK usgae high"
      description: "{{ $labels.instance }} - {{ $labels.device }} - {{ $labels.mountpoint }} DISK usage above 80% (current value: {{ $value }})"
  - alert: hostNetOutAlert
    expr: round(irate(node_network_transmit_bytes_total{device!~'tap.*|veth.*|br.*|docker.*|vir.*|lo.*|vnet.*'}[1m])/1024/1024) > 100
    for: 1m
    labels:
      severity: page
      dingAt: 18892621587
    annotations:
      summary: "Instance {{ $labels.instance }} NET-Out usgae high"
      description: "{{ $labels.instance }}-{{ $labels.device }} NET-Out (current value: {{ $value }}) MB/s"
  - alert: hostNetInAlert
    expr: round(irate(node_network_receive_bytes_total{device!~'tap.*|veth.*|br.*|docker.*|vir.*|lo.*|vnet.*'}[1m])/1024/1024) > 100
    for: 1m
    labels:
      severity: page
      dingAt: 18892621587
    annotations:
      summary: "Instance {{ $labels.instance }} NET-In usgae high"
      description: "{{ $labels.instance }}-{{ $labels.device }} NET-In (current value: {{ $value }}) MB/s"
  - alert: hostDiskWillFillIn4HoursAlert
    expr: predict_linear(node_filesystem_free_bytes{fstype!="tmpfs"}[1h], 4 * 3600) < 0
    for: 1m
    labels:
      severity: page
      dingAt: 18892621587
    annotations:
      summary: "Host disk will fill in 4 hours Instance {{ $labels.instance }}"
      description: "{{ $labels.instance }}-{{ $labels.device }}-{{ $labels.mountpoint }} Disk will fill in 4 hours at current write rate VALUE = {{ $value }}"
```
instancestats-alert.rules
```yml
groups:
- name: instanceStatsAlert
  rules:
  - alert: instanceUpDownAlert
    expr: up == 0
    for: 1m
    labels:
      severity: page
      dingAt: 18892621587
    annotations:
      summary: "Instance {{ $labels.instance }} is down"
      description: "{{ $labels.instance }} - {{ $labels.job }} is down"
```
mysqlstats-alert.rules
```yml
groups:
- name: mysqlStatsAlert
  rules:
  - alert: mysqlSlaveIOthreadNotRunningAlert
    expr: mysql_slave_status_master_server_id > 0 and ON (instance) mysql_slave_status_slave_io_running == 0
    for: 1m
    labels:
      severity: page
      dingAt: 18892621587
    annotations:
      summary: "MySQL Slave IO thread not running (instance {{ $labels.instance }})"
      description: "MySQL Slave IO thread not running on {{ $labels.instance }}\n  masterHost: {{ $labels.master_host }}"
  - alert: mysqlSlaveSQLthreadNotRunningAlert
    expr: mysql_slave_status_master_server_id > 0 and ON (instance) mysql_slave_status_slave_sql_running == 0
    for: 1m
    labels:
      severity: page
      dingAt: 18892621587
    annotations:
      summary: "MySQL Slave SQL thread not running (instance {{ $labels.instance }})"
      description: "MySQL Slave SQL thread not running on {{ $labels.instance }}\n  masterHost: {{ $labels.master_host }}"
  - alert: mysqlTooManyConnectionsAlert
    expr: avg by (instance) (max_over_time(mysql_global_status_threads_connected[5m])) / avg by (instance) (mysql_global_variables_max_connections) * 100 > 80
    for: 1m
    labels:
      severity: page
      dingAt: 18892621587
    annotations:
      summary: "MySQL too many connections (instance {{ $labels.instance }})"
      description: "More than 80% of MySQL connections are in use on {{ $labels.instance }}\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
  - alert: mysqlHighThreadsRunningAlert
    expr: avg by (instance) (max_over_time(mysql_global_status_threads_running[5m])) / avg by (instance) (mysql_global_variables_max_connections) * 100 > 60
    for: 1m
    labels:
      severity: page
      dingAt: 18892621587
    annotations:
      summary: "MySQL high threads running (instance {{ $labels.instance }})"
      description: "More than 60% of MySQL connections are in running state on {{ $labels.instance }}\n VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
  - alert: mysqlRestartedAlert
    expr: mysql_global_status_uptime < 60
    for: 1m
    labels:
      severity: page
      dingAt: 18892621587
    annotations:
      summary: "MySQL restarted (instance {{ $labels.instance }})"
      description: "MySQL has just been restarted, less than one minute ago on {{ $labels.instance }}\n VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```
rabbitmqstats-alert.rules
```yml
groups:
- name: rabbitmqStatsAlert
  rules:
  - alert: rabbitmqTooManyMessagesReadyAlert
    expr: rabbitmq_queue_messages_ready > 1000
    for: 1m
    labels:
      severity: page
      dingAt: 18892621587
    annotations:
      summary: "Too many messages ready Instance {{ $labels.instance }}"
      description: "Too many messages ready {{ $labels.instance }} - {{ $labels.job }} (current value: {{ $value }})"
```
redisstats-alert.rules
```yml
groups:
- name: redisStatsAlert
  rules:
  - alert: redisMissingMasterAlert
    expr: count(redis_instance_info{role="master"}) == 0
    for: 1m
    labels:
      severity: page
      dingAt: 18892621587
    annotations:
      summary: "Redis missing master (instance {{ $labels.instance }})"
      description: "Redis cluster has no node marked as master.\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
  - alert: redisTooManyMastersAlert
    expr: count(redis_instance_info{role="master"}) > 1
    for: 1m
    labels:
      severity: page
      dingAt: 18892621587
    annotations:
      summary: "Redis too many masters (instance {{ $labels.instance }})"
      description: "Redis cluster has too many nodes marked as master.\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
  - alert: redisMissingBackupAlert
    expr: time() - redis_rdb_last_save_timestamp_seconds > 60 * 60 * 24
    for: 1m
    labels:
      severity: page
      dingAt: 18892621587
    annotations:
      summary: "Redis missing backup (instance {{ $labels.instance }})"
      description: "Redis has not been backuped for 24 hours\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
  - alert: redisTooManyConnectionsAlert
    expr: redis_connected_clients > 100
    for: 1m
    labels:
      severity: page
      dingAt: 18892621587
    annotations:
      summary: "Redis too many connections (instance {{ $labels.instance }})"
      description: "Redis instance has too many connections\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
  - alert: redisNotEnoughConnectionsAlert
    expr: redis_connected_clients < 5
    for: 1m
    labels:
      severity: page
      dingAt: 18892621587
    annotations:
      summary: "Redis not enough connections (instance {{ $labels.instance }})"
      description: "Redis instance should have more connections (> 5)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
  - alert: redisRejuctedConnectionsAlert
    expr: increase(redis_rejected_connections_total[1m]) > 0
    for: 1m
    labels:
      severity: page
      dingAt: 18892621587
    annotations:
      summary: "Redis rejected connections (instance {{ $labels.instance }})"
      description: "Some connections to Redis has been rejected\n VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

grafana
```shell script
docker run -d \
--restart always \
-p 3000:3000 \
-v /dbdata/grafana/conf:/etc/grafana \
-v /dbdata/grafana/data:/var/lib/grafana \
-v /dbdata/grafana/log:/var/log/grafana \
--name=grafana \
-e "GF_INSTALL_PLUGINS=grafana-clock-panel,grafana-simple-json-datasource,grafana-piechart-panel" \
grafana/grafana
```

cadvisor
```shell script
--name=cadvisor \
--restart always \
-v /:/rootfs:ro \
-v /var/run:/var/run:ro \
-v /sys:/sys:ro \
-v /app/docker/:/var/lib/docker:ro \
-v /dev/disk/:/dev/disk:ro \
-p 9380:8080 \
google/cadvisor
```

mysql_exporter
```shell script
docker run -d \
--name mysql_exporter \
--restart always \
-p 9104:9104 \
-e DATA_SOURCE_NAME="username:password@(192.16.0.89:3306)/" \
prom/mysqld-exporter
```

redis_exporter
```shell script
docker run -d \
--name redis_exporter \
--restart always \
-p 9121:9121 \
oliver006/redis_exporter \
--redis.password 'password'
```

rabbitmq_exporter
```shell script
docker run -d \
--name rabbitmq_exporter \
--restart always \
-p 9419:9419 \
-e RABBIT_URL=http://192.16.0.89:15672 \
-e RABBIT_USER=username \
-e RABBIT_PASSWORD='password' \
kbudde/rabbitmq-exporter
```

alertmanager
```shell script
docker run -d \
--name alertmanager \
--restart always \
-p 9093:9093 \
-v /dbdata/alertmanager/alertmanager.yml:/etc/alertmanager/config.yml \
prom/alertmanager \
--config.file=/etc/alertmanager/config.yml
```
alertmanager.yml
```yml
global:
  resolve_timeout: 2m

# The root route on which each incoming alert enters.
route:
  # The root route must not have any matchers as it is the entry point for
  # all alerts. It needs to have a receiver configured so alerts that do not
  # match any of the sub-routes are sent to someone.
  receiver: 'dingtalk-robot-webhook'

  # The labels by which incoming alerts are grouped together. For example,
  # multiple alerts coming in for cluster=A and alertname=LatencyHigh would
  # be batched into a single group.
  #
  # To aggregate by all possible labels use '...' as the sole label name.
  # This effectively disables aggregation entirely, passing through all
  # alerts as-is. This is unlikely to be what you want, unless you have
  # a very low alert volume or your upstream notification system performs
  # its own grouping. Example: group_by: [...]
  group_by: ['alertname','host']

  # When a new group of alerts is created by an incoming alert, wait at
  # least 'group_wait' to send the initial notification.
  # This way ensures that you get multiple alerts for the same group that start
  # firing shortly after another are batched together on the first
  # notification.
  group_wait: 45s

  # When the first notification was sent, wait 'group_interval' to send a batch
  # of new alerts that started firing for that group.
  group_interval: 2m

  # If an alert has successfully been sent, wait 'repeat_interval' to
  # resend them.
  repeat_interval: 4h

  # All the above attributes are inherited by all child routes and can
  # overwritten on each.

  # The child route trees.
  routes:
  # This routes performs a regular expression match on alert labels to
  # catch alerts that are related to a list of services.
  - match_re:
      severity: ^(page|warning|danger)$
    receiver: dingtalk-robot-webhook

# Inhibition rules allow to mute a set of alerts given that another alert is
# firing.
# We use this to mute any warning-level notifications if the same alert is
# already critical.
inhibit_rules:
- source_match:
    host: 'linux-jd-master'
  target_match:
    host: 'linux-jd-slave'
  equal: ['alertname','instance','job']


receivers:
- name: 'dingtalk-robot-webhook'
  webhook_configs: 
  - send_resolved: true
    url: http://192.16.0.89:9527/notify/dingtalk/alertmanager
```

blackbox-exporter
```shell script
docker run -d \
--name blackbox-exporter \
--restart always \
-p 9115:9115 \
-v /dbdata/blackbox-exporter/blackbox.yml:/etc/blackbox_exporter/config.yml \
prom/blackbox-exporter 
```
blackbox.yml
```yml
modules: 
  http_2xx: 
    prober: http
    timeout: 10s
    http:
      method: GET
  http_post_2xx:
    prober: http
    timeout: 10s
    http:
      method: POST
      headers:
        Content-Type: application/json
      body: '{}'
  http_custom_ca_example:
    prober: http
    http:
      method: GET
      tls_config:
        ca_file: "/cert/4189589_jd.jdgczxc.com.key"
  tcp_connect_example:
    prober: tcp
    timeout: 5s
  icmp:
    prober: icmp
```

blackboxstats-alert.rules
```yml
groups:
  - name: blackboxStatsAlert
    rules:
      - alert: blackboxProbeFailedAlert
        expr: probe_success == 0
        for: 1m
        labels:
          severity: page
          dingAt: 18892621587
        annotations:
          summary: "Blackbox probe failed (instance {{ $labels.instance }})"
          description: "Probe failed\n  VALUE = {{ $value }}\n instance {{ $labels.instance }}"
      - alert: blackboxSlowProbeAlert
        expr: avg_over_time(probe_duration_seconds[1m]) > 3
        for: 1m
        labels:
          severity: page
          dingAt: 18892621587
        annotations:
          summary: "Blackbox slow probe (instance {{ $labels.instance }})"
          description: "Blackbox probe took more than 3s to complete\n  VALUE = {{ $value }}\n instance {{ $labels.instance }}"
      - alert: blackboxProbeHttpFailureAlert
        expr: probe_http_status_code <= 199 OR probe_http_status_code >= 400
        for: 1m
        labels:
          severity: page
          dingAt: 18892621587
        annotations:
          summary: "Blackbox probe HTTP failure (instance {{ $labels.instance }})"
          description: "HTTP status code is not 200-399\n VALUE = {{ $value }}\n instance {{ $labels.instance }}"
      - alert: blackboxProbeSlowHttpAlert
        expr: avg_over_time(probe_http_duration_seconds[1m]) > 3
        for: 1m
        labels:
          severity: page
          dingAt: 18892621587
        annotations:
          summary: "Blackbox probe slow HTTP (instance {{ $labels.instance }})"
          description: "HTTP request took more than 3s\n VALUE = {{ $value }}\n instance {{ $labels.instance }}"
      - alert: blackboxProbeSlowPingAlert
        expr: avg_over_time(probe_icmp_duration_seconds[1m]) > 3
        for: 1m
        labels:
          severity: page
          dingAt: 18892621587
        annotations:
          summary: "Blackbox probe slow ping (instance {{ $labels.instance }})"
          description: "Blackbox ping took more than 3s\n  VALUE = {{ $value }}\n instance {{ $labels.instance }}"
      - alert: blackboxSslCertificateWillExpireSoonAlert
        expr: probe_ssl_earliest_cert_expiry - time() < 86400 * 30
        for: 1m
        labels:
          severity: page
          dingAt: 18892621587
        annotations:
          summary: "Blackbox SSL certificate will expire soon (instance {{ $labels.instance }})"
          description: "SSL certificate expires in 30 days\n VALUE = {{ $value }}\n instance {{ $labels.instance }}"
```