```sh
kubectl create -f 0ns
kubectl create -f 1svc


kubectl create -f filebeat-config/fb-config-eventrouter.yaml -n monitoring

kubectl create -f filebeat-config/fb-config-go.yaml -n ysw-server
kubectl create -f filebeat-config/fb-config-php.yaml -n ysw-server
kubectl create -f filebeat-config/fb-config-java.yaml -n ysw-server
kubectl create -f filebeat-config/fb-config-vue.yaml -n ysw-server
kubectl create -f filebeat-config/fb-config-python.yaml -n ysw-server

kubectl create -f filebeat-config/fb-config-go.yaml -n sy-server
kubectl create -f filebeat-config/fb-config-php.yaml -n sy-server
kubectl create -f filebeat-config/fb-config-java.yaml -n sy-server
kubectl create -f filebeat-config/fb-config-vue.yaml -n sy-server

kubectl create -f svc/operation-mysql-svc.yaml -n ysw-server
kubectl create -f svc/operation-redis-svc.yaml -n ysw-server

kubectl create -f filebeat-config/fb-config-vue.yaml -n ysw-web
kubectl create -f filebeat-config/fb-config-vue.yaml -n sy-web

```