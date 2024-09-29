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
      - mysql
```
考虑到Tomcat不直接配置数据库连接，数据库连接由Web应用自身配置（例如在context.xml或其他配置文件中）。
（暂时不知道怎么弄）
```

  tomcat:
    image: tomcat:10.1.6-jdk17-temurin
    ports:
      - "8082:8080" 
    volumes:
      - /app/tomcat/webapps:/usr/local/tomcat/webapps
    environment:
      TZ: Asia/Shanghai
    depends_on:
      mysql:
        condition: service_healthy

```

esc  :wq退出后
在/app/tomcat/conf/context.xml内写上:
```
<Context>
  <Resource name="jdbc/MyDB"
            type="javax.sql.DataSource"
            maxTotal="100"
            maxIdle="30"
            maxWaitMillis="10000"
            username="root"
            password="Wkfroot"
            driverClassName="com.mysql.cj.jdbc.Driver"
            url="jdbc:mysql://mysql:3306/my_database" />
</Context>
```
退出，完成。
docker compose up -d启动。
新拉取的tomcat可能下载要花些时间。

### 注意
为了让war包包括你需要的全部依赖项，把应用程序pom.xml下的依赖项的<scope>全删了，变成默认。

再次docker-compose up -d 启动