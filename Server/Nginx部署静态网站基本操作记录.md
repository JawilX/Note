### 相关文件和命令
```shell
/usr/share/nginx/html/
/var/www/html/
vim /etc/nginx/nginx.conf
systemctl status nginx
systemctl enable nginx
systemctl status nginx
systemctl reload nginx
vim /var/log/nginx/access.log
vim /var/log/nginx/error.log
```

### nginx.conf
```
    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        rewrite ^(.*)$ https://$host$1 permanent;
    }
    
    server {
        listen       443 ssl http2 default_server;
        listen       [::]:443 ssl http2 default_server;
        server_name  _;
        root         /var/www/html/website;

        ssl_certificate "/etc/letsencrypt/live/www.domain.com/cert.pem";
        ssl_certificate_key "/etc/letsencrypt/live/www.domain.com/privkey.pem";
        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout  10m;
        ssl_ciphers HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers on;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }

```
