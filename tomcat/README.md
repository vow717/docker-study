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