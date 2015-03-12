---

layout: post
title: Raspberry Pi 2 首次安装
description: "Raspberry Pi 2 首次安装"
modified: 2015-03-11
tags: [sample post]
image:
  feature: abstract-12.jpg
share: false
---
### Raspberry Pi 2 安装

>系统自带的`vi`编辑器，在我的键盘上会有错误，所以安装`vim`，当然你也可以使用`nano`.	
	
#### 系统安装

系统启动时发现了两条错误。
{% highlight bash %}
[FAIL] startpar: service(s) returned failure: hostname.sh ... failed!
[warn] Kernel lacks cgroups or memory controller not avaiable,not starting cgroups.
{% endhighlight %}
第一条解决办法
{% highlight bash %}
vim /etc/hostname
{% endhighlight %}	
确保没有特殊字符，貌似只能使用字母，数字和`-`（是短横线，不是下划线）
	
第二条解决方法：
{% highlight bash %}
sudo vim /boot/cmdline.txt
cgroup_enable=memory (before elevator=deadline)
{% endhighlight %}	
使用`root`账号运行
{% highlight bash %}
sudo -s	
{% endhighlight %}
更新Packages Lists和执行升级
{% highlight bash %}
apt-get update
apt-get upgrade
{% endhighlight %}	
#### 看门狗（watchdog）

1、加载看门狗模块，编辑`/etc/modules`增加`bcm2708_wdog`
{% highlight bash %}
sudo modprobe bcm2708_wdog
sudo vim /etc/modules
{% endhighlight %}	
增加一行
{% highlight bash %}	
bcm2708_wdog
{% endhighlight %}	
2、安装系统软件和看门狗程序
{% highlight bash %}
sudo apt-get install chkconfig watchdog
{% endhighlight %}	
3、配置看门狗
{% highlight bash %}	
sudo vim /etc/watchdog.conf
{% endhighlight %}	
配置说明
{% highlight bash %}
max-load-1 = 24                 #一分钟内最大进程数
watchdog-device = /dev/watchdog #对应树莓派的硬件看门狗
temperature-device = /sys/class/thermal/thermal_zone0/temp #读取CPU温度
max-temperature = 80000  		#超过80度就重启
{% endhighlight %}
4、配置开机自启
{% highlight bash %}	
chkconfig watchdog on
sudo /etc/init.d/watchdog start
{% endhighlight %}	
看到出现
{% highlight bash %}	
[ ok ] Stopping watchdog keepalive daemon....
[ ok ] Starting watchdog daemon....
{% endhighlight %}
就证明启动成功了。

至此树莓派就算已经安装完成。由于我们的树莓派不可能随时连接着显示器，鼠标键盘，为了方面使用桌面环境，就需要安装`VNC`来开启远程桌面。

#### 安装VNC

1、安装VNC
	
	sudo apt-get install tightvncserver
	
安装完成后，最好设置`vnc`开机自启，下载启动脚本

	wget http://www.penguintutor.com/otherfiles/tightvncserver-init.txt
	sudo mv tightvncserver-init.txt /etc/init.d/tightvncserver
	
脚本16行`export USER='pi'`，如果你修改过系统用户名，记得将此处修改一下。（系统默认用户`pi`）

更改启动脚本用户组合用户，更改权限，增加自启

	sudo chown root:root /etc/init.d/tightvncserver	sudo chmod 755 /etc/init.d/tightvncserver
	sudo update-rc.d tightvncserver defaults
	
启动和停止命令
	
	sudo /etc/init.d/tightvncserver start
	sudo /etc/init.d/tightvncserver stop
	
启动命令运行以后，会提示你是否使用密码验证，最好设置一个密码。然后会提示：
	
	Would you like to enter a view-only password (y/n)?
	
输入`n`，不启用。

	New 'X' desktop is nomusics-pi:1

	Creating default startup script /home/pi/.vnc/xstartup
	Starting applications specified in /home/pi/.vnc/xstartup
	Log file is /home/pi/.vnc/nomusics-pi:1.log

	Starting TightVNC server for pi 
	
这样就启动完成，并且创建了编号为`1`的桌面。

下载 [VNC Viewer](http://www.realvnc.com/download/viewer/) 进行连接。

![VNC Viewer login](/images/raspberrypi/VNCViewer.png)


	
	
	