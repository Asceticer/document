Redhat7中yum安装以及问题解决办法(YUM和wget都无法使用的情况下)
原创ONLY&YOU 最后发布于2018-05-17 10:36:12 阅读数 14177  收藏
展开
简介      
        在linux系统中，由于Redhat自带的yum需注册才可以使用，因此，我们通过安装centos7.0中的yum代替。

        如果直接使用redhat自带的yum（比如输入：yum repolist），可能会出现以下提示：

This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
安装
1、环境准备
       先检查以下我们的linux系统环境，看看是不是Redhat7的版本 ，出现如下图所示的界面     

cat /etc/redhat-release

       检查系统中是否安装了yum以及安装了哪些包。

rpm -qa |grep yum


        删除redhat系统中自带的yum包

 rpm -qa|grep yum|xargs rpm -e --nodeps（不检查依赖，直接删除rpm包）
 rpm -qa |grep yum （查询确认）
2、下载yum安装包
        在系统中，我们使用wget命令，下载有关yum的相应包，执行以下命令，如果在下载过程中出现“404 Not Found”，则到源地址中下载最新的对应版本中的内容。

wget http://mirrors.163.com/centos/7/os/x86_64/Packages/python-iniparse-0.4-9.el7.noarch.rpm
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/python-urlgrabber-3.10-8.el7.noarch.rpm
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/yum-3.4.3-158.el7.centos.noarch.rpm
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/yum-metadata-parser-1.1.4-10.el7.x86_64.rpm
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/yum-plugin-fastestmirror-1.1.31-45.el7.noarch.rpm
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/yum-utils-1.1.31-45.el7.noarch.rpm



        如果未安装wget工具，我们在window系统中，将wget后面的下载链接拷贝到浏览器中，下载文件，然后将文件拷贝到U盘中（确保u盘的格式为fat32,ntfs格式的u盘挂载需要下载NTFS-3G，比较麻烦），通过挂载u盘的方法将文件导入到linux系统中（如果有ftp，也可以使用ftp将文件导入到linux系统）。由于我的Redhat安装在虚拟机中，对于虚拟机中如何挂载u盘，请参照“给VM虚拟机中的centOS Linux系统挂载U盘的方法图文教程”。下载完成后，包含以下文件：



3、安装
rpm -ivh *.rpm
如果出现依赖包的问题或者包冲突的问题，可以添加以下两个参数，进行强迫安装。

--force 即使覆盖属于其它包的文件也强迫安装
--nodeps 如果该RPM包的安装依赖其它包，即使其它包没装，也强迫安装。
rpm -ivh --force --nodeps yum*  最终的强制安装指令


4、配置源
        更改yum库的地址，可以使用网易的开源软件镜像站点下载地址（http://mirrors.163.com/.help/CentOS6-Base-163.repo），也可以使用阿里云的（http://mirrors.aliyun.com/repo/Centos-7.repo）或者其他的站点。这里以163的站点为例：

yum-config-manager --add-repo="http://mirrors.163.com/.help/CentOS6-Base-163.repo"
        然后切换到/etc/yum.repos.d/目录下，修改文件内容，将文件中的“$releasever”改为“7”，“RPM-GPG-KEY-CentOS-6”改为“RPM-GPG-KEY-CentOS-7”，使用一下命令进行全局替换。

:%s/$releasever/7/ge 
:%s/RPM-GPG-KEY-CentOS-6/RPM-GPG-KEY-CentOS-7/ge
更改后的文件内容如下（仅作参照）：

#CentOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the 
# remarked out baseurl= line instead.
#
#

[base]
name=CentOS-7 - Base - 163.com
baseurl=http://mirrors.163.com/centos/7/os/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=7&arch=$basearch&repo=os
gpgcheck=1
gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-7

#released updates 
[updates]
name=CentOS-7 - Updates - 163.com
baseurl=http://mirrors.163.com/centos/7/updates/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=7&arch=$basearch&repo=updates
gpgcheck=1
gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-7

#additional packages that may be useful
[extras]
name=CentOS-7 - Extras - 163.com
baseurl=http://mirrors.163.com/centos/7/extras/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=7&arch=$basearch&repo=extras
gpgcheck=1
gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-7

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-7 - Plus - 163.com
baseurl=http://mirrors.163.com/centos/7/centosplus/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=7&arch=$basearch&repo=centosplus
gpgcheck=1
enabled=0
gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-7

#contrib - packages by Centos Users
[contrib]
name=CentOS-7 - Contrib - 163.com
baseurl=http://mirrors.163.com/centos/7/contrib/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=7&arch=$basearch&repo=contrib
gpgcheck=1
enabled=0
gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-7
5、清除原有缓存，使设置生效
clean all   #清理yum缓存，使设置生效
yum makecache  #将服务器上的软件包信息缓存到本地,以提高搜索安装软件的速度
到此，配置完成。我们可以使用yum下载软件，比如：yum install wget等。

————————————————
版权声明：本文为CSDN博主「ONLY&amp;YOU」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/qq_34182808/java/article/details/80345476