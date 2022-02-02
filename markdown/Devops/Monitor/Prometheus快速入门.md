# Prometheus快速入门

## 安装和配置
首先安装并启动3个最基本的软件：
```
./node_exporter --web.listen-address 192.168.31.20:8080
./prometheus --config.file=prometheus.yml
./grafana-server web
```

查询的GraphQL语句：
```

```

## Grafana使用
By default, Grafana does not automatically refresh the dashboard. Queries run on 
their own schedule according to the panel settings. However, if you want to regularly 
refresh the dashboard, then click the down arrow next to the Refresh dashboard icon 
and then select a refresh interval.

