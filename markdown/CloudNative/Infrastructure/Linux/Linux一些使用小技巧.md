# Linux一些使用小技巧

## SSH客户端连接超时退出

使用类似XShell这样的客户端工具连接上主机后，过段时间就会自动断开。 报错`timed out waiting for input auto-logout`。

解决办法是编辑`/etc/profile`，增加如下内容：

```bash
export TMOUT=8640
```

然后执行`source /etc/profile`即可。

## 多文件合并

```bash
awk 'print $0' file1 file2 file3 >merge.txt
```

## 查看Linux机器的CPU个数、核数、逻辑CPU个数。

    总核数 = 物理CPU个数 * 每颗物理CPU核数。
    总逻辑CPU数 = 总核数 * 超线程数。

查看物理CPU个数：

```bash
cat /proc/cpuinfo | grep "physical id" | sort | uniq | wc -l
```

查看每个物理CPU的核数：

```bash
cat /proc/cpuinfo | grep "cpu cores" | uniq
```

查看逻辑CPU的个数：

```bash
cat /proc/cpuinfo | grep "processor" | wc -l
```

## find命令排除某几个文件夹

```bash
find / -not \( -path /opt/docker -prune \) -not \( -path /proc -prune \) -nouser
```

## 关闭IPv6
1、使用ifconfig命令查看网卡信息，如果出现inet6 fe80::20c:29ff:fed0:3514，说明机器开启了ipv6

2、编辑/etc/sysctl.conf配置，增加net.ipv6.conf.all.disable_ipv6=1

3、编辑/etc/sysconfig/network配置，增加 NETWORKING_IPV6=no，保存并退出

4、编辑/etc/sysconfig/network-scripts/ifcfg-eno16777736，确保IPV6INIT=no，ifcfg-eno16777736是根据自己机器的，
实际网卡信息来看，不是固定的

5、关闭防火墙的开机自启动
```
systemctl disable firewalld
systemctl stop firewalld
```

6、执行sysctl -p或者reboot重启命令

7、再次使用ifconfig进行验证，只剩下ipv4，ipv6消失了，关闭成功


