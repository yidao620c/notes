# Tomcat证书双向认证

Tomcat8的配置

``` xml
<Connector port="9443" protocol="org.apache.coyote.http11.Http11NioProtocol" maxThreads="150" SSLEnabled="true">
    <UpgradeProtocol className="org.apache.coyote.http2.Http2Protocol" />
    <SSLHostConfig
            truststoreFile="conf/root.p12"
            truststorePassword="222222"
            truststoreType="PKCS12"
            certificateVerification="required"
            truststoreAlgorithm="SunX509"
            sslProtocol="TLS">
        <Certificate certificateKeystoreFile="conf/server.p12"
           certificateKeystorePassword="222222"
           certificateKeystoreType="PKCS12" type="RSA" />
    </SSLHostConfig>
</Connector>
```

或者是下面的配置：

``` xml
<Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol" SSLEnabled ="true" 
        sslProtocol ="TLS" maxThreads="150"
        acceptCount="100" maxHttpHeaderSize="8192" maxKeepAliveRequests="100" scheme="https" secure="true"
        keystoreFile="conf/server.p12" keystorePass="333333" keystoreType="PKCS12"
        clientAuth="true" truststoreFile="root.p12" truststorePass="66666" truststoreType="PKCS12" truststoreAlgorithm="SunX509"
        ciphers="TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256"
        maxPostSize="10240" connectionTimeout="20000" allowTrace="false" xpoweredBy="false" 
        server="WebServer" URIEncoding="UTF-8" sslEnabledProtocols="TLSv1.2,TLSv1.3"/>
```

