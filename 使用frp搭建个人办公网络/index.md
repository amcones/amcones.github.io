# 使用frp搭建个人办公网络


<!--more-->

# 使用frp搭建个人办公网络
假如你有一台云服务器，除了建站，还能够做什么有意思的事情？当然是使用**frp**将个人的所有计算机资源进行互联啦！

使用星型拓扑结构，将所有家用电脑连到frp服务器上，然后就可以通过任何一个联网终端对**整个办公网络**进行SSH或者VNC连接。

重点在于：
- 安装并配置服务端、客户端
- 注意端口畅通
- 开机自启，保护进程

本文配置基于：服务端**ubuntu**，客户端**macOS Monterey**。

## 安装并配置服务端、客户端
首先在服务端和客户端上下载对应版本frp。
[frp下载](https://github.com/fatedier/frp/releases/tag/v0.40.0)

### 服务端配置
首先需要一台有公网ip的服务器就不用多说了。

以下命令仅供参考

Ubuntu下载frp并保存到当前文件夹的命令行
```bash
wget  https://github.com/fatedier/frp/releases/download/v0.31.1/frp_0.31.1_linux_amd64.tar.gz
```
解压tar.gz
```bash
tar -zxvf frp_0.31.1_linux_arm64.tar.gz
```
进入到文件夹下
```bash
cd frp_0.31.1_linux_arm64
```
然后服务端的配置都在frps.ini中，使用vim进行编辑
```bash
vim frps.ini
```
一般的配置如下
```ini
[common]
#bind_addr = 0.0.0.0
bind_port = 7000  //7000端口是和客户端连接时使用的端口。

//服务器查看端口账号
dashboard_port = 7500
dashboard_user = admin
dashboard_pwd = admin
```
在浏览器输入服务器ip:7500可以看到一个仪表台。

配置完成后，运行服务端即可启动服务端frp服务。
```bash
 ./frps -c ./frps.ini 
```

### 客户端
客户端配置与服务器相差不大，只不过修改的都是frpc文件。（frps的s表示server，frpc的c表示client）
```ini
[common]
server_addr = xx.xx.xx.xx   //服务端的IP地址
server_port = 7000          和服务端端口一致

[RDP]                   //远程桌面配置
type = tcp
local_ip = 0.0.0.0     //没太弄懂  填0.0.0.0和127.0.0.1都可以
local_port = 5900      //随便填，但是不能和其他端口重合，两边服务器都把这些端口打开
remote_port = 6001      //这个端口是远程连接的用户所使用的端口


[ssh]                   //连接的名字，不能重复
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000     远程连接的用户使用这个端口来访问内网机器
```
设置好后启动
```bash
 ./frpc -c ./frpc.ini 
```
在多个客户端上，只需要修改连接的名字和远程端口即可。注意服务端和客户端的端口入和出方向都要放通。

## 后台运行
不可能就这样挂着个黑终端使用，要确保终端窗口关闭后依然运行。
```bash
服务端： nohup ./frps -c frps.ini >/dev/null 2>&1 &

客户端： nohup ./frpc -c frpc.ini >/dev/null 2>&1 &

说明：>/dev/null 2>&1 &，表示丢弃。
```

## 开机自启，保护进程
由于我们要使用frp打破地域限制，往往需要一个长期稳定的连接，因此我们要设置开机自启，保护进程，做到只要计算机开机就可以访问。

在**ubuntu**上，可以用systemctl控制启动。
```bash
sudo vim /lib/systemd/system/frps.service
```
写入
```ini
[Unit]
Description=fraps service
After=network.target syslog.target
Wants=network.target

[Service]
Type=simple
#启动服务的命令（此处写你的frps的实际安装目录）
ExecStart=/你的frp安装路径/frps -c /你的frp安装路径/frps.ini

[Install]
WantedBy=multi-user.target
```
启动frps
```bash
sudo systemctl start frps
```
打开自启动
```bash
sudo systemctl enable frps
```

客户端也可以采用类似的操作，但我的客户端是**mac**，需要使用另外的方法。

配置plist实现开机自启：
```bash
touch ~/Library/LaunchAgents/frpc.plist
vim ~/Library/LaunchAgents/frpc.plist

# 以下为 frpc.plist 配置
https://streamelody.github.io/assets/attachment/frpc.plist

# 加载生效
sudo chown root ~/Library/LaunchAgents/frpc.plist
sudo launchctl load -w ~/Library/LaunchAgents/frpc.plist
```
需要注意的是，macOS开机自启服务有两个文件夹，分别为`LaunchAgents`和`LaunchDaemons`，区别在于**用户登录前后启动**。如果需要在登录之前就启动frp服务的话，就需要把plist放到`LaunchDaemons`中。

### 常用命令
重启应用：`sudo systemctl restart frps`

停止应用：`sudo systemctl enable frps`

查看日志：`sudo systemctl status frps`

## 使用
可以正常地使用控制台ssh 服务器ip:端口就可以访问内网机器，也可以使用系统自带的远程桌面来连接。

需要注意的是，如果不能正常连接，很有可能是mac的端口没有开放，可以换常用开放端口。

如果想使用ssh进行远程开发的话，无论是vsc还是gateway等似乎都只能在Linux上运行。所以，应当将作为中心节点的Linux服务器作为核心开发环境，无论是通过内网机器进行开发，或是从其它网络ssh到中心服务器上都能够得到全部的计算机资源。

---

> 作者: [amcones](https://amcones.cn)  
> URL: https://amcones.cn/%E4%BD%BF%E7%94%A8frp%E6%90%AD%E5%BB%BA%E4%B8%AA%E4%BA%BA%E5%8A%9E%E5%85%AC%E7%BD%91%E7%BB%9C/  

