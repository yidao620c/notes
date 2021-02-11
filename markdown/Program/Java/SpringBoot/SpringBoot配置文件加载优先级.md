# SpringBoot配置文件加载优先级

Spring Boot 所提供的配置优先级顺序比较复杂。如果Spring Boot在优先级更高的位置找到了配置，
那么它就会无视优先级低的配置。按照优先级从高到低的顺序，具体的列表如下所示：

1. 命令行参数。
   SpringApplication 类默认会把以`--`开头的命令行参数转化成应用中可以使用的配置参数， 
   如`--name=Alex`会设置配置参数`name`的值为`Alex`。如果不需要这个功能，
   可以通过`SpringApplication.setAddCommandLineProperties(false)`禁用解析命令行参数
2. 通过`System.getProperties()`获取的 Java 系统属性。
3. 操作系统环境变量。
4. 从`java:comp/env`得到的JNDI属性。
5. 通过`RandomValuePropertySource`生成的`random.*`属性。
6. 应用 Jar 文件之外的属性文件。(通过`--spring.config.location`参数)
   启动Jar包的时候, 指定一个外部配置文件：
   `java -jar demo.jar --spring.config.location=/xxx/xxx.properties`
7. 应用 Jar 文件内部的属性文件。 
   SpringApplication 类会搜索并加载 application.properties 文件来获取配置属性值。
   SpringApplication 类会在下面位置搜索该文件。
   > 1-当前目录的“/config”子目录。\
   > 2-当前目录。\
   > 3-classpath 中的“/config”包。\
   > 4-classpath\
   > 上面的顺序也表示了该位置上包含的属性文件的优先级。优先级按照从高到低的顺序排列
8. 在应用配置 Java 类（包含`@Configuration`注解的 Java 类）中通过`@PropertySource`
   注解声明的属性文件。 当`@PropertySource`中配置了多个配置文件时，后面的优先级高于前面的。
9. 通过`SpringApplication.setDefaultProperties`声明的默认属性。

可以使用`spring.profiles.active`来指定profile，
spring boot会读取`application-{profile}.properties`中的配置项覆盖`application.properties`
中相同的配置项。