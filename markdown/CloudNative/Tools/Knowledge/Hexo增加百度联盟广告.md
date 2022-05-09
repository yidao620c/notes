# Hexo增加百度联盟广告

博客使用的是hexo搭建，使用的next主题，但是配置文件中，没有百度联盟广告的配置项，只能自己搭建了。

## 一、获取广告JS代码
我使用的是百度广告，登录[百度联盟](https://union.baidu.com/)，代码位管理，创建代码位，
得到JS代码如下：
```javascript
<script type="text/javascript">
    var cpro_id = "u6792228";
</script>
<script type="text/javascript" src="//cpro.baidustatic.com/cpro/ui/cm.js"></script>
```

## 二、hexo添加百度广告

**新建baidu_union.swig文件**

在路径`\themes\next\layout\_macro`中添加`baidu_union.swig`文件，其内容为：
```
{% if theme.baidu_union.enabled %}
<script type="text/javascript">
    var cpro_id = "u6792228";
</script>
<script type="text/javascript" src="//cpro.baidustatic.com/cpro/ui/cm.js"></script>
{% endif %}
```

**修改 post.swig 文件**

在`\themes\next\layout\_macro\post.swig`中，在`<footer class="post-footer">`之前添加如下代码：
```
<div>
{% if not is_index %}
  {% include 'baidu_union.swig' %}
{% endif %}
</div>
```

**主题配置文件增加控制字段**

在主题配置文件`/themes/next/_config.yml`中添加以下字段开启此功能：
```yaml
baidu_union:
  enabled: true
```

完成以上设置之后，在每篇文章之后都会百度联盟广告。

## FAQ
很久没更新Hexo的文章，升级了下Node版本到v14之后，执行hexo编译时报错
```
The 'mode' argument must be integer. Received an instance of Object
```
原因是Hexo版本4.2.0只能使用Node 12.18来编译。因此需要将node版本又回退回去才行。

