# 使用expect实现远程登录

```bash
#!/bin/bash

#########################################################################
# 演示通过expect实现批量主机操作
# 任务：以普通用户SSH登录远程主机后su切换至root再执行命令，自动完成口令输入。
# author: xiongneng
# date: 2020-09-01
# 使用方法：
# 首先准备一个配置文件config.txt，把主机IP、普通用户密码、root密码。每行一条，逗号隔开。
# 然后执行 ./xx.sh config.txt
#########################################################################

main() {
    config_file=$1
    user_name=user
    for line in $(cat $config_file); do
        IFS=',' read -r a record <<<"$line"
        ip=$(record[0])
        user_pwd=$(record[1])
        root_pwd=$(record[2])
        /usr/bin/expect <<-EOF
        set timeout 10
        log_file error.log
        spawn -noecho ssh -tq $user_name@$ip "su -c \"
        chown root:root /etc/ssh/*.key && chmod 400 /etc/ssh/*.key;
        chown root:root /etc/ssh/*.key.pub && chmod 400 /etc/ssh/*.key.pub;
        systemctl restart sshd\""
        expect {
            "*yes/no" {send "yes\r"; exp_continue}
            "*s password" {send "${user_pwd}\r"}
            "*No route to host" {send_log "\nno route to ip, =====$ip=====\n"; exit}
            timeout {send_log "\nconnect ip timeout, =====$ip=====\n"; exit}
        }
        expect {
            "*s password" {send "\nuser password error, =====$ip=====\n"; exit}
            "Password:" {send "${root_pwd}\r"; exp_continue}
            "*Permission denied" {send_log "\nroot password error, =====$ip=====\n"; exit}
        }
        
EOF
    echo "ip=$ip process finished"
    done
    echo "All tasks finish successfully."
}

if [[ "$#" < 1 ]]; then
    echo "[error] input invalid, usage: "
    echo "./xx.sh config.txt"
    exit 1
fi

main $1
```