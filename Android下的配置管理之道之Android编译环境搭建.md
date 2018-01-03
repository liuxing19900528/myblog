**马哥的淘宝店:https://shop592330910.taobao.com/**


#编译环境搭建

马哥的淘宝店:https://shop592330910.taobao.com/




系统安装
启动服务器电源，首先配置好raid，系统分区使用raid1，编译分区使用raid0，员工的使用raid1，raid50
>一般像马哥的公司 
>专用的编译服务器使用的是dell R430。
>另外一种是 专用编译+员工编译使用的是dell R730。两种的区别就是R730配置的硬盘容量比较多，比较大。
>R730上面会运行一些日常编译任务，同时员工开发也会在上面做些编译。

dell的一般是ctrl + R 进入raid设置
dell的一般是按F11进入u盘启动
使用ubuntu  14.04.4 版本的 server版本的，没有图形界面的。

#添加sudo命令不用输入密码
编辑 sudo cat /etc/sudoers文件  在27行加入这一行  buildfarm ALL=(ALL)NOPASSWD:ALL
马哥的淘宝店:https://shop592330910.taobao.com/


```
$ sudo cat /etc/sudoers -n
     1    #
     2    # This file MUST be edited with the 'visudo' command as root.
     3    #
     4    # Please consider adding local content in /etc/sudoers.d/ instead of
     5    # directly modifying this file.
     6    #
     7    # See the man page for details on how to write a sudoers file.
     8    #
     9    Defaults    env_reset
    10    Defaults    mail_badpass
    11    Defaults    secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
    12    
    13    # Host alias specification
    14    
    15    # User alias specification
    16    
    17    # Cmnd alias specification
    18    
    19    # User privilege specification
    20    root    ALL=(ALL:ALL) ALL
    21    
    22    # Members of the admin group may gain root privileges
    23    %admin ALL=(ALL) ALL
    24    
    25    # Allow members of group sudo to execute any command
    26    %sudo    ALL=(ALL:ALL) ALL
    27    buildfarm ALL=(ALL)NOPASSWD:ALL
    28    # See sudoers(5) for more information on "#include" directives:
    29    
    30    #includedir /etc/sudoers.d
```
 马哥的淘宝店:https://shop592330910.taobao.com/


#配置静态IP
系统安装完成后就设置静态IP
规则(规则自己定，或者IT给出。马哥的公司是自己定义的，IT会把这一个12网段所有IP预留给马哥来使用的。)：
虚拟机IP  10.0.12.181，10.0.12.182，10.0.12.183，10.0.12.184，10.0.12.185。
产品编译专用服务器IP  10.0.12.3X  开始
员工用机架服务器IP  10.0.12.11X 开始
```
$ cat /etc/network/interfaces -n
     1    # This file describes the network interfaces available on your system
     2    # and how to activate them. For more information, see interfaces(5).
     3    
     4    # The loopback network interface
     5    auto lo
     6    iface lo inet loopback
     7    
     8    # The primary network interface
     9    #auto em1
    10   #iface em1 inet dhcp
    11    
    12    auto em1
    13    iface em1 inet static
    14    address 10.0.12.111
    15    netmask 255.255.255.0
    16    up route add default gw 10.0.12.1
    17    dns-search  example.com
    18    dns-nameservers 10.0.13.151 10.0.13.152
```

#配置hostname
修改/etc/hostname文件和/etc/hosts文件
规则：
虚拟机使用VM01，VM02这样的一个规则。
产品编译专用服务器使用bf-01，bf-02 这样的命名规则，bf代表buildfarm的缩写
员工用机架服务器使用RM01，RM02这样的命名规则，rm代表rack machine，就是机架 服务器的意思。
```
$ cat -n /etc/hostname                                                                                                             
     1    RM03
```
 马哥的淘宝店:https://shop592330910.taobao.com/


```
$ cat -n /etc/hosts                                                                                                                  
     1    127.0.0.1    localhost
     2    127.0.1.1    RM03
     3    
     4    # The following lines are desirable for IPv6 capable hosts
     5    ::1     localhost ip6-localhost ip6-loopback
     6    ff02::1 ip6-allnodes
     7    ff02::2 ip6-allrouters
```

