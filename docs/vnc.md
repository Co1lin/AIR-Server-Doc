# VNC

​	运维团队在 AIR 的部分服务器上安装了可视化图形界面，可以使用支持 VNC 协议的客户端连入。本文便是对其使用方法的简单说明。

​	目前安装了 VNC Server 的服务器有：

+ 10.0.0.14

## 配置 VNC Server

​	如果您之前没有配置过 VNC Server，首先 SSH 到上述服务器中，然后输入：

```bash
vncserver
```

​	您可能会看到类似于如下的提示：

```
Warning: discover-01:5 is taken because of /tmp/.X5-lock
Remove this file if there is no X server discover-01:5

Warning: discover-01:8 is taken because of /tmp/.X8-lock
Remove this file if there is no X server discover-01:8

New 'X' desktop is discover-01:27

Starting applications specified in /home/gaoha/.vnc/xstartup
Log file is /home/gaoha/.vnc/discover-01:27.log
```

​	注意这里 `New 'X' desktop is discover-01:27` 说明我们的 VNC 识别号为 27，请暂时记住这个 ID。

​	接着，我们要修改配置文件使得默认打开 gnome 桌面。在此之前，我们先将上述打开的服务端关闭：

```
vncserver -kill :27
```

​	通过指定上述识别 ID，我们就可以结束一个 VNC 服务端进程。

​	我们使用文本编辑器打开 `~/.vnc/xstartup`：

```
# Use nano or vim as you like
nano ~/.vnc/xstartup
vim ~/.vnc/xstartup
```

​	将其修改为：

```bash
#!/bin/sh

unset SESSION_MANAGER
exec /etc/X11/xinit/xinitrc

xrdb $HOME/.Xresources
xsetroot -solid grey
vncconfig -iconic &
x-terminal-emulator -geometry 80x24+10+10 -ls -title "$VNCDESKTOP Desktop" &
gnome-session &
```

接着，只需使用 `vncserver` 重新启动服务端进程，记住 ID 号即可。

## 安装 VNC Viewer

​	[VNC Viewer](https://www.realvnc.com/en/connect/download/viewer/) 是支持 VNC 协议的客户端。我们已在上述服务器上安装了服务端，在您的电脑上要做的，只有安装 VNC Viewer 这一件事。选择适用于您系统版本的客户端，进行安装即可。

![image-20220621203727944](https://s2.loli.net/2022/06/21/sOVNyLbw18fBWQe.png)

​	在 VNC Viewer 中添加服务器，对端 IP 地址填入 `<Server IP>:<ID>`，接着进行连接即可。

![image-20220621204540421](https://s2.loli.net/2022/06/21/mKHLszABZICoFp9.png)