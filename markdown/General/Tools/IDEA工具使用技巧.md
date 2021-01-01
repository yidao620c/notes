# IDEA工具使用技巧

## 推荐插件

1. `JavaDoc`，可以一键生成标准化的JavaDoc注释文档，非常方便。
1. `Grep Console`。控制台输出日志时可配置日志级别的颜色。
1. `Maven Helper`。用于分析jar包依赖，可以快速找到冲突的jar包。
1. `Alibaba Java Coding Guidelines`。静态代码检查插件，可对整个项目或单个文件进行检查。
1. `JRebel`。热加载神器，多的我就不讲了。

## 设置保护色

保护眼睛的颜色：`#C7EDCC`

代码中禁用`import *`设置。
`File > Settings > Editor > Code Style > Java > Imports >
"Class count to use import with '*'" and "Names count to use static import with '*'"`， 设置成9999就行了。

IDEA黑色主题：OneDark，下载后自己再修改修改一下颜色，搞成最舒服的就行了。

## IDEA自定义带JavaDoc的getter和setter模板

IDEA里面按`Alt`+`Insert`可弹出生成`getter/setter`方法模板的提示，默认的模板并不会带JavaDoc。可自定义模板。

![](https://xnstatic-1253397658.file.myqcloud.com/20201203_idea01.jpg)

### getter模板

```
/**
 * Gets the value of $field.name
 * @return the value of $field.name
 */
public ##
#if($field.modifierStatic)
static ##
#end
$field.type ##
#set($name = $StringUtil.capitalizeWithJavaBeanConvention($StringUtil.sanitizeJavaIdentifier($helper.getPropertyName($field, $project))))
#if ($field.boolean && $field.primitive)
  is##
#else
  get##
#end
${name}() {
  return $field.name;
}
```

### setter模板

```
/**
 * Sets the $field.name
 * <p>You can use get$StringUtil.capitalizeWithJavaBeanConvention($StringUtil.sanitizeJavaIdentifier($helper.getPropertyName($field, $project)))() to get the value of $field.name</p>
 * @param $field.name $field.name
 */
#set($paramName = $helper.getParamName($field, $project))
public ##
#if($field.modifierStatic)
    static ##
#end
void set$StringUtil.capitalizeWithJavaBeanConvention($StringUtil.sanitizeJavaIdentifier($helper.getPropertyName($field, $project)))($field.type $paramName) {
#if ($field.name == $paramName)
    #if (!$field.modifierStatic)
        this.##
    #else
        $classname.##
    #end
#end
$field.name = $paramName;
}
```

## IDEA不能下载maven依赖包的源码

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