#安装jdk
谷歌推荐用open-jdk编译Android
好像是从5.0版本开始使用open-jdk 7 了。之前的版本都是用的oracle的jdk的。
```
sudo apt-get update
sudo apt-get install -y openjdk-7-jdk
```
添加 openjdk：
```
sudo add-apt-repository ppa:openjdk-r/ppa
sudo apt-get update
sudo apt-get install openjdk-7-jdk
sudo apt-get install openjdk-8-*
sudo apt-get install openjdk-7-*
```
添加 oracle jdk：
To install the oracle JDK, use the following command – 
```
sudo add-apt-repository ppa:webupd8team/java
```
配置默认的jdk：
```
sudo update-alternatives --config java
```
 如果不能链接外网，可以到一个能链接外网的ubuntu系统上把安装包复制过来，到如下路径取复制：
`apt的缓存目录： /var/cache/apt/archives/`

#安装其他软件
ubuntu 14.04.4 需要安装的软件
马哥的淘宝店:https://shop592330910.taobao.com/


```
sudo apt-get install -y git gnupg flex bison gperf build-essential \
zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 \
lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z-dev ccache \
libgl1-mesa-dev libxml2-utils xsltproc unzip \
android-tools-adb android-tools-fastboot mkisofs python-pil genisoimage \
libexpat1-dev libxml2-dev  libssl-dev \
vim zsh htop iotop ctags policycoreutils cscope cifs-utils tmux \
python-psutil
```
马哥的淘宝店:https://shop592330910.taobao.com/



ubuntu 16.04 需要安装的软件 (目前16.06的python 版本高一些。暂不推荐使用16.04)
```
sudo apt-get install -y bc
sudo apt install python-six
sudo apt install python-requests
```

安装perl xml解析模块
 
还需要安装一个perl的模块解析xml文件用的
 
切换root，执行
第1个模块：```cpan install XML::Parser  ``` 

可以执行 这个命令来检查是否安装了这个模块```perl -MXML::Parser -e 1 ```
马哥的淘宝店:https://shop592330910.taobao.com/


 
第2个模块：```cpan install XML::LibXML```

马哥的淘宝店:https://shop592330910.taobao.com/


禁止安装的模块
 这个会导致高通amss部分代码编译失败，高通的代码比较垃圾。python里面的一个import有问题导致的。
 
```
 sudo apt-get install python-openpyxl
```

#安装gerrit用的ssh public key
```
ssh-keygen -t rsa
```
从其他服务器上面复制过来， 放在$HOME目录下 ，复制完这个 后面的scp，rsync命令就不用输入密码了
```
scp -r buildfarm@10.0.12.31:~/.ssh ~
```
ssh不用输入密码可以这样做： 把其他服务器上面的public key 内容黏贴到这个文件里面~/.ssh/authorized_keys   
```
$ cat .ssh/authorized_keys                                                                                                         
ssh-rsa xg9vVBHSuiK5tOWh+l8H4BEO4fwuwHbu+L0jyYRf+xDNL3G74/ZsMrqtPSPvFmdBpx6TWJJ+51G38yJJYtPmLVNyQy2ZiDVhDSARjiehcYBdLh3/UODrRKUJLm2bXURpTMt buildfarm@jenkins-slave1
ssh-rsa xKALCkpwMJe0rybdfgY7/+5+3/QmEjX5J9ySuRQX/FoWS7OYthGGfae+j7CQxtaxixASBQmKxrdRPjbeVvatxTGg8W2Oi6W9pFKKN14QxGFGDzgm8GFDCHstizIQX7ebEobtDbtnF mamh@mamh-pc
```

#安装repo，gerrit.py 等脚本
记得还要把repo，gerrit.sh  复制到/usr/local/bin 目录下面，这样其他人ssh登录环境变量path里面才会有
```
scp -r buildfarm@10.0.12.31:~/bin ~/bin
```

#安装qcom编译工具
```
scp -r buildfarm@10.0.12.31:~/.ssh ~，先放在$HOME目录下，复制完移到/home 目录下面,有个20GB多
scp -r buildfarm@10.0.12.31:~/../qcom ~
```

