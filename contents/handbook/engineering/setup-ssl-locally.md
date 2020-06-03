---
title: Setup SSL locally
sidebar: Handbook
showTitle: true
---

Setting up SSL locally can be useful if you're trying to debug issues.

1. Create key
```
openssl req -x509 -newkey rsa:4096 -sha256 -days 3650 -nodes \
  -keyout example.key -out example.crt -subj "/CN=secure.posthog.dev" \
  -addext "subjectAltName=DNS:secure.posthog.dev,IP:10.0.0.1"
```
2. Trust the key for Chrome/Safari
```
sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain localhost.crt
```
3. Add `secure.posthog.dev` to /etc/hosts
```
127.0.0.1 secure.posthog.dev
```
4. Install nginx and add the following config
```nginx
     upstream backend {
         server 127.0.0.1:8000;
     }
     server {
         server_name secure.posthog.dev;
         rewrite ^(.*) https://secure.posthog.dev$1 permanent;
     }
 
     server {
         listen       443 ssl;
         server_name  secure.posthog.dev;
         ssl_certificate  /Users/timglaser/dev/localhost.crt;
         ssl_certificate_key /Users/timglaser/dev/localhost.key    ;
         ssl_prefer_server_ciphers  on;
         ssl_session_cache    shared:SSL:1m;
         ssl_session_timeout  5m;
         ssl_ciphers          HIGH:!aNULL:!MD5;
         location / {
            proxy_pass http://backend;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $http_host;
            proxy_redirect off;
            proxy_set_header X-Forwarded-Proto $scheme;
         }
         location /static/ {
            proxy_pass http://127.0.0.1:8234/static/;
        }
     }
```

5. Add the following command to start nginx
```bash
nginx -p /usr/local/etc/nginx/ -c /usr/local/etc/nginx/nginx.conf
```

6. You can stop the nginx server with
```bash
nginx -p /usr/local/etc/nginx/ -c /usr/local/etc/nginx/nginx.conf -s stop
```

7. To run local development, use
```bash
bin/start-http
```