一、登入VM  
 马哥的淘宝店:https://shop592330910.taobao.com/


0. VM 简介
   VM 虚拟机是提供给软件人员的linux开发环境， SCM bright.ma 已给大家初装好了环境，目前初装了：jdk、git、repo、gcc、g++、…… 也就是说软件常用的开发、编译的软件已初装好。

1.  如何登入VM
大家可以在本机通过linux远程连接软件访问VM，这里推荐：SecureCRT、Xshell，putty，可在\\10.0.12.12\dailybuild\share\安装程序    目录下面 下载。 当然也可以使用你习惯的其他软件。
目前软件有5台虚拟机，每个虚拟机分12~16个用户，分配如下， 最后一列是虚拟机的owner，
如果大家有什么需求可以向他们反馈，owner评估后会统一反馈给bright.ma统一安装。
如果安装的只是一个简单的脚本，一个可执行的文件，建议放到自己的$HOME/bin 这目录下面就可以了。

Xshell 使用 参考 X shell
putty  使用 参考 PuTTY

提示输入用户名：
VM用户名为 邮箱用户名， 如：ellen ， 默认密码：123456,改vm 用户密码： 

```
passwd username
```

2. 配置git环境
这一步，这个命令别忘了user.name 这是参数的一部分的，别忘了！！！
 
这几个命令会生成一个 git的配置文件， 在 ~/.gitconfig  里。没有这个文件就新建一个！！一般配置下面红色框起来的四个就可以了！！

当然也可以配置一下git命令的别名，例如上面是我的git别名配置。在alias下面那些。git别名还是很有用的，减少键盘的录入字符。

```
[color]
    ui = true
[user]
    email = bright.ma@example.com
    name = <马哥的淘宝店:https://shop592330910.taobao.com/>
[core]
    editor = vim
    autocrlf = false
    whitespace = cr-at-eol
[alias]
    st = status
    sts = status -s
    ci = commit -s
    ca = commit --amend -s

    co = checkout

    cpc = cherry-pick --continue
    cpa = cherry-pick --abort
    rbc = rebase --continue
    rba = rebase --abort

    br   = branch
    bra   = branch -a
    brr   = branch -r
    brv  = branch -v
    brvv = branch -vv

    last = log -1 HEAD
    lo = log --oneline
    ll = log --graph --format=format:'%C(bold blue)%h%C(reset) - %C(red)%<(80,trunc)%s%C(reset) %C(bold black)— %an%C(reset)%C(bold green)%d%C(reset) - %C(bold green)(%ar)%C(reset)' --abbrev-commit --date=relative --show-signature 
    lll = log --pretty='format:%an<gitlog>%ae<gitlog>%H<gitlog>%ci<gitlog>%s'
    lc = log --left-right --cherry-pick --date=short --pretty='%m || %h ||  %<(80,trunc)%s (%<(10,trunc)%an) (%cd)'
    lr = log --show-notes=review

    sh = stash
    shl = stash list

    ri = rebase -i
    ra = rebase --abort
    rc = rebase --continue
[merge]
    tool = vimdiff3

[pack]
	windowMemory = 100m
	packSizeLimit = 100m
	threads = 1
```

3. 生成SSH key
 
  -t type     Specify type of key to create. 代表的key的类型。这里用的rsa的类型。当然还有一种dsa类型的key。都可以的。
生成key的时候一路按回车既可，不要乱输入东西。 
特别注意自己的~/.ssh/目录下面已经生成过 key的话这里就不要重新生成key了。
如果生成ssh过程中，输入密码处直接回车，默认密码为空
成功之后会生成如下两个文件，注意不要随便动这两个文件，不要改名，不要该他们的权限等。注意这两个文件的位置，是在~/.ssh 这个路径下面的。

当然也可以复制之前有的key文件过来，在新的服务器上面 运行如下命令：

```
scp  -r  <旧的服务器上面的用户名>@<旧的服务器的ip>：~/.ssh    ~

```


例如：

```
scp -r  mamh@10.0.12.123:~/.ssh   ~
```

 
 
4. Ubutnu 的登录帐号和 gerrit帐号不一致的需要 更改这个文件~/.ssh/config。
例如
我的 Ubuntu登录帐号是mamh
 
我的gerrit的帐号是bright.ma
 
       这个时候 需要配置一下~/.ssh/config 文件。没有这个文件就新建一个！！！基本上配置这2行就够了，注意顺序不能反。
```
Host gerrit.example.com
User bright.ma
```

     


二、登入Gerrit web
1. 注册并检查Gerrit配置
用浏览器（推荐火狐、chrome浏览器）打开网页http://gerrit.example.com ，右上角 sign in
输入域用户名和密码，看清楚下图是怎么输入域账号的！！！！帐号全部是小写
 
打开如图所示界面（检查username; email 是否正确）：

 2. 提供公钥
到VM 中把刚才生成的public keys 拷到gerrit/settings/ssh public keys中，
 
将输出的内容粘贴到Add SSH Public Key中：
 
点击Add按钮

3.  测试与Gerrit服务器的连接
配置完成后，可通过如下命令测试与Gerrit服务器的连接：

```
ssh -p 29418 gerrit.example.com
```

>如果出现这个错误： Agent admitted failure to sign using the key.  ，解决方法：执行命令：ssh-add

注意出现如下表示链接成功！！！

```

  ****    Welcome to Gerrit Code Review    ****

  Hi 马哥的淘宝店:https://shop592330910.taobao.com/ , you have successfully connected over SSH.

  Unfortunately, interactive shells are disabled.
  To clone a hosted Git repository, use:

  git clone ssh://bright.ma@gerrit.example.com:29418/REPOSITORY_NAME.git

Connection to 10.0.12.123 closed.

```

```
The authenticity of host '10.0.12.181 (10.0.12.181)' can't be established.
ECDSA key fingerprint is 21:59:e9:55:1b:7f:61:d3:64:6e:1a:6e:5d:bf:c4:bc.
Are you sure you want to continue connecting (yes/no)? yes 出现这个的话 要输入yes
 
```


>怎么配置.ssh/config文件在上面，往上面找。

这个gerrit.example.com 测试连通了,配置好了。

4.登录http://gerrit.example.com:8080/#/admin/projects/，可以看到project的列表，就说明已经有权限了。
如果看不到列表，新员工同事找IT看看域控里面有没有你的名字，是不是在对应的组织架构组里面。
 

三、虚拟机下代码
收到CMO回复的权限开通邮件后即可下载代码

>注意 
>文档 里面有个.ssh/config 文件 里面要配置的。访问gerrit的时候，登录ubuntu的用户名和gerrit上面的用户名不一致，所以才要配置这个的。

四、怎么push代码到gerrit上？
>当然还有个py脚本方便大家push patch，这个在每台vm上面都放在了/usr/local/bin 目录下的
> 在某个代码库下面：执行 gerrit.py -b  远程分支名
> 就可以push代码了。
> 例如 push到 master分支：gerrit.py -b master
> 例如 push到 zl2.0_20160430 分支：gerrit.py -b zl2.0_20160430 
> 获得帮助 使用--help选项，这个一般每个命令都会有的。

```
$ gerrit.py -h
Usage: gerrit.py [options] arg1 arg2

Options:
  -h, --help            show this help message and exit

  git push to gerrit options:
    -b BRANCH, --branch=BRANCH
                        what remote branch want to push
    -r REVIEWER, --reviewer=REVIEWER
                        reivew email address, such as:
                        review1email,review2email,review3email
    -d, --drafts        push to gerrit as drafts


```

  

