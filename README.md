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

## 打包前端作业到虚拟机
在idea下压缩成的war包。
Windows 10以上自带SSH，直接用SSH命令。
win+r打开主机终端
scp 主机需要传输的文件绝对地址 虚拟机用户名@虚拟机地址:需要传输到虚拟机的目录
然后可能需要输入密码。
需要传输到tomcat下的webapps里，再运行容器。
这样在你主机的电脑上浏览器打开虚拟机ip＋对应端口就可以打开你的前端作业了。