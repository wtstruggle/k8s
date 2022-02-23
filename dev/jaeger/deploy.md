#  Jaeger
## k8s 中部署jaeger
jaeger 使用1.25版本，使用kubeneters operator 部署

jaeger 使用sidecar模式部署

执行以下命令

```kubectl
kubectl apply -f jaegertracing.io_jaegers_crd.yaml -n monitoring
kubectl apply -f service_account.yaml -n monitoring
kubectl apply -f role.yaml -n monitoring
kubectl apply -f role_binding.yaml -n monitoring
kubectl apply -f operator.yaml -n monitoring
kubectl apply -f cluster_role.yaml -n monitoring
kubectl apply -f cluster_role_binding.yaml -n monitoring
kubectl apply -f jaeger.yaml -n monitoring
```

ui、query、collector、storage 等配置信息都在 `jaeger.yaml` 文件中

## 项目中使用

在项目的Deployment.yaml文件中引入jaeger的sidecar注解即可(参照`efk/filebeat.yaml`)
```yaml
  annotations:
    "sidecar.jaegertracing.io/inject": "true"

      annotations:
        sidecar.istio.io/inject: "true"
```
 
如果不想使用sidecar模式可以在项目中直接配置collector地址
```yaml
    http-sender:
      url: http://jaeger-prod-collector-headless.monitoring.svc.cluster.local:14268/api/traces
```

## jaeger 引入kafka
搭建kafka环境并创建 `jaeger-spans`的topic

jaeger.yaml配置
