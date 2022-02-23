#项目
## 拉新活动
### 后台管理
http://operations-vue.artron.net/index
账号密码  admin  admin123
### h5
https://artexpress.artron.net/wapluckdraw/activity/app/1
###源码

|  名称   | 地址  | 备注  |
|  ----  | ----  | ---  |
| h5抽奖  | https://git.artron.net/java/luckdraw|    |
| vue后台管理  | https://git.artron.net/java/artron-operations-web-admin|    |
| java服务端  | https://git.artron.net/java/artron-operations |    |
 
#k8s
#基础服务
服务都在183上，IP 192.168.64.183
账号密码 sunbaoqing BEdnwIrHyUlc
##es
配置信息查看efk的deploy.md
```
#目录
/home/sunbaoqing/source/elasticsearch-7.10.1
```
##kafka
```
#目录
/home/sunbaoqing/container_data
#通过docker-compose的compose.yml 启动
```

##yapi
yapi依赖于mongodb，启动前需要先启动mongodb

账号密码
http://192.168.64.183:3000/group/11
sunbaoqing@artron.net  art_2021
```
#目录
/home/sunbaoqing/my-yapi
##启动命令
pm2 stop yapi
```

#### mongo
```
#目录
/usr/bin/mongod
#配置
/etc/mongod.conf
```

##开发jenkins
账号密码

http://192.168.64.183:8080/login?from=%2F
art_admin  art_2021
art_test  1111111
 
```
/usr/local/jenkins
nohup java  -jar jenkins.war >jenkins.log 2>&1 &
#jenkins 工作目录
/home/sunbaoqing/.jenkins
```