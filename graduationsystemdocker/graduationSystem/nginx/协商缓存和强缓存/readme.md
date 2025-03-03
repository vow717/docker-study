# 强缓存和协商缓存
在 Web 性能优化中，缓存机制起着至关重要的作用。强缓存和协商缓存是两种主要的缓存策略，合理运用它们可以显著提升网页加载速度，减少服务器负载和网络流量。Nginx 作为一款高性能的 Web 服务器，能够方便地对这两种缓存机制进行配置。  
## 一、强缓存
### （一）原理
强缓存是指浏览器在请求资源时，先在本地缓存中查找该资源。如果找到且缓存未过期，浏览器直接使用本地缓存的资源，不会向服务器发送请求。只有当缓存过期或者本地没有该资源的缓存时，浏览器才会向服务器发送请求获取资源。
### （二）相关字段
#### Expires
说明：这是一个 HTTP 1.0 协议中的字段，它的值是一个绝对时间的 GMT 格式日期。浏览器在接收到带有Expires字段的响应后，会将当前时间与Expires的值进行比较。如果当前时间在Expires指定的时间之前，就认为缓存未过期，直接使用本地缓存。  
示例：Expires: Thu, 15 Apr 2025 20:00:00 GMT，表示在这个时间之前，浏览器可以直接使用本地缓存的资源。
#### Cache - Control
说明：这是 HTTP 1.1 协议中用于控制缓存的字段，相比Expires更加灵活和强大。它有多个可选值，常见的与强缓存相关的值如下： 
max - age：指定缓存的最大有效时间，单位为秒。例如max - age = 3600表示缓存有效期为 1 小时。  
public：表示响应可以被任何对象（包括代理服务器等）缓存。  
private：表示响应只能被浏览器缓存，代理服务器不能缓存。  
示例：Cache - Control: max - age = 3600, public，表示该资源缓存有效期为 1 小时，并且可以被公共缓存（如代理服务器）缓存。  
### （三）Nginx 配置示例
```nginx
server {
    listen       80;
    server_name  example.com;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;

        # 针对HTML文件设置强缓存
        if ($uri ~* \.(html?)$) {
            add_header Cache - Control "max - age = 3600, private";
        }
        # 针对静态资源设置强缓存
        if ($uri ~* \.(css|js|png|jpg|jpeg|gif|svg)$) {
            add_header Cache - Control "max - age = 31536000, public";
        }
    }
}
```


在上述配置中：  
对于 HTML 文件，设置Cache - Control为max - age = 3600, private，表示 HTML 文件在浏览器本地缓存有效期为 1 小时，且只能被浏览器缓存。  
对于常见的静态资源文件（如 CSS、JavaScript、图片等），设置Cache - Control为max - age = 31536000, public，意味着这些静态资源缓存有效期为 1 年，并且可以被公共缓存（如代理服务器）缓存，大大减少了对这些资源的重复请求。

## 二、协商缓存
### （一）原理
协商缓存是浏览器与服务器之间的一种协作机制。浏览器在请求资源时，会在请求头中携带一些标识信息，告诉服务器它本地缓存的资源情况。服务器根据这些信息来判断浏览器缓存的资源是否仍然有效。如果有效，服务器返回一个特殊的响应状态码 304 Not Modified，告知浏览器可以使用本地缓存；如果无效，服务器则返回新的资源。
### （二）相关字段
#### Last - Modified：
服务器在响应资源时，会在响应头中添加Last - Modified字段，它表示该资源在服务器上的最后修改时间。
#### If - Modified - Since：
浏览器再次请求该资源时，会在请求头中带上If - Modified - Since字段，其值为上次服务器响应中Last - Modified的值。服务器接收到请求后，会将If - Modified - Since的值与资源在服务器上的实际最后修改时间进行对比。如果资源的最后修改时间没有变化，服务器返回 304 Not Modified 状态码，浏览器就会使用本地缓存；如果资源的最后修改时间发生了变化，服务器会返回 200 OK 状态码，并返回最新的资源。

#### ETag：
是服务器为资源生成的一个唯一标识，通常是根据资源的内容计算得出的哈希值等。服务器在响应头中添加ETag字段，将这个唯一标识返回给浏览器。
#### If - None - Match：
浏览器后续请求该资源时，会在请求头中带上If - None - Match字段，其值为上次服务器响应中的ETag值。服务器收到请求后，会将If - None - Match的值与服务器上该资源当前的ETag值进行比较。如果两个值相等，说明资源没有变化，服务器返回 304 Not Modified 状态码，浏览器使用本地缓存；如果不相等，服务器返回 200 OK 状态码，并返回新的资源。
### （三）Nginx 配置示例
```nginx
server {
    listen       80;
    server_name  example.com;

    location / {
        root   /var/www/html;
        index  index.html index.htm;

        # 开启Last - Modified和ETag
        add_header Last - Modified $date_gmt;
        add_header ETag $etag;
        # 让浏览器发送If - Modified - Since和If - None - Match请求头
        if_modified_since on;
        etag on;

        # 针对不同类型文件设置缓存相关参数
        location ~* \.(css|js|png|jpg|jpeg|gif|svg)$ {
            # 缓存时间，可以根据实际情况调整
            expires 30d;
            # 对静态资源开启协商缓存
            add_header Cache - Control "public, max - age = 2592000";
        }
    }
}
```
在上述配置中：  
add_header Last - Modified $date_gmt;和add_header ETag $etag;用于在响应头中添加Last - Modified和ETag字段。
if_modified_since on;和etag on;分别开启了If - Modified - Since和If - None - Match的功能。  
对于常见的静态资源文件类型（如.css、.js、图片等），设置了expires 30d表示缓存 30 天，同时通过add_header Cache - Control "public, max - age = 2592000";进一步设置了缓存控制，结合协商缓存，能更好地管理静态资源的缓存情况。  
## 三、强缓存与协商缓存的结合使用
在实际应用中，通常会同时使用强缓存和协商缓存，充分发挥它们的优势。一般的策略是：  
对于不常更新的静态资源，如网站的样式表、脚本文件、图片等，设置较长的强缓存时间（如一年），同时开启协商缓存。这样在缓存有效期内，浏览器直接使用本地缓存，减少网络请求；当缓存过期后，通过协商缓存判断资源是否真的有更新，避免不必要的资源下载。  
对于动态生成的页面或者更新频繁的资源，设置较短的强缓存时间或者不设置强缓存，主要依靠协商缓存来确定是否使用本地缓存，确保用户能及时获取到最新内容。  
通过合理配置 Nginx 来实现强缓存和协商缓存的有效结合，可以显著提升网站的性能和用户体验，同时减轻服务器的负载和网络带宽压力。  