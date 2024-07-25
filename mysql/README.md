首先进入dp目录里
## mysql
### 创建个例子mysql容器
docker run --name temp -itd -e MYSQL_ROOT_PASSWORD=Wkfroot mysql:8.0.27
### 将配置文件cp到主机里
docker cp temp:/etc/mysql/my.cnf .

docker cp temp:/etc/mysql/conf.d .
### 删除这个容器
docker rm -f temp
### 修改my.cnf主配置文件字符集
vim my.cnf <br>
```
[mysql] 
default-character-set=utf8mb4
[mysql.servcer]
default-character-set=utf8mb4
```
### 写dockerfile脚本
vim Dockerfile

FROM mysql:8.0.27

esc :wq退出
### 构建镜像
docker build -t myappdb .

### 运行容器
docker run --name appDB -itd \
-p 3307:3306 \ #端口映射\
-v ./my.cnf:/etc/mysql/my.cnf \  #数据卷的挂载 \
-v ./conf.d:/etc/mysql/conf.d \
-v ./data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=Wkfroot \
--network appnet \  #加入到自定义网络中去\
myappdb

### 不管自定义镜像的话，在app目录下vim docker-compose.yml
```
version: "3.8"
services:
  appdb:  
    image: mysql:8.0.27       
    ports:              
      - "3308:3306"            
    volumes:            
      - ./db/conf.d:/etc/mysql/conf.d      
      - ./db/data:/var/lib/mysql        
      - ./db/my.cnf:/etc/mysql/my.cnf        
      - ./db/mysql-files:/var/lib/mysql-files     

    networks:       
      - net1  
    environment:  
      MYSQL_ROOT_PASSWORD: Wkfroot   

networks: 
  net1:  
    driver: bridge   
```

然后esc :wq保存退出， <br>
docker-compose up -d启动容器

### 连接
用navicat 新建连接，ip地址写192.168.0.104(我虚拟机的ip) <br>
端口3308(yml文件里映射的)  <br>
用户名是root  <br>
密码是Wkfroot <br>
测试连接--连接成功  

### 问题
1.容器启动后自动关闭 <br>
可能是端口被占用，关闭其他冲突的容器 <br>
2.权限问题 <br>
docker exec -it app-appdb-1(docker ps看看这个名字是什么) /bin/bash <br>
允许任何主机的连接 <br>
CREATE USER 'root'@'%' IDENTIFIED BY 'Wkfroot'; <br>
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION; <br>
FLUSH PRIVILEGES; 

## tomcat
### 续写yml
找到java17对应的tomcat版本 <br>
在appdb下又写: 
```
tomcat:  
    image: tomcat:9.0.73-jdk17-temurin 
    ports:   
      - "8082:8080" 
    environment:
      MYSQL_HOST: appdb
      MYSQL_PORT: 3306
      MYSQL_DATABASE: my_database
      MYSQL_USER: root
      MYSQL_PASSWORD: Wkfroot
    volumes:
      - ./webapps:/usr/local/tomcat/webapps
      - ./tomcat/conf:/usr/local/tomcat/conf
      - ./tomcat/lib:/usr/local/tomcat/lib  # 挂载目录以包含 MySQL JDBC 驱动
    depends_on:
      - appdb
    networks:
      - net1
```
esc  :wq退出后
在/app/tomcat/context.xml内写上:
```
<Context>
  <Resource name="jdbc/MyDB"
            auth="Container"
            type="javax.sql.DataSource"
            maxTotal="100"
            maxIdle="30"
            maxWaitMillis="10000"
            username="root"
            password="Wkfroot"
            driverClassName="com.mysql.cj.jdbc.Driver"
            url="jdbc:mysql://appdb:3306/my_database?useSSL=false&amp;serverTimezone=UTC"/>
</Context>
```
退出，完成。
docker-compose up -d启动。
新拉取的tomcat可能下载要花些时间。

### 容器启动，删除
docker-compose up -d <br>
后台模式创建并运行

docker-compose stop <br>
停止容器

docker-compose down <br>
关闭并删除容器（保留卷和网络）

docker-compose down -v <br>
删除容器的同时删除关联的卷
docker-compose down --remove-orphans <br>
删除为服务创建的网络
