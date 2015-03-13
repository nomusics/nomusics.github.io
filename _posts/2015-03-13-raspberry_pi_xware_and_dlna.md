---
layout: post
title: Raspberry Pi 2安装迅雷和DLNA
modified: 2015-03-13
image:
  feature: abstract-1.jpg
share: false
---

>如果你有空闲的硬盘，并且正好购买了树莓派的话，可以把树莓派打造成一台带远程下载的NAS服务器。

在树莓派上挂载移动硬盘，使用迅雷远程下载固件进行远程下载，Samba和DLNA进行共享服务。

准备工作：

- 一块闲置的移动硬盘（带独立供电）
- 一块树莓派(Raspberry Pi)

由于树莓派的电压只有5V 2.0A，所以最好使用带独立供电的移动硬盘，或者购买一块独立供电的集线器（强烈推荐使用集线器）。

<!--more-->

### 挂载硬盘

共有三种格式`NTFS`、`exFAT`、`ext4`，前两种格式在树莓派下比较吃CPU，所以我们使用`ext4`格式来进行，当然你也可以使用其余两种格式，安装方式都一样。

{% highlight bash %}
sudo apt-get install ntfs-3g		#NTFS格式
sudo apt-get install exfat-nofuse	#exFAT格式
{% endhighlight %}

**树莓派默认支持ext4格式，所以不用安装任何模块。**

格式化需要挂载的分区，插入树莓派以后使用`df -h`进行查看。

{% highlight bash %}
Filesystem      Size  Used Avail Use% Mounted on
rootfs           30G  2.5G   26G   9% /
/dev/root        30G  2.5G   26G   9% /
devtmpfs        427M     0  427M   0% /dev
tmpfs            87M  268K   86M   1% /run
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           173M     0  173M   0% /run/shm
/dev/mmcblk0p1   56M   15M   42M  26% /boot
none            431M     0  431M   0% /sys/fs/cgroup
/dev/sdb1       7.3G  1.7G  5.2G  25% /media/b5f19bb8-fbf8-4b18-b5d9-1abf2d4533bf
{% endhighlight %}

`/dev/sdb1` 就是我挂载的硬盘（😭 太穷了，没有移动硬盘，所有挂载了一个8G的U盘进行演示）

`NTFS`格式最后在windows下进行格式化，传说有人在树莓派下使用`mkfs.ntfs`对一个200G的硬盘进行格式化，耗时两三个小时。

把`sdb1`格式化成`ext4`格式：

**一定要记得，不可以在分区挂载之后再进行格式化！！需要将分区卸载后再进行！！否则会出错！！**

{% highlight bash %}
sudo umount /dev/sda1		#卸载分区
sudo mkfs.ext4 /dev/sda1	#格式化分区
{% endhighlight %}

新建一个目录用于挂载你们的移动硬盘

{% highlight bash %}
sudo mkdir -p /home/Share/usb
{% endhighlight %}

编辑`/etc/fstab`文件，在最后面添需要挂载的分区和挂载路径。

{% highlight bash %}
sudo vim /etc/fstab
{% endhighlight %}

{% highlight bash %}
/dev/sda1       /home/Share/usb      ext4    defaults,noatime        0       0
/dev/sda1       /home/Share/usb      ntfs    defaults,noatime,uid=1000,gid=1000        0       0
/dev/sda1       /home/Share/usb      exfat    defaults,noatime,uid=1000,gid=1000        0       0
{% endhighlight %}

**上面三行分别针对的`ext4`格式的硬盘，`NTFS`格式的硬盘，`exFAT`格式的硬盘，根据你上面格式化的格式进行选择。**

> noatime代表不记录文件访问时间，可以大大提升性能。NTFS 和 exFAT 并没有 Linux/Unix 权限系统，所以需要加上uid=1000,gid=1000指定这个文件的拥有者。[^1]

然后重新挂载分区：

{% highlight bash %}
sudo mount -a
{% endhighlight %}

再次输入`df -h`：

{% highlight bash %}
Filesystem      Size  Used Avail Use% Mounted on
rootfs           30G  2.5G   26G   9% /
/dev/root        30G  2.5G   26G   9% /
devtmpfs        427M     0  427M   0% /dev
tmpfs            87M  268K   86M   1% /run
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           173M     0  173M   0% /run/shm
/dev/mmcblk0p1   56M   15M   42M  26% /boot
none            431M     0  431M   0% /sys/fs/cgroup
/dev/sdb1       7.3G  1.7G  5.2G  25% /home/Share/usb
{% endhighlight %}

