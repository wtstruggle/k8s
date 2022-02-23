#  Prometheus
## k8s 中部署Prometheus

### 部署operator、gragfa、promethues、alertmanager
```bash
kubectl ns monitoring
kubectl create -f setup -f grafana -f prometheus -f alertmanager
kubectl create -f svc-monitor/traefik-service-monitor.yaml
kubectl create -f exporter/blackbox -f exporter/node -f k8s  -f prometheus-adapter
```

### 配置信息
#### grafana 持久化
将grafana 的grafana-storage存储类型由默认的emptyDir 改成pvc
```yaml
      - name: grafana-storage
        persistentVolumeClaim:
          claimName: monitoring-grafana-pvc
```
#### grafana 路由
[grafana-route.yaml](grafana/grafana-route.yaml)

#### grafana 账号
admin  art_2021


#### grafana  dashboards
https://grafana.com/grafana/dashboards/6417

##### k8s
###### dashboard [7249](https://grafana.com/grafana/dashboards/7249)
> Shows basic stuff about
- cluster health (pod status count, pod restarts etc.)
- cluster nodes (cpu, memory, storage etc.)
- running pods (cpu, memory etc.)
- nginx ingress controller (connections, resp time)
###### dashboard[6417](https://grafana.com/grafana/dashboards/6417)
> Summary metrics about containers running on Kubernetes nodes.
- Cluster pod/cpu/memory/disk usage
- Deployment replicas
- Pods running、pending、failed
- Containers running
###### dashboard[12123](https://grafana.com/grafana/dashboards/12123)
> prometheus operator
- Node count,Running pods,running Container ,actual volume max_map_count

##### traefik
###### dashboard [11462](https://grafana.com/grafana/dashboards/11462)
> Simple dashboard for Traefik 2, visualizing every important metric for your Traefik instance(s).
- uptime,404 error count,503 error count,avg response time

#### promethues 持久化

在[prometheus-prometheus.yaml](prometheus/prometheus-prometheus.yaml) 增加以下配置
prometheus 使用storageClass 方式动态创建pvc
```yaml
  storage:
    volumeClaimTemplate:
      spec:
        storageClassName: prometheus-data-db
        resources:
          requests:
            storage: 500Gi
```

#### promethues 路由
[promethues-route.yaml](promethues/promethues-route.yaml)

#### alertmanager 路由
[alertmanager-route.yaml](alertmanager/alertmanager-route.yaml)

### k8s 监控
kubelet、kube-apiserver、kube-state、coredns监控信息

###  export
#### exporter\blackbox
采集http、dns、tcp、icmp、post、ssl等相关信息

#### exporter\node
采集主机信息


#### prometheus-adapter
Kubernetes资源指标、自定义指标和外部指标api的实现。

### 删除promehteus
` kubectl delete --ignore-not-found=true -f alertmanager  -f exporter/node -f exporter/blackbox -f grafana -f k8s -f prometheus -f prometheus-adapter -f setup -f svc-monitor`

## 自动服务发现
### 通过additional的endpoints监控service
定义additional
`kubectl create secret generic additional-configs --from-file=additional/prometheus-additional.yaml -n monitoring`
[prometheus-additional.yaml](additional/prometheus-additional.yaml)
```yaml
- job_name: 'kubernetes-endpoints'
  kubernetes_sd_configs:
    - role: endpoints
  relabel_configs:
    - source_labels: [ __meta_kubernetes_service_annotation_prometheus_io_scrape ]
      action: keep
      regex: true
    - source_labels: [ __meta_kubernetes_service_annotation_prometheus_io_scheme ]
      action: replace
      target_label: __scheme__
      regex: (https?)
    - source_labels: [ __meta_kubernetes_service_annotation_prometheus_io_path ]
      action: replace
      target_label: __metrics_path__
      regex: (.+)
    - source_labels: [ __address__, __meta_kubernetes_service_annotation_prometheus_io_port ]
      action: replace
      target_label: __address__
      regex: ([^:]+)(?::\d+)?;(\d+)
      replacement: $1:$2
    - action: labelmap
      regex: __meta_kubernetes_service_label_(.+)
    - source_labels: [ __meta_kubernetes_namespace ]
      action: replace
      target_label: kubernetes_namespace
    - source_labels: [ __meta_kubernetes_service_name ]
      action: replace
      target_label: kubernetes_name
```
默认promethues会监控带有`prometheus.io/scrape: "true"`注解的service,监控的path为 `/metrics`,scheme为http
如要修改默认配置可以参考以下示例
```yaml
  annotations:
    prometheus.io/scrape: "true" #必填
    prometheus.io/path: "/metrics"
    prometheus.io/scheme: "http"
    prometheus.io/port: "8080" #多个端口时，如果不填会监控所有端口
```

### ServiceMonitor 匹配label方式
ServiceMonitor通过匹配servic的label标签和namespace方式让promethues监控service

[golang-service-monitor.yaml](svc-monitor/golang-service-monitor.yaml)
```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: golang-service-monitor
  labels:
    team: backend
spec:
  namespaceSelector:
    matchNames:
      - auction
      - sunbaoqing
  selector:
    matchLabels:
      tier: backend
  endpoints:
    - bearerTokenFile: /var/run/secrets/kubernetes.io/serviceaccount/token
      honorLabels: true
      port: http
      path: /metrics
      interval: 15s
    - bearerTokenFile: /var/run/secrets/kubernetes.io/serviceaccount/token
      honorLabels: true
      port: http-metrics
      path: /actuator/prometheus
      interval: 30s
```
> port 对应填写service的spec.prots.name
> path 填写对应暴露出的metrics 路径


参考 [demo-deploy.yaml](../efk/demo/demo-deploy.yaml)
