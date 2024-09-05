# 创建mysql

## 下载docker和docker-compose
首先要将docker,docker-compose都下载了，去官网下。

## 下载mysql镜像
然后拉取mysql镜像，可以docker pull mysql直接拉取最新的，
我是拉取的8.0.27版本的，docker pull mysql:8.0.27。

## 下载tomcat镜像
同理

## 准备活动
mkdir -p app/db
cd app
准备写docker-compose.yml

## 容器启动，删除
docker compose up -d <br>
后台模式创建并运行

docker compose stop <br>
停止容器

docker compose down <br>
关闭并删除容器（保留卷和网络）

docker compose down -v <br>
删除容器的同时删除关联的卷
docker compose down --remove-orphans <br>
删除为服务创建的网络

## 注意
（我这开放8082端口出去）
防火墙设置允许外部设备访问8082端口。
sudo firewall-cmd --zone=public --add-port=8082/tcp --permanent
sudo firewall-cmd --reload
