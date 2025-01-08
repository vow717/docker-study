
### default.conf.template:
server {
    listen 80;
    server_name localhost;
    gzip_static on;
    gzip_http_version 1.1;
    proxy_http_version 1.1;

    location / {
        root /usr/share/nginx/html;
        index index.html index.htm;
        try_files $uri $uri/ /index.html; //vue的history模式，不然刷新会404,
    }

    location /api {
        proxy_pass http://${bhost}; //反向代理
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

### nginx里面： 
  gzip_static on;开启静态压缩  
  gzip_http_version 1.1;设置在 HTTP/1.1 协议下启用 Gzip 压缩，确保只有符合 HTTP/1.1 协议的请求才会被考虑进行 Gzip 压缩  
  proxy_http_version 1.1;用于指定在转发请求时使用的 HTTP 协议版本  

## try_files $uri $uri/ /index.html
对于支持 Vue.js History 模式至关重要。当用户在浏览器中访问一个基于 Vue.js History 模式的前端路由时，由于服务器上实际上可能并没有与该路由对应的物理文件（例如，http://example.com/dashboard这个路由对应的物理文件不存在），  
服务器（nginx）收到请求后：  
首先按照$uri查找，肯定找不到与这个前端路由对应的文件。然后按照$uri/查找目录下的默认文件，通常也找不到，因为这是前端路由，不是真正的目录结构对应的文件。  
最后，nginx返回index.html。这个index.html文件包含了 Vue.js 应用的所有 JavaScript 代码，包括路由相关的逻辑。当浏览器接收到index.html后，Vue.js 的路由机制就会在浏览器端接管，根据当前的 URL 路径（例如http://example.com/dashboard）来渲染出对应的页面内容，就好像这个页面是从服务器上获取到的真正的dashboard页面一样。这样，nginx的这个配置就巧妙地解决了 Vue.js History 模式下前端路由与服务器文件系统不匹配的问题，使得用户能够顺利地访问基于 History 模式的 Vue.js 应用中的各个路由。  