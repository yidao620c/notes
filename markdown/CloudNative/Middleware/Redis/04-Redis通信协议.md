# Redis教程04-Redis通信协议

术语协议代表网络通信中客户端跟服务端直接使用的一种约定。对于Redis来讲，
这种通信协议名叫Redis Serialization Protocol（RESP，Redis序列化协议）。

不同编程语言实现的redis客户端都是通过实现这种协议来与Redis服务端通信。

## 使用netcat和Redis通信
首先启动Redis服务端，比如我就启动在本地，监听端口号为默认的6379。

1、使用netcat向Redis服务器发送PING命令：
```bash
echo -e "*1\r\n\$4\r\nPING\r\n" | nc 127.0.0.1 6379
+PONG
```

2、使用SET和INCR命令设置一个整数，并将其加1：
```bash
echo -e "*3\r\n\$3\r\nSET\r\n\$5\r\nmykey\r\n\$2\r\n99\r\n" |nc 127.0.0.1 6379
+OK
echo -e "*2\r\n\$4\r\nINCR\r\n\$5\r\nmykey\r\n" |nc 127.0.0.1 6379
:100
```

3、一次性发送多个命令给Redis服务器：
```bash
echo -e "*3\r\n\$3\r\nset\r\n\$3\r\nfoo\r\n\$3\r\nbar\r\n*2\r\n\$3\r\nget\r\n\$3\r\nfoo\r\n" \
| nc 127.0.0.1 6379
+OK
$3
bar
```

## 工作原理
由于RESP协议只包含五种类型的命令，所以理解起来并不困难。

比如我们向服务器发送`"*1\r\n\$4\r\nPING\r\n"`。该命令开头的星号表示这是一个数组类型。

* `1` 代表这个数组的大小
* `\r\n（CRLF）`是RESP中每个部分的终结符。
* `$4`之前的反斜杠是$符号的转义字符。
* `$4`表示接下来是4个字符组成的字符串。
* `PING`为字符串本身。
* `+PONG`是PING命令的响应字符串，+号表示响应是一个简单字符串类型。

接下来，INCR命令返回的结果是`:2`，数字签名的冒号表示是一个整型。

总之，客户端发送给Redis服务器的命令实际上就是一组字符串组成的RESP数组。之后按不同的命令、
使用上述五种RESP类型之一对命令进行解析响应。

## 官方参考
有关Redis协议更详细介绍，参考官网：https://redis.io/topics/protocol
