# Linux一些使用小技巧

## SSH客户端连接超时退出
使用类似XShell这样的客户端工具连接上主机后，过段时间就会自动断开。
报错`timed out waiting for input auto-logout`。

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