#安装本地repo mirror
从其他服务器上面复制过来，先放在$HOME目录下，复制完移到/home 目录下面，有了这个下载代码5分钟。
```
scp -r buildfarm@10.0.12.31:~/../mirror ~
```
mirror 目录的名称不要换，就要用这个。这里是为了统一。
 马哥的淘宝店:https://shop592330910.taobao.com/


上面的几步可以使用如下的脚本来做
从其他安装好的服务器上面复制脚本setupbf.sh 和 adduser.sh
```
$ cat setupbf.sh  -n
     1    #!/bin/bash
     2    
     3    sudo apt-get update
     4    sudo apt-get install -y openjdk-7-jdk \
     5    git gnupg flex bison gperf build-essential \
     6    zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 \
     7    lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z-dev ccache \
     8    libgl1-mesa-dev libxml2-utils xsltproc unzip android-tools-adb \
     9    android-tools-fastboot mkisofs python-pil genisoimage libexpat1-dev \
    10    vim zsh htop 
    11    
    12    scp -r buildfarm@10.0.12.31:~/.ssh ~
    13    
    14    
    15    scp -r buildfarm@10.0.12.31:~/.vim ~
    16    scp -r buildfarm@10.0.12.31:~/.vimrc ~
    17    scp -r buildfarm@10.0.12.31:~/.zshrc ~
    18    scp -r buildfarm@10.0.12.31:~/.gitconfig ~
    19    scp -r buildfarm@10.0.12.31:~/bin ~/bin
    20    
    21    scp -r buildfarm@10.0.12.31:~/../qcom ~
    22    scp -r buildfarm@10.0.12.31:~/../zeusis-mirror ~
    23    scp -r buildfarm@10.0.12.31:~/../zeusis-mirror-k2 ~
```

#挂载内存到 /tmp 目录

（内存在64GB以上的可以做这一步）
在android编译的时候会用10GB左右的/tmp目录的空间，编译的时候一些临时文件会暂时存放到这个/tmp目录下面。
当然不会是一直是10GB，用完一部分就会删除一部分，如果两个同时编译就要准备10GB 乘以  2 的大小了。三个人乘以3，四个人乘以4，这样来推算
在 /etc/fstab文件中加入如下这一行
```
挂载内存一半做为临时目录
tmpfs      /tmp            tmpfs defaults,noatime,nodiratime,mode=1777,size=50%  0  0
``` 
```
$ cat /etc/fstab -n
     1    # /etc/fstab: static file system information.
     2    #
     3    # Use 'blkid' to print the universally unique identifier for a
     4    # device; this may be used with UUID= as a more robust way to name devices
     5    # that works even if disks are added and removed. See fstab(5).
     6    #
     7    # <file system> <mount point>   <type>  <options>       <dump>  <pass>
     8    # / was on /dev/sda2 during installation
     9    UUID=2fd1b09a-5b92-479f-b053-199882478223 /               ext4    errors=remount-ro 0       1
    10    # swap was on /dev/sda1 during installation
    11    UUID=97b35584-1dee-4fc3-ad9e-97898aa5c831 none            swap    sw              0       0
    12    UUID=2a9d12ef-0475-49db-b7f3-73ba93ab14c9 /work           ext4    defaults    0   0
    13    tmpfs       /tmp            tmpfs   defaults,noatime,nodiratime,mode=1777,size=50%  0  0
```

#挂载dailybuild文件服务器到  /dailybuild 这个目录

注意每日构建的服务器需要多加一行，挂载dailybuild文件服务器到  “/dailybuild” 这个目录，这个是必须挂载的。
```
$ cat /etc/fstab -n
     1    # /etc/fstab: static file system information.
     2    #
     3    # Use 'blkid' to print the universally unique identifier for a
     4    # device; this may be used with UUID= as a more robust way to name devices
     5    # that works even if disks are added and removed. See fstab(5).
     6    #
     7    # <file system> <mount point>   <type>  <options>       <dump>  <pass>
     8    # / was on /dev/sda2 during installation
     9    UUID=bce8bce1-dd31-4a69-8725-a2382a0258f2 /               ext4    errors=remount-ro 0       1
    10    # /home was on /dev/sdc1 during installation
    11    UUID=bfc81376-3e14-461c-8fbc-c24defef5911 /home           ext4    defaults        0       2
    12    # swap was on /dev/sda1 during installation
    13    UUID=1a555d7f-293b-459a-87f1-337ebd4dcc20 none            swap    sw              0       0
    14    tmpfs    /tmp            tmpfs   defaults,noatime,nodiratime,mode=1777,size=50%  0  0
    15    //10.0.12.12/dailybuild     /dailybuild       cifs    username=xxxxxxxxx,password=yyyyyyyy,uid=buildfarm,gid=buildfarm,dir_mode=0755,file_mode=0644 0 0
```

