# ElasticSearch
本教程是讲es部署到 虚拟机上
## 下载

```sh
curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.10.1-linux-x86_64.tar.gz
tar -xzvf elasticsearch-7.10.1-linux-x86_64.tar.gz
cd elasticsearch-7.10.1
./bin/elasticsearch -d # -d 为后台运行

```
## 配置
`config/elasticsearch.yml`
```YAML
cluster.name: artron_test_es #集群名称,多个节点相同的集群名称视为一个集群
node.name: node-183-9200   #节点名称，每一个es实例视为一个节点
path.data: /path/to/data   #数据存放目录，不配置默认是es的bin同级目录data,(可以配置多个磁盘目录)
path.logs: /path/to/logs   #日志存放目录，不配置默认是es的bin同级目录log
network.host: 192.168.64.183 #绑定服务Ip地址
http.port: 9200             #http访问端口
cluster.initial_master_nodes: ["node-183-9200"] # 集群主节点信息
xpack.security.enabled: true # 安全认证
xpack.license.self_generated.type: basic #安全认证
xpack.security.transport.ssl.enabled: true #安全认证
```
### 插件
```
#plubing
/home/sunbaoqing/source/elasticsearch-7.10.1/plugins
安装了ik和analysis-icu插件
```
## 系统设置
###  设置内核参数
```sh
vi /etc/sysctl.conf
sysctl -p #执行命令sysctl -p生效
```
`vm.max_map_count=262144`
### 配置当前用户每个进程打开文件数量
启动时如果出现
` max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]`
```sh
vi /etc/security/limits.conf
sysctl -p
```
添加以下内容
```conf
es_starter_user hard nofile 65535
es_starter_user soft nofile 65535
```

es_starter_user 为启动es 的用户

修改完limits.conf，重新登录

## 账户
执行下面命令开始设置密码
`./bin/elasticsearch-setup-passwords  interactive`

|  账户   | 密码  | 备注  |
|  ----  | ----  | ---  |
| elastic  | art_2021 | 管理员  |
| apm_system  | art_2021 |   |
| kibana_system  | art_2021 |   |
| logstash_system  | art_2021 |   |
| beats_system  | art_2021 |   |
| remote_monitoring_user  | art_2021 |   |


## pipelines
[es-pipeline.md](es-pipeline.md)

## k8s deploy
es已经部署在服务器上，所以k8s 集群内部访问es需要使用emdpoints服务。
创建无头服务,kibana、logstash可以通过服务名访问外包的es集群
[es-svc.yaml](../1svc/es-svc.yaml)
```yaml
kind: Service
apiVersion: v1
metadata:
  name: log-es-svc
  namespace: monitoring
spec:
  type: ClusterIP
  ports:
    - port: 9200
      targetPort: 9200
---
kind: Endpoints
apiVersion: v1
metadata:
  name: log-es-svc
  namespace: monitoring
subsets:
  - addresses:
      - ip: 192.168.64.183
    ports:
      - port: 9200
```
## es索引清除
### 定时清除
[es-clean-index-cronJob.yaml](efk/es-clean-index-cronJob.yaml)
```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: es-clean-index
  namespace: monitoring
spec:
  schedule: "30 00 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: cronjob
              image: centos
              imagePullPolicy: IfNotPresent
              command:
                - /bin/sh
                - -c
                - sh /usr/local/clean.sh k8s* 21;
                  sh /usr/local/clean.sh side* 21;
              volumeMounts:
                - name: es-clean-config
                  mountPath: /usr/local
          restartPolicy: OnFailure
          volumes:
            - name: es-clean-config
              configMap:
                name: es-clean-config
```
清除索引脚本
[es-clean-index-sh.yaml](efk\es-clean-index-sh.yaml)
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: es-clean-config
  namespace: monitoring
data:
  clean.sh: |
    #!/bin/bash
    prefix=$1
    day=$2
    echo $prefix
    comp_date=`TZ=UTC-8 date -d "$day day ago" +"%Y.%m.%d"`
    curl   -XGET --user "elastic:art_2021" http://log-es-svc.monitoring.svc.cluster.local:9200/_cat/indices/$prefix$comp_date | awk -F" " '{print $3}' | while read LINE
    do
        echo $LINE
        curl  -v -i -XDELETE --user "elastic:art_2021" http://log-es-svc.monitoring.svc.cluster.local:9200/$LINE
        echo "-----"
    done

