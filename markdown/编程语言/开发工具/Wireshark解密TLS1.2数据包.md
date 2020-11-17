# Wireshark解密TLS1.2数据包

对于HTTPS通信的数据包，如果想要Wireshark在抓包过程中自动解密，则要求密钥套件算法中不能使用DH或DHE算法。
可将Nginx的密钥套件配置修改成如下：
```nginx
ssl_ciphers HIGH:!aNULL:!DH;!DHE:!ECDH;
```

下载Nginx的私钥或者导出私钥和证书为nginx.p12格式的正式库文件。
然后下载最新的Wireshark3.2.7，选择`编辑 》 首选项 》 protocols 》 TLS`，配置私钥或PCKS12格式的证书库即可。