员工用服务器晚上都会拿来做一下dailybuild的构建的，所以呢这里也就需要挂载dailybuild文件服务器拉。
当然员工用服务器挂载dailybuild的方法不要写入这个fstab文件，因为密码是明文的，害怕其他员工看到做误操作。
这里挂载的dailybuild的目录的权限也有讲究。目录的属主是buildfarm这个帐号只允许buildfarm这个帐号执行写操作。
员工的vm挂载Dailybuild方法是在jenkins的slave链接上的时候执行的一个shell脚本。Slave Setups这个插件。
马哥的淘宝店:https://shop592330910.taobao.com/

#挂载硬盘
员工用的服务器：
```
2个1.2TB的sas 硬盘做raid1，分区20GB  swap，其他都是根分区。
6个其他的硬盘做raid50，作为编译目录，挂载到  /work  目录下面。work下面创建buildfarm目录，设置权限是700，属主是builfarm:buildfarm.
```

每日构建的服务器：
```
2个1.2TB的sas 硬盘做raid1，分区20GBswap，其他都是根分区。
2个其他的硬盘做raid0，作为编译目录，挂载到/home/buildfarm/jenkins目录下面。
```

```
ubuntu创建分区
（单个硬盘大于2TB的分区需要使用gpt格式分区表）
Use parted’s mklabel command to set disk label to GPT as shown below.

# parted /dev/sdb
进入到(parted) 交互界面下了
 
(parted) mklabel gpt
(parted) mkpart primary 0GB 17TB  
(parted) q. （退出）
 
 
创建文件系统，ext4格式的文件系统
# mkfs.ext4 /dev/sdb1

挂载到 /work 下面
# mount /dev/sdb1 /work


释放保留空间这里设置为100mb
#   tune2fs -m 0.1 /dev/sdb1
```

#安装samba（员工服务器需要这一步）

首先要安装samba
```
sudo apt-get install samba
```

这里可以从这个地址复制一个smb的配置文件。
```
wget ftp://10.0.10.117/share/smb.conf -O smb.conf
```

也可以从其他配置好的服务器上面scp过来一个，例如
```
scp  10.0.12.181:/etc/samba/smb.conf  ~
```
把这个配置文件复制到/etc/samba/smb.conf
然后 service smbd restart， 这样samba就配置好了。

samba的配置是可以让员工使用ubuntu的帐号密码来访问自己的ubuntu上面的家目录。
smb.conf主要打开了

【homes】子项下面的配置，可以参考下面红色的部分
马哥的淘宝店:https://shop592330910.taobao.com/

```

   193    #======================= Share Definitions =======================
   194    
   195    # Un-comment the following (and tweak the other settings below to suit)
   196    # to enable the default home directory shares. This will share each
   197    # user's home directory as \\server\username
   198    [homes]
   199       comment = Home Directories
   200       browseable = yes
   201    
   202    # By default, the home directories are exported read-only. Change the
   203    # next parameter to 'no' if you want to be able to write to them.
   204       read only = no
   205    
   206    # File creation mask is set to 0700 for security reasons. If you want to
   207    # create files with group=rw permissions, set next parameter to 0775.
   208       create mask = 0755
   209    
   210    # Directory creation mask is set to 0700 for security reasons. If you want to
   211    # create dirs. with group=rw permissions, set next parameter to 0775.
   212       directory mask = 0755
   213    
   214    # By default, \\server\username shares can be connected to by anyone
   215    # with access to the samba server.
   216    # Un-comment the following parameter to make sure that only "username"
   217    # can connect to \\server\username
   218    # This might need tweaking when using external authentication schemes
   219       valid users = %S
   220   
```

