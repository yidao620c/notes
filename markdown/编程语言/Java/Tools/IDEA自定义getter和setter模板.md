# IDEA自定义带JavaDoc的getter和setter模板

IDEA里面按Alt+Insert可弹出生成getter/setter方法模板的提示，默认的模板并不会带JavaDoc。可自定义模板。

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
