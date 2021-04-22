
# 安装Darwin Streaming Server搭建RTSP流媒体服务器 #

最近因为工作需要一个RTSP流媒体链接用于测试，网上找了很久也没有，索性自己搭建一个RTSP流媒体服务器。

最开始是用live555MediaServer，这个部署很简单，但是live555MediaServer不支持3gp格式，无奈只能还其他的RTSP流媒体服务端软件。
后面找到Darwin Streaming Server，这个是苹果公司的开源软件，目前Darwin Streaming Server的最新版本是6.0.3。
不过经过我亲身尝试，要在Linux下源码编译安装最新版的Darwin Streaming Server v6.0.3是很困难的，需要打两个补丁(`dss-6.0.3.patch和dss-hh-20080728-1.patch`)以及替换Install安装脚本文件（无法下载），但就算打了这两个补丁也不一定能安装成功（编译环境存在问题？），所以如果要装Darwin Streaming Server v6.0.3，建议通过RPM包安装，可参考下面这篇文章：

如果确实要源码编译安装最新版的Darwin Streaming Server 6.0.3，请参考以下两篇文章：

http://www.codeproject.com/Articles/41874/Darwin-Streaming-Server-6-0-3-setup-customization（推荐）

http://yaoyj.blog.51cto.com/868664/765899

但注意：上面两篇文章中说的dss-6.0.3.patch和dss-hh-20080728-1.patch需要去别的地方下载，macosforge.org上面已经无法下载了！

不过，我个人建议还是安装Darwin Streaming Server v5.5.5这个版本，安装简单，稳定！

下面开始介绍CentOS 6.5 64位上安装Darwin Streaming Server搭建RTSP流媒体服务器的方法：

一、首先说明下我的系统环境：

CentOS 6.5 64位最小化安装（用的CentOS-6.5-x86_64-minimal.iso），关闭iptables和SELINUX：

    # chkconfig iptables off
    # vi /etc/sysconfig/selinux（把enforcing修改为disabled，然后重启系统）

二、安装后至少需要安装以下5个组件，确保软件安能正常安装及运行：

# yum -y install gcc gcc-c++ perl ld-linux.so.2 libstdc++.so.6 #

三、SSH登录，并切换到root用户；

四、下载Darwin Streaming Server v5.5.5 Linux安装包：

下载，然后通过SFTP或者FTP上传到服务器上。

如果服务器可以上网，也可以直接用wget命令下载：

    # wget

五、解包DarwinStreamingSrvr5.5.5-Linux.tar.gz文件：

    # tar -zxvf DarwinStreamingSrvr5.5.5-Linux.tar.gz

六、cd到DarwinStreamingSrvrlinux-Linux目录：

    # cd DarwinStreamingSrvrlinux-Linux

七、安装Darwin Streaming Server v5.5.5：

    # ./Install

八、打开浏览器，访问Darwin Streaming Server WEB界面（ip换成你服务器的IP）：

初始设置向导：

1、Setup Assistant MP3 Broadcast Password：这里重复输入你的密码，然后点Next下一步；

2、Setup Assistant Secure Administration：不用勾选，直接点Next下一步；

3、Setup Assistant Media Folder:默认为/usr/local/movies，不建议修改，直接点Next下一步；

4、Setup Assistant Streaming on Port 80：不建议改端口，点Finish完成设置向导。


九、测试RTSP流媒体服务是否可用：

我用的VLC media player播放器测试，首先打开VLC media player，然后点左上角“媒体”——“打开网络串流”，然后输入网络URL，例如我想播放/usr/local/movies下的sample_h264_1mbit.mp4，则打开下面这个链接：

    rtsp://192.168.200.77/sample_h264_1mbit.mp4

打开成功后如下图所示：

    sample_h264_1mbit

十、启动及开机自动启动：

1、手动运行：

    # sudo /usr/local/sbin/DarwinStreamingServer
    # sudo /usr/local/sbin/streamingadminserver.pl

备注：

第一个命令为开启DarwinStreamingServer服务，这个服务运行了就可以通过RTSP访问流媒体了；

第二个命令为开启（默认端口1220）。

2、如果要开机自动运行，则把上面两条命令（不用sudo）加到/etc/rc.local文件（exit 0之前）中即可。