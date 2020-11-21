# IDEA不能下载maven依赖包的源码

直接点击查看源码，报错：`cannot download sources`

使用Maven命令。经过测试，好用。下载了所有POM里的依赖包的source，这点不是想要的，原来只想下载想看的依赖的source。
参考：[IDEA-165800 Can’t download dependency's source code](https://youtrack.jetbrains.com/issue/IDEA-165800)

使用如下命令行下载：
```
mvn dependency:resolve -Dclassifier=sources
```

如果只想下载指定的包，使用（多个使用逗号分隔）：
```
mvn dependency:sources -DincludeArtifactIds=commons-io,:mybatis
```

参考：[Get source JARs from Maven repository](http://stackoverflow.com/questions/2059431/get-source-jars-from-maven-repository)