```

### 清除历史索引
使用 k8s  job 只执行一次
` sh /usr/local/clean.sh side* 7 ;`
此例是指，清除side开头7天以前的索引

[es-clean-index-job.yaml](efk/es-clean-index-job.yaml)
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: es-clean-index-job
spec:
  completions: 1
  parallelism: 1
  template:
    metadata:
      name: es-clean-index-job
    spec:
      containers:
        - name: job
          image: centos
          imagePullPolicy: IfNotPresent
          command:
            - /bin/sh
            - -c
            - cd /usr/local;
              sh /usr/local/clean.sh side* 7 ;
              sh /usr/local/clean.sh k8s* 7
          volumeMounts:
            - name: es-clean-config
              mountPath: /usr/local
      restartPolicy: OnFailure
      volumes:
        - name: es-clean-config
          configMap:
            name: es-clean-befor-day-config

```
清除索引脚本
使用configmap生成清除历史索引脚本。支持指定索引前缀 和 指定N天以前
[es-clean-index-befor-day.yaml](efk/es-clean-index-befor-day.yaml)
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: es-clean-befor-day-config
  namespace: monitoring
data:
  clean.sh: |
    #!/bin/bash
    prefix=$1
    day=$2
    function delete_indices() {
        format_date=`echo $1| sed 's/-/\./g'`
       sh ./clean_day_index.sh $prefix $format_date
    }
    curl -XGET --user "elastic:art_2021" http://log-es-svc.monitoring.svc.cluster.local:9200/_cat/indices/$prefix | awk -F" " '{print $3}' | awk -F"-" '{print $NF}' | egrep "[0-9]*\.[0-9]*\.[0-9]*" | sort | uniq  | sed 's/\./-/g' | while read LINE_DATE
    do
        comp_date=`date -d "$day day ago" +"%Y-%m-%d"`

        date1="$LINE_DATE 00:00:00"
        date2="$comp_date 00:00:00"
        t1=`date -d "$date1" +%s`
        t2=`date -d "$date2" +%s`

        if [ $t1 -le $t2 ]; then
        delete_indices $LINE_DATE
        fi
    done;
  clean_day_index.sh: |
    #!/bin/bash
    prefix=$1
    day=$2
    echo $prefix"-"$day""
      curl    -XGET --user "elastic:art_2021" http://log-es-svc.monitoring.svc.cluster.local:9200/_cat/indices/$prefix$day | awk -F" " '{print $3}' | while read LINE
        do
            echo $LINE
            curl  -v -i -XDELETE --user "elastic:art_2021" http://log-es-svc.monitoring.svc.cluster.local:9200/$LINE
            echo "-----"
        done

```
# kibana

## k8s deploy
[kibana.yaml](kibana.yaml)
下面配置通过环境变量方式将es的地址 账号密码传到kibana中
 
### route
[monitoring-route.yaml](monitoring-route.yaml)
traefik 路由配置
 

## es索引
`Stack Management> Index Patterns` 设置可以检索的索引
## 搜索
`_index : "k8s-golang-artron-job-2021.10.09" and log.msg: "人*"`
`_index : "k8s-golang-artron-job-2021.10.09" and log.msg~聊成`
https://www.cnblogs.com/softidea/p/12469771.html
```
一 字段搜索
限定字段全文搜索 ：field:value
精确搜索 ：filed:"value"（关键字加上双引号 ）
字段本身是否存在
_exists_:http ：返回结果中需要有 http 字段
_missing_:http ：不能含有 http 字段

二 通配符
? 匹配单个字符
* 匹配0到多个字符
kiba?a, el*search
? * 不能用作第一个字符，例如 ：?text *text

三 正则
es支持部分正则功能
mesg:/mes{2}ages?/

四 模糊搜索
~  : 在一个单词后面加上~启用模糊搜索
first~  也能匹配到 frist

五 近似搜索
在短语后面加上~
"select where"~3 表示 select 和 where 中间隔着3个单词以内。

六 范围搜索
数值和时间类型的字段可以对某一范围进行查询
status:[200 TO 400]
date:{"now-6h" TO "now"}
[ ] 表示端点数值包含在范围内，{ } 表示端点数值不包含在范围内。

七 逻辑操作
AND   
OR
+ ：搜索结果中必须包含此项
- ：不能含有此项
+apache -jakarta test ：结果中必须存在 apache，不能有 jakarta，test 可有可无。

八 分组
(jakarta OR apache) AND jakarta
字段分组
title:(+return +"pink panther")
九 转义特殊字符
+ - && || ! () {} [] ^" ~ * ? : \
以上字符当作值搜索的时候需要用\转义
十 lucene 语法参考
https://lucene.apache.org/core/5_2_0/queryparser/org/apache/lucene/queryparser/classic/package-summary.html
```

# filebeat
k8s 中 filebeat 使用边车模式和应用服务驻在同一个pod中,通过volumeMounts挂在应用日志到filebeat中

sidecar 模式在pod中需要增加filebeat 的containers,并且需要配置应用的日志挂在到filebeat中
## deployment配置
[demo-deploy.yaml](demo/demo-deploy.yaml)
## filebeat配置
[fb-config-java.yaml](../filebeat-config/fb-config-java.yaml)

# k8s event
通过使用kube-eventrouter 收集k8s  event 日志信息

eventrouter通过监听集群event，然后推送到kafka，在通过logstash 写入到es中
[event](event)
[fb-config-eventrouter.yaml](../filebeat-config/fb-config-eventrouter.yaml)