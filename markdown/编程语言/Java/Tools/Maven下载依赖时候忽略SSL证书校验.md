# Maven下载依赖时候忽略SSL证书校验

```bash
mvn clean && mvn compile -Dmaven.wagon.http.ssl.insecure=true -Dmaven.wagon.http.ssl.allowall=true -Dmaven.wagon.http.ssl.ignore.validity.dates=true
```
