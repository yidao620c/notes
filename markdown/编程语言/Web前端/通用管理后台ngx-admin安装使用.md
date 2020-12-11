# 通用管理后台ngx-admin安装使用

## 介绍

The most popular admin dashboard based on Angular 9+ and Nebular with Eva Design System support.
Free and Open Source for personal and commercial purposes.

## 安装和使用

我的环境是Win10环境，首先你得准备如下这些工具：

1. Git - <https://git-scm.com>
1. Node.js - <https://nodejs.org>. Please note the version should be >=8
1. Npm - Node.js package manager, comes with Node.js. Please make sure npm version is >=5

Win10上面还需要安装编译环境，并且还必须得使用管理员模式打开PowShell运行才行。点击Win开始菜单，
然后输入PowerShell，在右边会出现`以管理员身份运行`，点击打开即可。
```bash
npm config set registry https://registry.npm.taobao.org
npm install -g node-gyp
npm install --global --production windows-build-tools
```

然后我也不用git去clone了，因为访问github实在太慢。就直接去网站<https://github.com/akveo/ngx-admin.git>下载zip压缩包。
然后解压到当前目录，进入`nginx-admin`，执行安装命令：
```bash
npm i
```

没发现Error则表示一切顺利，完成ngx-amdin的安装。

## 启动

执行命令`npm start`以开发者模式打开，然后浏览器打开<http://localhost:4200>就能访问了。

如果想发布生产环境的包，则执行
```bash
npm run build:prod
```

最终会在`dist`文件夹生产目标静态资源文件，直接复制到web服务器比如Nginx中即可。

