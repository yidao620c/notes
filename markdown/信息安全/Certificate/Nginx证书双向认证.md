# Nginx证书双向认证

```nginx
server {
    listen       443 ssl;
    server_name  192.168.20.112;
    ssl_certificate conf/nginx.crt;
    ssl_certificate_key conf/nginx.key;
    ssl_password_file conf/fifo;
    ssl_protocols       TLSv1.2;
    ssl_ciphers         "ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:AES256-GCM-SHA384:AES128-GCM-SHA256:AES256-SHA256:AES256-SHA:HIGH:!MEDIUM:!LOW:!aNULL:!eNULL:!EXPORT:!DES:!MD5:!PSK:!RC4:@STRENGTH";
    ssl_prefer_server_ciphers on;
    ssl_dhparam ./ssl/dh2048.pem;

    root /usr/local/NSP/nginx/html/deployui;

    location /GatewayService/ {
        proxy_pass https://192.168.20.112:18006/GatewayService/;
        #如果证书里面配置了common name，且和proxy_pass中的ip不匹配,设置此参数为common name
        proxy_ssl_name server.xncoding.com;
        proxy_ssl_protocols TLSv1.1 TLSv1.2;
        proxy_ssl_certificate conf/nginx.crt;
        proxy_ssl_certificate_key conf/nginx.key;
        proxy_ssl_password_file conf/fifo;
        proxy_ssl_trusted_certificate conf/root.crt;
        proxy_ssl_verify on;
        proxy_ssl_verify_depth 2;

        proxy_connect_timeout 600s;
        proxy_read_timeout 600s;
        proxy_send_timeout 600s;
        default_type application/json;
        proxy_intercept_errors off;
    }
}
```