完成上面的samba配置后，linux下面的软连接文件通过samba是能看到但是目录是打不开的。
所以这里就要加入下面的这三个配置了，在【global】子项
```
    
    22    #======================= Global Settings =======================
    23    
    24    [global]
    25    
    26    #can access symbol link file in windows with samba
    27    unix extensions = no
    28    follow symlinks = yes
    29    wide links = yes
```

#添加员工账户
注意以下在root用户下操作比较方便
使用这个job添加员工的帐号：http://jenkins.xxx.com/view/AutoTools/job/buildfarm-auto-manageuser/
添加员工的ubuntu帐号，把adduser.sh复制到root的家目录下面，在root用户下执行如下脚本

```
$ cat adduser.sh  -n
     1    #!/bin/bash
     2    #这个脚本需要在root下面执行
     3    
     4    #wget ftp://10.0.10.117/share/smb.conf -O smb.conf
     5    #cp smb.conf /etc/samba/smb.conf
     6    #service smbd restart
     7    
     8    PASS="123456"
     9    BASE_WORK="/work"
    10   
    11    for n in bright.ma
    12    do
    13        LOGIN=$(echo $n|awk -F "." '{print $1}')
    14        echo "cmd =useradd -m -s /bin/bash $LOGIN -c $n="
    15        useradd -m -s /bin/bash $LOGIN -c $n
    16    
    17        echo "cmd =echo $LOGIN:$PASS | chpasswd="
    18        echo $LOGIN:$PASS | chpasswd
    19    
    20        echo "cmd =echo -ne $PASS\n$PASS\n | smbpasswd -a -s $LOGIN="
    21        echo -ne "$PASS\n$PASS\n" | smbpasswd -a -s $LOGIN
    22    
    23        USER_WORK=$BASE_WORK/$LOGIN
    24        LINK_WORK=/home/$LOGIN/work
    25    
    26        mkdir $USER_WORK && chown $LOGIN:$LOGIN $USER_WORK -R
    27        ln -s $USER_WORK $LINK_WORK && chown $LOGIN:$LOGIN $LINK_WORK -h
    28    done
    29    
```
注意在for语句中 分别列出了这个服务器上面要添加的帐号，空格分开，这里使用邮箱帐号@前面的那个字符串，真正的ssh登录ubuntu的帐号会是‘.’点号前面的一部分，整个的作为帐号的一个注释。
名字相同的不要分在一个服务器上面。
 例如
我的邮箱是bright.ma@xxx.com  ， 这里这个脚本for语句后面我写上 bright.ma， 执行脚本之后 会创建ubuntu的帐号是bright，这个也是ssh远程登录过来的帐号，帐号中的注释会是bright.ma。
这里介绍几个概念： “家目录”：这里就是   /home/bright   这样一个路径， 可以用echo  $HOME  查看。一般给员工装的帐号的家目录都是在/home 这个路径下面。
马哥的淘宝店:https://shop592330910.taobao.com/


#修改语言为英文
安装系统的时候选择的安装语言是中文，这里配置为英文好些。因为编译的时候一些出错信息会显示为中文的，这样不太好去解析。


```
$ cat /etc/default/locale -n                                                                                                                 
     1    LANG="en_US.UTF-8"
     2    LANGUAGE="en_US:en"
```

#配置全局git环境


