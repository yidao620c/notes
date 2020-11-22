# MySQL8遇到的一些问题

springboot应用使用MySQL8之后，启动出现异常。现在总结一下。

## 无法加载认证插件

`Unable to load authentication plugin 'caching_sha2_password'`

在网上搜索后发现是由于MySQL在8.0后验证方式由`mysql_native_password`变为`caching_sha2_password`，
所以连接时会报这个错。

解决方法：在命令行中进入mysql后运行
```sql
alter user root@localhost identified with mysql_native_password by 'password';
```

## 未知的系统参数

`com.mysql.cj.core.exceptions.CJException: Unknown system variable 'query_cache_size'`

这是由于mysql的驱动版本号不匹配引起的。

解决办法：可以到这里<https://mvnrepository.com/artifact/mysql/mysql-connector-java>
查看对应的驱动版本号写上即可，mysql 8.0.13可以用如下驱动：
```xml
<dependency>
     <groupId>mysql</groupId>
     <artifactId>mysql-connector-java</artifactId>
     <version>8.0.13</version>
     <scope>runtime</scope>
</dependency>
```