看最后一条的路径，这就表示已经挂载成功了。并且重启以后自动挂载。

验证一下是否能进入：

{% highlight bash %}
pi@nomusics-pi ~ $ cd /home/Share/usb/
pi@nomusics-pi /home/Share/usb $ ls
lost+found
pi@nomusics-pi /home/Share/usb $ 
{% endhighlight %}

> `lost+found`这个目录是使用标准的ext2/ext3档案系统格式才会产生的一个目录，目的在于当档案系统发生错误时， 将一些遗失的片段放置到这个目录下。

### 安装过程中出错
---
**在接下来的安装过程中如果出现以下错误：**

{% highlight bash %}
perl: warning: Setting locale failed.
perl: warning: Please check that your locale settings:
	LANGUAGE = (unset),
	LC_ALL = (unset),
	LC_CTYPE = "zh_CN.UTF-8",
	LANG = "en_US.UTF-8"
    are supported and installed on your system.
perl: warning: Falling back to the standard locale ("C").
{% endhighlight %}

**解决办法是：**

{% highlight bash %}
export LANG=en_US:zh_CN.UTF-8
export LC_ALL=C
source ~/.profile
{% endhighlight %}

---

### 安装迅雷固件

首先选择树莓派CPU架构能用的 [迅雷固件](http://luyou.xunlei.com/thread-12545-1-1.html){:target="_blank"} 下载。

**如果对Linux系统不熟悉的用户，最好使用VNC远程桌面进行下载。**

迅雷远程下载固件命名规则：

Xware版本号_（cpu架构_指令集_c库类型｜产品名称）.zip

树莓派选择`armel_v5te_glibc`这个版本：**Xware\*.\*.\**_armel_v5te_glibc.zip** （\*是版本号）

下载并解压：

{% highlight bash %}
sudo mkdir /home/xware
sudo unzip Xware*.*.**_armel_v5te_glibc.zip -d /home/xware
{% endhighlight %}

运行迅雷远程下载固件：

{% highlight bash %}
pi@nomusics-pi ~ $ cd /home/xware/
pi@nomusics-pi /home/xware $ ./portal
initing...
try stopping xunlei service first...
killall: ETMDaemon: no process killed
killall: EmbedThunderManager: no process killed
killall: vod_httpserver: no process killed
setting xunlei runtime env...
port: 9000 is usable.

YOUR CONTROL PORT IS: 9000

starting xunlei service...
etm path: /home/pi/xware
execv: /home/pi/xware/lib/ETMDaemon.

getting xunlei service info...
Connecting to 127.0.0.1:9000 (127.0.0.1:9000)

THE ACTIVE CODE IS: XXXXXX

go to http://yuancheng.xunlei.com, bind your device with the active code.
finished.
{% endhighlight %}

最后的`THE ACTIVE CODE IS: XXXXXX`，其中`XXXXXX`就是你的激活码。

打开[迅雷远程下载](http://yuancheng.xunlei.com/){:target="_blank"}，输入上一步中得激活码就完成了设备添加。

![Xware Active](/images/raspberrypi/xware_active.png)

我们需要迅雷远程下载开机自启，所以先创建启动脚本，然后添加到开机启动项里。

创建Xware启动脚本：

{% highlight bash %}
sudo vim /etc/init.d/xware
{% endhighlight %}

{% highlight bash %}
#!/bin/sh
#
# Xware initscript
#
### BEGIN INIT INFO
# Provides:          xware
# Required-Start:    $network $local_fs $remote_fs
# Required-Stop::    $network $local_fs $remote_fs
# Should-Start:      $all
# Should-Stop:       $all
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Start xware at boot time
# Description:       A downloader
### END INIT INFO

do_start()
{
        /home/xware/portal
}

do_stop()
{
        /home/xware/portal -s
}

case "$1" in
  start)
    do_start
    ;;
  stop)
    do_stop
    ;;
esac
{% endhighlight %}

设置开机自启：

{% highlight bash %}
sudo chown root:root /etc/init.d/xware
sudo chmod 755 /etc/init.d/xware 
sudo chkconfig xware on
{% endhighlight %}

检查开机启动项：

{% highlight bash %}
sudo chkconfig --list
{% endhighlight %}

查找`xware`项：

{% highlight bash %}
xware                     0:off  1:off  2:on   3:on   4:on   5:on   6:off
{% endhighlight %}

3，4，5状态为on，就说明设置成功了。

### DLNA

### Samba

[^1]: 来源[利用树莓派组建支持迅雷离线下载的NAS](http://www.dozer.cc/2014/05/raspberry-pi-nas/){:target="_blank"}

