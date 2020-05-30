# nginx
同一个服务器，多个网站对应多个域名。
```
server {
    listen       80;
    server_name www.kuzhikeji.com kuzhikeji.com;
    location / {
       proxy_pass http://39.105.12.152:80;
      }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
server {
    listen       80;
    server_name 39.105.12.152;
    location / {
       proxy_pass http://39.105.12.152:8080;
      }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}

server {
    listen       80;
    server_name www.begoodluck.com begoodluck.com;
    location / {
       proxy_pass http://39.105.12.152:8081;
      }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}

server {
    listen       80;
    server_name www.zhangxiao.live zhangxiao.live;
    location / {
       proxy_pass http://39.105.12.152:8090;
      }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

