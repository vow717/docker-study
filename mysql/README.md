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




