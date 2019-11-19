---
title: kibana.nginx
date: 2019-08-29 10:20:48
tags:
---

### nginx配置文件 kibana.conf

```shell
server {
    listen 80;

    server_name <YourKibanaIP>;

    auth_basic "Restricted Access";
    auth_basic_user_file /etc/nginx/htpasswd.users.kibana;

    location / {
        proxy_pass http://localhost:5601;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;        
    }
}
```


### 生成http认证用户

```
echo "kadmin:`openssl passwd -apr1`" | sudo tee -a ./htpasswd.users.kibana

```