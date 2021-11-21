# RSA加解密

## 通过openssl工具实现

1）公钥加密（注意：如果使用证书加密的话，需要使用`-certin`选项，`-inkey`指定证书文件）

```bash
openssl rsautl -encrypt -in test.txt -out test.enc -inkey public.crt -pubin
```

2）私钥解密

```bash
openssl rsautl -decrypt -in test.enc -out test.dec -inkey private.crt
```
