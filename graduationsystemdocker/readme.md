# 毕设管理系统前后端数据库部署全过程

## mysql
mysql:  
    image: mysql:8.0.33  
    ports:  
      - "3310:3306"  
    volumes:  
      - ./mysql/data:/var/lib/mysql  
    environment:  
      TZ: Asia/Shanghai  
      MYSQL_ROOT_PASSWORD: ${mysql_password}  
      MYSQL_DATABASE: my_database  
    command:  
      --max_connections=2500  
    healthcheck:  
      # test: mysql --user=root --password=1157 -e "select 1;"  
      test: mysql -uroot -p$$MYSQL_ROOT_PASSWORD -e "select 1;"  
      interval: 10s  
      timeout: 3s  
      retries: 3  

使用mysql:8.0.33版本（可能有些老了已经）  
端口映射无所谓，在内部使用，3306就可以  
environment配环境不多说  
下面的command和healthcheck:  
允许同时连接到数据库服务器的最大客户端连接数。通过将其设置为 2500  
test: mysql -uroot -p$$MYSQL_ROOT_PASSWORD -e "select 1;"：这是健康检查执行的具体测试命令。  
mysql 是用于连接到 MySQL 数据库的客户端命令。  
-uroot 表示以 root 用户身份登录到数据库，这是 MySQL 中默认的超级管理员用户。  
-p$$MYSQL_ROOT_PASSWORD 用于指定密码，这里通过使用环境变量   MYSQL_ROOT_PASSWORD 来传递密码值，在 Docker 环境中，两个 $ 符号的写法是为了正确地获取并传递该环境变量的值，确保在连接数据库时能使用正确的密码进行验证。  
-e "select 1;" 是要在数据库中执行的 SQL 语句，这里执行了一个简单的 select 1 查询，目的是验证数据库是否能够正常接收和处理查询请求，只要数据库能够正确执行这条简单语句并返回结果，就说明数据库在基本的查询功能方面是正常的，也就意味着服务处于健康状态。    
interval 配置：  
interval: 10s：表示健康检查的间隔时间为 10 秒，即每隔 10 秒就会执行一次上述的健康检查测试命令，持续监控 mysql 服务的健康状况。  
timeout 配置：  
timeout: 3s：设定了健康检查测试命令的超时时间为 3 秒。如果执行 mysql -uroot -p$$MYSQL_ROOT_PASSWORD -e "select 1;" 这个命令在 3 秒内没有完成，Docker 就会认为此次健康检查失败。  
retries 配置：  
retries: 3：当一次健康检查失败后，Docker 会按照 interval 设定的时间间隔再次进行健康检查，这里的 retries 值规定了可以重试的次数。也就是说，如果连续 3 次健康检查都失败了（每次执行测试命令超时或者返回错误结果等情况都算失败），那么 Docker 就会判定 mysql 服务处于不健康状态，进而可能根据 restart 等相关策略采取对应的操作（比如重启该服务等）。  

## openjdk
因为我用的后端java是21,所以镜像是openjdk:21
openjdk:  
    image: openjdk:21  
    command: ["java", "-jar", "/app/graduation-project-process-management-wkf-0.0.1-SNAPSHOT.jar"]  
    volumes:  
      - ./openjdk/upload:/graduationsystem/upload  
      - ./openjdk/graduation-project-process-management-wkf-0.0.1-SNAPSHOT.jar:/app/graduation-project-process-management-wkf-0.0.1-SNAPSHOT.jar  
    ports:  
      - '8080'  
    depends_on:  
      - mysql  

### command:
java：  
这是 Java 虚拟机（JVM）的可执行命令，用于启动 Java 程序。当在容器环境中执行这个命令时，它会调用容器内已经安装好的 Java 运行时环境（在这个例子中，容器基于 openjdk:21 镜像构建，所以会使用该镜像中包含的 OpenJDK 21 来运行程序）。  
-jar：  
这是 java 命令的一个参数选项，用于告诉 JVM 要以 JAR 文件的形式来运行一个 Java 应用程序。它会自动查找并加载 JAR 文件中包含的主类（在 JAR 文件的 MANIFEST.MF 文件中会指定主类信息），然后执行该主类中定义的 main 方法，从而启动整个 Java 应用程序。  
/app/graduation-project-process-management-wkf-0.0.1-SNAPSHOT.jar：  
这是具体要运行的 JAR 文件的路径。在容器内部，这个路径指向了应用程序打包后的 JAR 文件所在位置。这里表明容器启动后，JVM 会去加载并运行这个特定的 JAR 文件所代表的 Java 应用程序。  

<br>
volumes卷挂载。upload是学生教师上传下载pdf报告的存储位置。  
.jar包挂载到容器里面,通过java -jar启动的  

### 注意后端yml配置
   url: r2dbcs:mysql://mysql:3306/my_database  
   username: root  
   password: Wkfroot  
   密码和mysql:3306
   以及upload: '/graduationsystem/upload/'；graduationsystem/upload与卷挂载的一致  
## nginx
nginx:  
    image: nginx:latest  
    restart: always  
    volumes:  
      - ./nginx/templates:/etc/nginx/templates  
      - ./nginx/html/dist:/usr/share/nginx/html # 将前端构建产物挂载到Nginx容器>中  
    ports:  
      - "81:80"  
    depends_on:  
      - openjdk  
    environment:  
      - bhost=openjdk:8080  

html/dist是vite打包vue而成的静态文件,挂载到nginx容器里面  
templates，其实很多都是config/default.conf，但这里是templates/default.conf.templates，因为为了可以在docker-compose.yml里面动态改变反向代理。（看nginx目录下）  
environment下写bhost=openjdk:8080，这样nginx/templates/default.conf.templates下proxy_pass http://${bhost}; 这样就可以动态改变了，不用跑到nginx里面再改，麻烦。  