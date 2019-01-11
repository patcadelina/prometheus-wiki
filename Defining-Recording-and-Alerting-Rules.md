
groups:
- name: etcd.k8s-product-a
  rules:
  - alert: InsufficientMembers
    annotations:
      summary: etcd cluster insufficient members
      description: If one more etcd member goes down the cluster will be unavailable
    expr: |
      if count(up{job="k8s-product-a-etcd"} == 0) > (count(up{job="k8s-product-a-etcd"}) / 2 - 1)
    for: 3m
    labels:
      severity: critical

  - alert: NoLeader
    annotations:
      summary: etcd member has no leader
      description: etcd member {{ $labels.instance }} has no leader
    expr: |
      if etcd_server_has_leader{job="k8s-product-a-etcd"} == 0
    for: 1m
    labels:
      severity: critical

  - alert: HighNumberOfLeaderChanges
    annotations:
      summary: a high number of leader changes within the etcd cluster are happening
      description: etcd instance {{ $labels.instance }} has seen {{ $value }} leader changes within the last hour
    expr: |
      if increase(etcd_server_leader_changes_seen_total{job="k8s-product-a-etcd"}[1h]) > 3
    labels:
      severity: warning

  - alert: HighNumberOfFailedGRPCRequests
    annotations:
      summary: a high number of gRPC requests are failing
      description: {{ $value }}% of requests for {{ $labels.grpc_method }} failed on etcd instance {{ $labels.instance }}
    expr: |
      if sum by(grpc_method) (rate(etcd_grpc_requests_failed_total{job="k8s-product-a-etcd"}[5m])) / sum by(grpc_method) (rate(etcd_grpc_total{job="k8s-product-a-etcd"}[5m])) > 0.01
    for: 10m
    labels:
      severity: warning

  - alert: HighNumberOfFailedGRPCRequests
    annotations:
      summary: a high number of gRPC requests are failing
      description: {{ $value }}% of requests for {{ $labels.grpc_method }} failed on etcd instance {{ $labels.instance }}
    expr: |
      if sum by(grpc_method) (rate(etcd_grpc_requests_failed_total{job="k8s-product-a-etcd"}[5m])) / sum by(grpc_method) (rate(etcd_grpc_total{job="k8s-product-a-etcd"}[5m])) > 0.05
    for: 5m
    labels:
      severity: warning

  - alert: GRPCRequestsSlow
    annotations:
      summary: slow gRPC requests
      description: on etcd instance {{ $labels.instance }} gRPC requests to {{ $label.grpc_method }} are slow
    expr: |
      if histogram_quantile(0.99, rate(etcd_grpc_unary_requests_duration_seconds_bucket[5m])) > 0.15
    for: 10m
    labels:
      severity: critical

  - alert: HighNumberOfFailedHTTPRequests
    annotations:
      summary: a high number of HTTP requests are failing
      description: {{ $value }}% of requests for {{ $labels.method }} failed on etcd instance {{ $labels.instance }}
    expr: |
      if sum by(method) (rate(etcd_http_failed_total{job="k8s-product-a-etcd"}[5m])) / sum by(method) (rate(etcd_http_received_total{job="k8s-product-a-etcd"}[5m])) > 0.01
    for: 10m
    labels:
      severity: warning

  - alert: HighNumberOfFailedHTTPRequests
    annotations:
      summary: a high number of HTTP requests are failing
      description: {{ $value }}% of requests for {{ $labels.method }} failed on etcd instance {{ $labels.instance }}
    expr: |
      if sum by(method) (rate(etcd_http_failed_total{job="k8s-product-a-etcd"}[5m])) / sum by(method) (rate(etcd_http_received_total{job="k8s-product-a-etcd"}[5m])) > 0.05
    for: 5m
    labels:
      severity: critical

  - alert: HTTPRequestsSlow
    annotations:
      summary: slow HTTP requests
      description: on etcd instance {{ $labels.instance }} HTTP requests to {{ $label.method }} are slow
    expr: |
      if histogram_quantile(0.99, rate(etcd_http_successful_duration_seconds_bucket[5m])) > 0.15
    for: 10m
    labels:
      severity: warning

  - alert: FdExhaustionClose
    annotations:
      summary: file descriptors soon exhausted
      description: {{ $labels.job }} instance {{ $labels.instance }} will exhaust its file descriptors soon
    expr: |
      if predict_linear(instance:fd_utilization[1h], 3600 * 4) > 1
    for: 10m
    labels:
      severity: warning

  - alert: FdExhaustionClose
    annotations:
      summary: file descriptors soon exhausted
      description: {{ $labels.job }} instance {{ $labels.instance }} will exhaust its file descriptors soon
    expr: |
      if predict_linear(instance:fd_utilization[10m], 3600) > 1
    for: 10m
    labels:
      severity: critical

  - alert: EtcdMemberCommunicationSlow
    annotations:
      summary: etcd member communication is slow
      description: etcd instance {{ $labels.instance }} member communication with {{ $label.To }} is slow
    expr: |
      if histogram_quantile(0.99, rate(etcd_network_member_round_trip_time_seconds_bucket[5m])) > 0.15
    for: 10m
    labels:
      severity: warning

  - alert: HighNumberOfFailedProposals
    annotations:
      summary: a high number of proposals within the etcd cluster are failing
      description: etcd instance {{ $labels.instance }} has seen {{ $value }} proposal failures within the last hour
    expr: |
      if increase(etcd_server_proposals_failed_total{job="k8s-product-a-etcd"}[1h]) > 5
    labels:
      severity: warning

  - alert: HighFsyncDurations
    annotations:
      summary: etcd membehigh fsync durations
      description: etcd instance {{ $labels.instance }} fync durations are high
    expr: |
      if histogram_quantile(0.99, rate(etcd_disk_wal_fsync_duration_seconds_bucket[5m])) > 0.5
    for: 10m
    labels:
      severity: warning

  - alert: HighCommitDurations
    annotations:
      summary: high commit durations
      description: etcd instance {{ $labels.instance }} commit durations are high
    expr: |
      if histogram_quantile(0.99, rate(etcd_disk_backend_commit_duration_seconds_bucket[5m])) > 0.25
    for: 10m
    labels:
      severity: warning