#kafka
183上kafka通过docker-compose部署

由于腾讯云不支持压缩和密码，所以连接kafka的用户密码未启用

[compose.yml](compose.yml)

`docker-compsoe -f composey.yml up -d `
`docker stop kafka`
`docker start kafka`