```
cat /etc/gitconfig 
[core]
	editor = vim
[color]
	ui = true
[mergetool "vimdiff3"]
	cmd = vim -f -d -c \"wincmd J\" \"$MERGED\" \"$LOCAL\" \"$BASE\" \"$REMOTE\"
[merge]
	tool = vimdiff3
[alias]
	# basic alias
	st = status
	commit = commit -s
	ci = commit -s
	revert = revert -s
	br = branch
	co = checkout
	df = diff
	dc = diff --cached
	cp = cherry-pick
	unstage = reset
	uncommit = reset --soft HEAD^
	# Compare two branches
	compare = "!f() { RED=$(/bin/echo -e \"\\e[31m\"); GREEN=$(/bin/echo -e \"\\e[32m\"); RST=$(/bin/echo -e \"\\e[0m\"); \
	/bin/echo -e \"${RED}--- $1\\n${GREEN}+++ $2$RST\"; git log --left-right --cherry-pick --oneline --pretty=\"%m %h %C(yellow)%cd%Creset %s %C(bold blue)<%ae>%Creset\" \
	--date=short $1...$2|perl -ple \"s/^< (\\w+) /$RED- \\1 $RST/; s/^> (\\w+) /$GREEN+ \\1 $RST/\"; }; f"
	# git log shortcuts
	lg = log --graph --date=short --pretty=format:'%C(yellow)%h %Cgreen%cd%C(bold yellow)%d%Creset %s %C(bold blue)<%ae>%Creset' --abbrev-commit
	ll = log --graph --date=short --pretty=format:'%C(yellow)%h %Cgreen%cd%C(bold yellow)%d%Creset %s %C(bold blue)<%ae>%Creset' --abbrev-commit --numstat
	logf = log --pretty=fuller
	lp = log -p --pretty=fuller

	ri = rebase --interactive --autosquash
	rc = rebase --continue
	rs = rebase --skip

	this = !git init && git add . && git commit -m \"initial commit\"
	alias = !git config --list | grep 'alias\\.' | sed 's/alias\\.\\([^=]*\\)=\\(.*\\)/\\1\\t=> \\2/' | sort
	ignore = "!f() { prefix=${GIT_PREFIX:-.}; for file in \"$@\"; do echo \"$file\" >> $prefix/.gitignore; done; }; f"
	ignoreunknown = "!if [ \"$GIT_PREFIX\" != \"\" ]; then cd \"$GIT_PREFIX\"; fi; git ls-files --other --exclude-standard | xargs git ignore"
	bractive = "for-each-ref --sort=-committerdate --format='%1B[32m%(committerdate:iso8601) %1B[34m%(committerdate:relative) %1B[0;m%(refname:short)' refs/heads/"
	tarball = "!f() { dirname=${PWD##*/}; git archive HEAD --prefix=$dirname/ | gzip > $dirname.tgz; }; f"
	fixchid = !git show --pretty=\"%s%n%n%b\" -s |sed \"/^Change-Id: /d\" |git ci --amend -F -
	fixup = "!f() { git commit -m \"fixup! $(git log -1 --format=%s $@)\"; }; f"
	squash = "!f() { git commit -m \"squash! $(git log -1 --format=%s $@)\"; }; f"
	showf = show --pretty=fuller
	cleanall = "!f() { git clean -d -f; git reset --hard; }; f"

	type = cat-file -t
	dump = cat-file -p
[url "ssh://gerrit.xxx.com"]
	pushInsteadOf = ssh://gerrit-sh.xxx.com
[url "ssh://gerrit.xxx.com"]
	pushInsteadOf = ssh://gerrit-xi.xxx.com
[url "ssh://gerrit.xxx.com"]
	pushInsteadOf = ssh://gerrit-sz.xxx.com


```

马哥的淘宝店:https://shop592330910.taobao.com/


#设置jack server相关的环境变量

```
cat /etc/profile.d/android.sh 
export ANDROID_JACK_VM_ARGS="-Xmx8g -Dfile.encoding=UTF-8 -XX:+TieredCompilation"
```

#设置arm license相关的环境变量

```
cat /etc/profile.d/armlicense.sh 
export ARMLMD_LICENSE_FILE=8226@armlicense.xxx.com
```

#禁用mlocate定时任务

```
sudo chmod -x /etc/cron.daily/mlocate
```
马哥的淘宝店:https://shop592330910.taobao.com/


 
#禁用更新检查

因为没有网络，所以禁用更新，不然后台会有很多个

├─check-new-relea,172082 /usr/lib/ubuntu-release-upgrader/check-new-release -q
├─check-new-relea,180007 /usr/lib/ubuntu-release-upgrader/check-new-release -q
├─check-new-relea,186866 /usr/lib/ubuntu-release-upgrader/check-new-release -q
├─check-new-relea,191486 /usr/lib/ubuntu-release-upgrader/check-new-release -q
├─check-new-relea,193655 /usr/lib/ubuntu-release-upgrader/check-new-release -q
├─check-new-relea,193686 /usr/lib/ubuntu-release-upgrader/check-new-release -q

```
sudo sed -i 's/Prompt=.*/Prompt=never/' /etc/update-manager/release-upgrades
```
马哥的淘宝店:https://shop592330910.taobao.com/


 


