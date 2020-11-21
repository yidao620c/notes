# SpringBoot应用启动脚本

对于SpringBoot应用的启动非常方便，自己用Shell写了两个启动脚本。一个是更新源码编译方式启动，一个是直接运行jar包。
是两个很常见的使用场景。

## 从源码库拉取最新源码编译后运行脚本

```bash
#!/bin/bash
# 从源码库拉取最新源码编译后运行脚本
# 先clone相应的分支下来：
# git clone ssh://git@xx.xx.xx.xx:7999/xxx.git
# 远程调试启动：
# nohup java -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005 -Xms512m -Xmx1024m -jar -Dspring.profiles.active=${profile} ${jarfile} >/dev/null 2>&1 &

function start {
    profile="$1"
    echo "启动环境profile=${profile}"
    jarfile=$(ls target/*.jar)
    if [[ "$?" == "0" ]]; then
        stop $profile $jarfile
    fi
    branch=$(git branch |awk '{print $2}')
    git pull origin ${branch}
    echo "更新完代码开始重新打包"
    mvn clean && mvn clean && mvn package -DskipTests=true
    if [[ "$?" != "0" ]]; then
        echo "编译出错，退出！"
        exit 1
    fi
    echo "nohup java -Xms1024m -Xmx1024m -jar -Dspring.profiles.active=${profile} ${jarfile} >temp.log 2>&1 &"
    nohup java -Xms1024m -Xmx1024m -jar -Dspring.profiles.active=${profile} ${jarfile} >temp.log 2>&1 &
    echo "启动应用中，请查看日志文件..."
}

function stop {
    profile="$1"
    jarfile="$2"
    ps aux | grep "${jarfile}" | grep "spring.profiles.active=${profile}" | grep -v grep > /dev/null
    if [[ "$?" == "0" ]]; then
        echo "该应用还在跑，我先停了它"
        pid=$(ps aux | grep "${jarfile}" | grep "spring.profiles.active=${profile}" | grep -v grep |awk '{print $2}')
        if [[ "$pid" != "" ]]; then
            kill -9 $pid
        fi
        echo "停止应用成功..."
    fi
}

if [[ "$1" == "start" ]]; then
    if [[ "$#" < 2 ]]; then
        echo "请输入正确参数：./run.sh start {profile}"
        exit 1
    fi
    profile="$2"
    if [[ "$profile" != "dev" && "$profile" != "test" && "$profile" != "show" && "$profile" != "prod" ]]; then
        echo "参数错误，请输入正确的profile参数，使用方法："
        echo "./run.sh start {profile}    ==> 启动应用，{profile}取值：dev|test|show|prod"
        exit 1
    fi
    start "${profile}"
elif [[ "$1" == "stop" ]]; then
    if [[ "$#" < 2 ]]; then
        echo "请输入正确参数：./run.sh stop  {profile}"
        exit 1
    fi
    profile="$2"
    if [[ "$profile" != "dev" && "$profile" != "test" && "$profile" != "show" && "$profile" != "prod" ]]; then
        echo "参数错误，请输入正确的profile参数，使用方法："
        echo "./run.sh stop {profile}     ==> 停止应用，{profile}取值：dev|test|show|prod"
        exit 1
    fi
    jarfile=$(ls target/*.jar)
    stop $profile $jarfile
else
    echo "参数错误，使用方法：{}参数是必填的，[]参数可选"
    echo "./run.sh start {profile}    ==> 启动应用，{profile}取值：dev|test|show|prod"
    echo "./run.sh stop  {profile}    ==> 停止应用，{profile}取值：dev|test|show|prod"
    exit 1
fi
```

## 直接运行jar包

```bash
#!/bin/bash
# 直接运行jar包
# 远程调试启动：
# nohup java -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005 -Xms512m -Xmx1024m -jar -Dspring.profiles.active=${profile} ${jarfile} >/dev/null 2>&1 &

function start {
    profile="$1"
    echo "Running profile=${profile}"
    jarfile=$(ls *.jar)
    if [[ "$?" == "0" ]]; then
        stop $profile $jarfile
    fi
    echo "nohup java -Xms1024m -Xmx1024m -jar -Dspring.config.location=./application.yml -Dspring.profiles.active=${profile} ${jarfile} >temp.log 2>&1 &"
    nohup java -Xms1024m -Xmx1024m -jar -Dspring.config.location=./application.yml -Dspring.profiles.active=${profile} ${jarfile} >temp.log 2>&1 &
    echo "启动应用中，请查看temp.log日志文件..."
}

function stop {
    profile="$1"
    jarfile="$2"
    ps aux | grep "${jarfile}" | grep "spring.profiles.active=${profile}" | grep -v grep > /dev/null
    if [[ "$?" == "0" ]]; then
        echo "该应用还在跑，我先停了它"
        pid=$(ps aux | grep "${jarfile}" | grep "spring.profiles.active=${profile}" | grep -v grep |awk '{print $2}')
        if [[ "$pid" != "" ]]; then
            kill -9 $pid
        fi
        echo "停止应用成功..."
    fi
}

if [[ "$1" == "start" ]]; then
    if [[ "$#" < 2 ]]; then
        echo "请输入正确参数：./run.sh start {profile}"
        exit 1
    fi
    profile="$2"
    if [[ "$profile" != "dev" && "$profile" != "test" && "$profile" != "show" && "$profile" != "prod" ]]; then
        echo "参数错误，请输入正确的profile参数，使用方法："
        echo "./run.sh start {profile}    ==> 启动应用，{profile}取值：dev|test|show|prod"
        exit 1
    fi
    start "${profile}"
elif [[ "$1" == "stop" ]]; then
    if [[ "$#" < 2 ]]; then
        echo "请输入正确参数：./run.sh stop  {profile}"
        exit 1
    fi
    profile="$2"
    if [[ "$profile" != "dev" && "$profile" != "test" && "$profile" != "show" && "$profile" != "prod" ]]; then
        echo "参数错误，请输入正确的profile参数，使用方法："
        echo "./run.sh stop {profile}     ==> 停止应用，{profile}取值：dev|test|show|prod"
        exit 1
    fi
    jarfile=$(ls *.jar)
    stop $profile $jarfile
else
    echo "参数错误，使用方法："
    echo "./run.sh start {profile}    ==> 启动应用，{profile}取值：dev|test|show|prod"
    echo "./run.sh stop  {profile}    ==> 停止应用，{profile}取值：dev|test|show|prod"
    exit 1
fi
```
