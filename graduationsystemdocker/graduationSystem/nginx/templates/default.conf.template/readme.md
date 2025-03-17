
### default.conf.template:
server {
    listen 80;
    server_name localhost;
    gzip_static on;
    gzip_http_version 1.1;
    proxy_http_version 1.1;

    location / {
        add_header Cache-Control "no-cache"; //协商缓存
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

    location ~ \.(png|jpg|jpeg|gif)$ {
        add_header Cache-Control "max-age=2592000, public";
        etag on;
        expires 30d;
    }

    location ~ \.(css|js)$ {
        add_header Cache-Control "max-age=31536000, public";
        # 可以添加 ETag 和 Last-Modified 相关的配置
        etag on;
        expires 1y;
    }
}

### nginx里面： 
  gzip_static on;开启静态压缩  
  gzip_http_version 1.1;设置在 HTTP/1.1 协议下启用 Gzip 压缩，确保只有符合 HTTP/1.1 协议的请求才会被考虑进行 Gzip 压缩  
  proxy_http_version 1.1;用于指定在转发请求时使用的 HTTP 协议版本  

### try_files $uri $uri/ /index.html
对于支持 Vue.js History 模式至关重要。当用户在浏览器中访问一个基于 Vue.js History 模式的前端路由时，由于服务器上实际上可能并没有与该路由对应的物理文件（例如，http://example.com/dashboard这个路由对应的物理文件不存在），  
服务器（nginx）收到请求后：  
首先按照$uri查找，肯定找不到与这个前端路由对应的文件。然后按照$uri/查找目录下的默认文件，通常也找不到，因为这是前端路由，不是真正的目录结构对应的文件。  
最后，nginx返回index.html。这个index.html文件包含了 Vue.js 应用的所有 JavaScript 代码，包括路由相关的逻辑。当浏览器接收到index.html后，Vue.js 的路由机制就会在浏览器端接管，根据当前的 URL 路径（例如http://example.com/dashboard）来渲染出对应的页面内容，就好像这个页面是从服务器上获取到的真正的dashboard页面一样。这样，nginx的这个配置就巧妙地解决了 Vue.js History 模式下前端路由与服务器文件系统不匹配的问题，使得用户能够顺利地访问基于 History 模式的 Vue.js 应用中的各个路由。  

### add_header Cache-Control "no-cache";
当服务器发送响应时，添加这个头信息告诉浏览器和中间缓存服务器（如代理服务器），在使用缓存的内容之前，必须先与服务器进行验证。这并不意味着禁止缓存，而是要求每次使用缓存之前都要进行一个称为 “再验证（revalidation）” 的过程。  
具体来说，当浏览器想要使用一个被标记为no - cache的缓存文件时，它会向服务器发送一个带有条件请求头（如If - Modified - Since或If - None - Match）的请求。服务器会根据这些条件来判断缓存的文件是否仍然有效。如果有效，服务器会返回一个304 Not Modified的响应，告诉浏览器可以继续使用缓存中的文件；如果文件已经被修改，服务器会返回新的文件内容。  
如果不设置其他的默认用If-Modified-Since,如果 etag on;那就用etag。

### css和js文件
这些文件通常更新频率相对较低，可以设置较长的缓存时间，同时采用版本化的文件名。例如，在打包时给文件名添加版本号或哈希值，这样在文件内容更新时，文件名会改变，客户端会请求新的文件，而旧文件可以继续被缓存。在 Nginx 配置中，可以为这些文件单独设置 Cache-Control 头  

### 图片文件
图片文件的更新频率也相对较低，同样可以设置较长的缓存时间。可以根据图片的类型和用途进行更细致的区分，例如，对于用户头像这类可能会更新的图片，可以设置较短的缓存时间；对于一些静态的图标等，可以设置较长的缓存时间。  