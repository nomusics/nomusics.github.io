---
layout: post
title: Raspberry Pi 2安装迅雷和DLNA（二）
modified: 2015-03-16
tags: [Raspberry Pi,DLNA,Xware]
image:
  feature: abstract-1.jpg
  thumb: raspberrypi/raspberrypi_install_dlna_thumb.jpg
share: false
---
名词解释：

> DLNA的全称是DIGITAL LIVING NETWORK ALLIANCE(数字生活网络联盟)， 其宗旨是Enjoy your music, photos and videos, anywhere anytime， DLNA(Digital Living Network Alliance) 由索尼、英特尔、微软等发起成立、旨在解决个人PC，消费电器，移动设备在内的无线网络和有线网络的互联互通，使得数字媒体和内容服务的无限制的共享和增长成为可能，目前成员公司已达280多家。[^1]

应用场景：

> 我坐在马桶上点我手机里的歌曲，客厅里电脑的音箱立刻播放了我手机里的音乐，下一首快进暂停都可以手机控制，或者你拍了一段高清的视频，用手机那小小的屏幕实在是没有阖家观赏的效果，不怕，我从容的选择电脑播放，在手机里点到视频，家里23寸的显示器立刻流畅的播放刚刚拍下的热腾腾的高清视频。而且在手机也可以浏览电脑里的图片音乐和视频.[^1]

<!--more-->

### DLNA编译

在树莓派上通过`apt-get install minidlna`安装的`minidlna`版本为`1.0.24` 不支持rm，rmvb格式，据传说官方这么做是为了兼容老硬件。我们的树莓派（Raspberry Pi）解析1080P的rm，rmvb格式毫无压力，所以我们通过更改安装包，重新编译安装使`minidlna`支持rm，rmvb格式的片源。

##### 准备工作

下载`minidlna`安装包

