# create svc
[tencent-apigw-svc.yaml](tencent-apigw-svc.yaml)
由于traefik不支持ExternalName，所以使用service+endpoint方式

然后ingressroute 的svc指向tencent-apigw-svc
# 腾讯云APIGW 配置

## 创建后端通道
apigw是通过后端通道访问tke集群中的pod
通道类型选择TKE,选择私有网络、输入服务名称、选择tke集群、选择命名空间、选择svc和端口，选择后端类型


## 创建服务
服务实例必须选择专享型，访问方式 内网VPC。如果通过traefik转发过来的请求，无需选择https
设置自定义域名:新增域名>输入域名地址(必须是合法域名)>选择协议(http)>填写自定义映射路径>环境选择发布、路径输入/

## 创建api
选择服务>新建API
前端配置:输入路径、选择请求方法(所有请求选择ANY)、鉴权类型默认即可

后端配置:选择后端类型(VPC内资源)、选择vpc、对接方式选择后端通道、后端通道类型选择TKE(通道为前面创建的)、
输入后端路径(转发时候会在请求前加上该值，默认/)、选择请求方法、设置超时时间

响应结果: 返回类型选择JSON

## 创建插件
### 自定义插件
输入插件名称、插件类型选择自定义响应体、选择函数(如果没有点击下面按钮创建)
勾选自定义内容

### 云函数
新建云函数>模版创建>(搜索响应)自定响应函数>实现业务逻辑（默认是python）


### 绑定api
选择插件>选择绑定api>选择具体服务和需要绑定的api