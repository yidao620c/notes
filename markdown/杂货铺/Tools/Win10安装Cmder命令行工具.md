# Win10安装Cmder命令行工具

Cmder把`conemu`，`msysgit`和`clink`打包在一起，是Windows上面命令行利器。
让你无需配置就能使用一个真正干净的Linux终端！里面可以使用大量的 linux 命令；比如 grep, curl(没有 wget)；
像vim, grep, tar, unzip, ssh, ls, bash, perl 对于爱折腾的Coder更是痛点需求。
她甚至还附带了漂亮的monokai配色主题。 作为一个压缩档的存在, 可即压即用。
你甚至可以放到USB就可以虽时带着走，连调整过的设定都会放在这个目录下。

## 安装

Cmder官网 <http://cmder.net/>，直接去官网下载二进制压缩包即可。

下载后解压缩到某个目录，注意目录中不要包含空格。

## 配置

1、把 `cmder` 加到环境变量，可以把Cmder.exe存放的目录添加到系统环境变量；加完之后,Win+r一下输入cmder即可。

2、添加 cmder 到右键菜单。
在某个文件夹中打开终端, 这个是一个(超级)痛点需求, 实际上上一步的把 cmder 加到环境变量就是为此服务的, 
在管理员权限的终端输入以下语句即可。
Win10中以管理员身份打开PoweShell，然后执行：
```bash
Cmder.exe /REGISTER ALL
```

## 解决文字重叠问题
`Win + ALT + P` 唤出设置界面 > `mian` > `font` > `monospce`,去掉那勾勾即可。

