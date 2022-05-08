# Nginx日志打印请求头和响应头

```nginx
http {
    log_format log_req_resp '$remote_addr | $http_x_real_ip | $http_x_forwarded_for'
    '[$time_local]' '"$request" $status $body_bytes_sent' "$http_referer" "$http_user_agent"
    $request_time $content_type $host;
    server {
        access_log logs/access.log log_req_resp;
    }
}
```

> 注意，对于自定义的请求头或响应头，使用`http_`开头即可。
>
>