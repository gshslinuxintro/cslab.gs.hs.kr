---
layout: page
title: Contest Management System Docs
subtitle: Configuring Nginx
#toc: true
#toc_title: Custom Title
menubar: cms_menu
show_sidebar: false
---
# Configuration Example
아래는 ```nginx.conf```의 예시다.
```
upstream cws {
    ip_hash;
    keepalive 500;
    server localhost:18888;
}
upstream aws {
    ip_hash;
    keepalive 50;
    server localhost:18889;
}
upstream rws {
    keepalive 500;
    server localhost:18890;
}
server {
    listen 80;
    server_name 115.23.235.135;
    location / {

        #alias /var/www/html/;
        root   /var/www/html;
        #index  index.html index.htm;
    }
    location /assets/ {
        alias /var/www/html/assets/;
    }
    location ^~ /aws/ {
        proxy_pass http://aws/;
        include proxy_params;
        proxy_redirect http://$host/ /aws/;
        proxy_redirect https://$host/ /aws/;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        client_max_body_size 800M;
        #auth_basic "AdminWebServer";
        #auth_basic_user_file /etc/nginx/htpasswd;
    }
    location ^~ /rws/ {
        proxy_pass http://rws/;
        include proxy_params;

        proxy_redirect http://$host/ /rws/;
        proxy_redirect https://$host/ /rws/;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_buffering off;
        #auth_basic "RankingWebServer";
        #auth_basic_user_file /etc/nginx/htpasswd;
    }
    location ^~ /cws/ {
        proxy_pass http://cws/;
        include proxy_params;
        proxy_redirect http://$host/ /cws/;
        proxy_redirect https://$host/ /cws/;
        include proxy_params;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        client_max_body_size 500M;
    }
    location ^~ /status/ {
        proxy_pass http://status/;
        include proxy_params;
        proxy_redirect http://$host/ /status/;
        proxy_redirect https://$host/ /status/;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        client_max_body_size 500M;
        auth_basic "Status";
        auth_basic_user_file /etc/nginx/htpasswd;
    }

}
```