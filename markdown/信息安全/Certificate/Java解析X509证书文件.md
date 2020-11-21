# Java解析X509证书文件

## 通过PKCS12格式的证书库文件获取证书对象

```java
InputStream inStream = new FileInputStream("c:/certificate.p12");

KeyStore ks = KeyStore.getInstance("PKCS12");
ks.load(inStream, "password".toCharArray());  

String alias = ks.aliases().nextElement();
certificate = (X509Certificate) ks.getCertificate(alias);
System.out.println(certificate .getNotAfter());
```

## 通过pem文件获取证书对象

```java
CertificateFactory fact = CertificateFactory.getInstance("X.509");
X509Certificate certificate = (X509Certificate) fact.generateCertificate(new FileInputStream ("conf/server.crt"));
```

## 获取证书的公钥
```java
PublicKey pk = certificate.getPublicKey();
```

