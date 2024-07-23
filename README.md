# 创建mysql

## 下载docker和docker-compose
首先要将docker,docker-compose都下载了，去官网下。

## 下载mysql镜像
然后拉取mysql镜像，可以docker pull mysql直接拉取最新的，
我是拉取的8.0.27版本的，docker pull mysql:8.0.27。

## 创建mysql容器
然后创建一个项目，mkdir myproject, cd myproject进入项目目录。
接着创建一个docker-compose.yml文件：vim docker-compose.yml，
按i进入编辑模式，然后输入内容后,按esc退出编辑模式，然后输入:wq保存退出。

## 运行mysql容器
然后在myproject目录下docker-compose up -d创建并启动容器并后台运行。
等一会儿在myproject目录下ls，
会看到有db_data文件夹，这个是持久化数据的文件夹,

### docker-compose stop 停止容器
### docker-compose down 删除容器
### docker-compose start 启动容器

之后进入idea，右边点击database添加数据库，
ip地址的虚拟机ip,在虚拟机上输入ifconfig查看,端口是在docker-compose.yml文件中配置的3306.

## 查看数据
在idea上随便加些表。
在myproject目录下docker exec -it my_mysql_container bash 进入mysql容器，
然后mysql -u root -p,输入密码,然后use my_database;show tables查看数据库表。