- [项目地址](http://sourceforge.net/projects/minidlna/){:target="_blank"}
- 最新安装包：[minidlna-1.1.4.tar.gz](http://sourceforge.net/projects/minidlna/files/minidlna/1.1.4/minidlna-1.1.4.tar.gz/download){:target="_blank"} (494.5 kB)

安装编译工具

{% highlight bash %}
sudo apt-get install build-essential gettext ffmpeg libavutil-dev libavcodec-dev libavformat-dev libjpeg-dev libsqlite3-dev libexif-dev libid3tag0-dev libbogg-dev libvorbis-dev libflac-dev libFLAC-dev libvorbis-dev
{% endhighlight %}

#### 修改源码[^2]

> **注意：仔细核对代码，复制粘贴后是否有多余的特殊字符，不要出现特殊符号之类的，我第一次安装因为多了一个特殊符号，整整折腾了一天才找到原因。 ⊙﹏⊙b汗**

下载，解压，进入目录：

{% highlight bash %}
sudo wget http://sourceforge.net/projects/minidlna/files/minidlna/1.1.4/minidlna-1.1.4.tar.gz/download
sudo tar zxvf minidlna-1.1.4.tar.gz
cd minidlna-1.1.4
{% endhighlight %}

编辑 `metadata.c`

查找代码：

{% highlight c %}
else if( strncmp(ctx->iformatctx->name, "matroska", 8) == 0 )
	xasprintf(&m.mime, "video/x-matroska");
else if( strcmp(ctx->iformatctx->name, "flv") == 0 )
	xasprintf(&m.mime, "video/x-flv");
{% endhighlight %}

之后添加：

{% highlight c %}
else if( strcmp(ctx->iformat->name, "rm") == 0 )
	xasprintf(&m.mime, "video/x-pn-realvideo");
else if( strcmp(ctx->iformat->name, "rmvb") == 0 )
	xasprintf(&m.mime, "video/x-pn-realvideo");
{% endhighlight %}

增加后的代码：

{% highlight c %}
else if( strcmp(ctx->iformat->name, "mov,mp4,m4a,3gp,3g2,mj2") == 0 && ends_with(path, ".mov") )
	xasprintf(&m.mime, "video/quicktime");
else if( strncmp(ctx->iformat->name, "matroska", 8) == 0 )
	xasprintf(&m.mime, "video/x-matroska");
else if( strcmp(ctx->iformat->name, "flv") == 0 )
	xasprintf(&m.mime, "video/x-flv");
else if( strcmp(ctx->iformat->name, "rm") == 0 )
	xasprintf(&m.mime, "video/x-pn-realvideo");
else if( strcmp(ctx->iformat->name, "rmvb") == 0 )
	xasprintf(&m.mime, "video/x-pn-realvideo");
{% endhighlight %}

编辑 `upnpglobalvars.h`

查找代码：

{% highlight c %}
"http-get:*:application/ogg:*"
{% endhighlight %}

替换代码：

{% highlight c %}
"http-get:*:application/ogg:*," \
"http-get:*:video/x-pn-realvideo:*"
{% endhighlight %}

替换后的代码：

{% highlight c %}
"http-get:*:audio/mp4:*," \
"http-get:*:audio/x-wav:*," \
"http-get:*:audio/x-flac:*," \
"http-get:*:application/ogg:*" \
"http-get:*:video/x-pn-realvideo:*"
{% endhighlight %}

> 注意：此处修改应为替换，因为 `"http-get:*:application/ogg:*,"` 下面又多了一样，所以此行结尾处应该添加 `\`。

编辑 `utils.c`

查找代码：

{% highlight c %}
ends_with(file, ".flv") || ends_with(file, ".xvid")  ||
{% endhighlight %}

之后添加：

{% highlight c %}
ends_with(file, ".rm")  || ends_with(file, ".rmvb")  ||
{% endhighlight %}

添加后的代码

{% highlight c %}
ends_with(file, ".avi") || ends_with(file, ".divx")  ||
ends_with(file, ".asf") || ends_with(file, ".wmv")   ||
ends_with(file, ".mp4") || ends_with(file, ".m4v")   ||
ends_with(file, ".mts") || ends_with(file, ".m2ts")  ||
ends_with(file, ".m2t") || ends_with(file, ".mkv")   ||
ends_with(file, ".vob") || ends_with(file, ".ts")    ||
ends_with(file, ".flv") || ends_with(file, ".xvid")  ||
ends_with(file, ".rm")  || ends_with(file, ".rmvb")  ||
{% endhighlight %}

#### 编译安装

{% highlight bash %}
./autogen.sh
./configure --disable-nls
sudo make && make install
{% endhighlight %}

**可能出现的错误提示**

错误提示：

{% highlight bash %}
You must have autoconf installed to compile minidlna.
{% endhighlight %}

{% highlight bash %}
You must have automake installed to compile minidlna.
{% endhighlight %}

解决办法：

{% highlight bash %}
sudo apt-get install autoconf
{% endhighlight %}

错误提示：

{% highlight bash %}
Generating configuration files for minidlna, please wait....
autoreconf: Entering directory '.'
autoreconf: running: autopoint --force
Can't exec "autopoint": No such file or directory at /usr/share/autoconf/Autom4te/FileUtils.pm line 345.
autoreconf: failed to run autopoint: No such file or directory
autoreconf: autopoint is needed because this package uses Gettext
{% endhighlight %}

解决办法：

{% highlight bash %}
sudo apt-get install autopoint
{% endhighlight %}

#### 配置`minidlna`

编译安装的不会自动拷贝配置文件：`minidlna.conf`，需要自己拷贝过去。

配置文件：

{% highlight bash %}
sudo cp minidlna.conf /etc/minidlna.conf
{% endhighlight %}

编辑配置文件：

{% highlight bash %}
sudo vim /etc/minidlna.conf
{% endhighlight %}

{% highlight bash %}
media_dir=/home/Share/usb/	#媒体文件夹位置
friendly_name=My DLNA Server	#DLNA名称
db_dir=/var/cache/minidlna	#DLNA缓存数据库文件
{% endhighlight %}

创建minidlna启动脚本：
{% highlight bash %}
sudo vim /etc/init.d/minidlnad
{% endhighlight %}

{% highlight bash %}
#!/bin/sh
### BEGIN INIT INFO
# Provides:          minidlna
# Required-Start:
# Required-Stop::
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Start minidlna at boot time
# Description:       A minidlna Server
### END INIT INFO

do_start()
{
        /usr/local/sbin/minidlnad
}

do_stop()
{
        killall minidlnad
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

{% highlight bash %}
sudo chown root:root /etc/init.d/minidlnad
sudo chmod 755 /etc/init.d/minidlnad 
sudo chkconfig minidlnad on
{% endhighlight %}


启动`minidlna`服务：

{% highlight bash %}
sudo /etc/init.d/minidlnad start
{% endhighlight %}

经过测试，观看1080P的rmvb影片，随意拖动快进后退，毫无压力，CPU温度保持在40摄氏度左右。

[^1]:参考来源：[百度百科DLNA](http://baike.baidu.com/link?url=0E0rE01UGcOO12KzRuJ2vhieF3MgydzHn1j0iSn1spq5eSMZWQMtEPVMAmbp2QQGPSyMUTrYOrcwlRqwD10Lsa){:target="_blank"}
[^2]:参考来源：[让minidlna支持rmvb格式电影](http://www.shaohu.me/enhance_minidlna/){:target="_blank"}
