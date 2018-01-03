马哥私房菜 https://shop592330910.taobao.com/

谷歌对android的源代码管理使用的是git。但是在git的基础上，谷歌开发出来了一套新的工具，python写的一套脚本，名字是repo。


Android源代码工程（AOSP）是非常多的git仓库组成的。目前估计有上百个独立的git仓库。
怎么管理这些仓库呢？使用一个清单文件（maniest.xml）来管理每个仓库。
这个xml也需要个仓库存放来管理，就是manifest.git 仓库。
那怎么来下载一套aosp的代码呢？就是使用repo工具，通过manifest.xml 来下载所有的仓库，所有的代码。

到目前为止，我们提到了三种类型的Git仓库，分别是Repo仓库、Manifest仓库和AOSP子项目仓库。Repo仓库通过Manifest仓库可以获得所有AOSP子项目仓库的元信息。有了这些元信息之后，我们就可以通过Repo仓库里面的Python脚本来操作AOSP的子项目。
repo是个工具仓库，这个工具仓库中有个一个文件，名字是repo。这个是一个引导脚本。我们一般会使用这个引导脚本来把整个repo仓库下载下来，整个仓库有了才会有整个功能。一个单一的repo文件里面代码有限的，功能有限的。

manifest是一个管理xml的仓库，每个xml里面都是列的谷歌各个子项目git的相关信息。


----------


#安装Repo

一个repo文件，python写的。几百行代码。
```
可以从清华aosp镜像网站git clone下一个整个仓库。然后把仓库里面的repo文件复制到 /usr/local/bin 目录下面。

https://aosp.tuna.tsinghua.edu.cn/tools/repo
```

```
$ git clone https://aosp.tuna.tsinghua.edu.cn/tools/repo                                                                               
Cloning into 'repo'...
remote: Counting objects: 3967, done.
remote: Compressing objects: 100% (1798/1798), done.
remote: Total 3967 (delta 2555), reused 3392 (delta 2108)
Receiving objects: 100% (3967/3967), 1.97 MiB | 820.00 KiB/s, done.
Resolving deltas: 100% (2555/2555), done.
```
我们可以看到repo仓库里面的所有的文件，其中有一个repo文件。
```
$ tree repo                                                                                                                            
repo
├── color.py
├── command.py
├── COPYING
├── docs
│   └── manifest-format.txt
├── editor.py
├── error.py
├── event_log.py
├── git_command.py
├── git_config.py
├── gitc_utils.py
├── git_refs.py
├── git_ssh
├── hooks
│   ├── commit-msg
│   └── pre-auto-gc
├── main.py
├── manifest_xml.py
├── pager.py
├── platform_utils.py
├── platform_utils_win32.py
├── progress.py
├── project.py
├── pyversion.py
├── README.md
├── repoM
├── subcmds
│   ├── abandon.py
│   ├── branches.py
│   ├── checkout.py
│   ├── cherry_pick.py
│   ├── diffmanifests.py
│   ├── diff.py
│   ├── download.py
│   ├── forall.py
│   ├── gitc_delete.py
│   ├── gitc_init.py
│   ├── grep.py
│   ├── help.py
│   ├── info.py
│   ├── __init__.py
│   ├── init.py
│   ├── list.py
│   ├── manifest.py
│   ├── overview.py
│   ├── prune.py
│   ├── rebase.py
│   ├── selfupdate.py
│   ├── smartsync.py
│   ├── stage.py
│   ├── start.py
│   ├── status.py
│   ├── sync.py
│   ├── upload.py
│   └── version.py
├── SUBMITTING_PATCHES.md
├── tests
│   ├── fixtures
│   │   ├── gitc_config
│   │   └── test.gitconfig
│   ├── test_git_config.py
│   └── test_wrapper.py
├── trace.py
└── wrapper.py

5 directories, 59 files

```
安装完成repo命令就可以执行一下。会输出一些帮助信息的。
```
$ repo                                                                                                                         
usage: repo COMMAND [ARGS]
The most commonly used repo commands are:
  abandon        Permanently abandon a development branch
  branch         View current topic branches
  branches       View current topic branches
  checkout       Checkout a branch for development
  cherry-pick    Cherry-pick a change.
  diff           Show changes between commit and working tree
  diffmanifests  Manifest diff utility
  download       Download and checkout a change
  grep           Print lines matching a pattern
  info           Get info on the manifest branch, current branch or unmerged branches
  init           Initialize repo in the current directory
  list           List projects and their associated directories
  overview       Display overview of unmerged project branches
  prune          Prune (delete) already merged topics
  rebase         Rebase local branches on upstream branch
  smartsync      Update working tree to the latest known good revision
  stage          Stage file(s) for commit
  start          Start a new branch for development
  status         Show the working tree status
  sync           Update working tree to the latest revision
  upload         Upload changes for code review
See 'repo help <command>' for more information on a specific command.
See 'repo help --all' for a complete list of recognized commands.

我们在一个已经repo init过的目录下面执行会显示帮助信息的，每个列出来的都是一个repo的子命令。
和git类似，git clone，其中clone是git的一个子命令，每个子命令又有许多选项参数等。repo 也是类似的。

一般的我们常用的就是 repo init  和  repo sync 2个命令。

```


----------


#repo init

repo脚本安装好了我们就可以下载aosp代码。首先是新建个目录，然后执行repo init命令。也就是对这个空目录进行初始化。
在初始化过程中会把完整的repo仓库下载下来，完整的repo仓库会有更多其他功能，其他子命令，其他选项等。

```
$ repo                                                                                                                                 
error: repo is not installed.  Use "repo init" to install it here.
随便一个目录执行repo命令是会保持的，然后会提示你先 repo init 安装，这个安装就是会安装到当前目录，在当前目录下面的.repo 目录。一个隐藏目录。
```



```
建立工作目录:
mkdir aosp && cd aosp

初始化仓库:
一般的是这样一个简单的命令：
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest
但是呢国内是不行的。这个命令中-u后面跟的就是一个manifest清单仓库。一般叫manifest名。这个仓库名可以是其他的，通常不会改。


repo init -u https://source.codeaurora.org/quic/la/platform/manifest.git \
-b release -m LA.UM.6.6.r1-02700-89xx.0.xml \
--repo-url=https://aosp.tuna.tsinghua.edu.cn/tools/repo \
--repo-branch=stable

```




#repo init的选项
```
$ repo  init  -h

Usage: repo init -u url [options]

Options:
  -h, --help            show this help message and exit

  Logging options:
    -q, --quiet         be quiet

  Manifest options:
    -u URL, --manifest-url=URL
                        manifest repository location
    -b REVISION, --manifest-branch=REVISION
                        manifest branch or revision
    -m NAME.xml, --manifest-name=NAME.xml
                        initial manifest file
    --mirror            create a replica of the remote repositories rather
                        than a client working directory
    --reference=DIR     location of mirror directory
    --depth=DEPTH       create a shallow clone with given depth; see git clone
    --archive           checkout an archive instead of a git repository for
                        each project. See git archive.
    -g GROUP, --groups=GROUP
                        restrict manifest projects to ones with specified
                        group(s) [default|all|G1,G2,G3|G4,-G5,-G6]
    -p PLATFORM, --platform=PLATFORM
                        restrict manifest projects to ones with a specified
                        platform group [auto|all|none|linux|darwin|...]
    --no-clone-bundle   disable use of /clone.bundle on HTTP/HTTPS

  repo Version options:
    --repo-url=URL      repo repository location
    --repo-branch=REVISION
                        repo branch or revision
    --no-repo-verify    do not verify repo source code

  Other options:
    --config-name       Always prompt for name/e-mail

```


我们还拿之前那个高通的下载代码的命令来讲。基本上常用的选项都用上了。
```
repo init -u https://source.codeaurora.org/quic/la/platform/manifest.git \
-b release -m LA.UM.6.6.r1-02700-89xx.0.xml \
--repo-url=https://aosp.tuna.tsinghua.edu.cn/tools/repo \
--repo-branch=stable

```
>-u选项，这个选项和 --manifest-url 这个长选项是一样的，效果相同。在linux中命令有的有长选项，
有的有短选项，有的是符合posix标准的选项的等等。这个选项后面是要跟一个url的，这个url就是
manifest仓库的url地址。这个仓库要有权限下载。


>-b选项，这个是设置manifest仓库的分支的，manifest.xml是放到git中管理，在git中就有可能有不同
的分支的。例如这里高通就使用了一个release分支来存放manifest.xml文件。当然这个分支名称可以随便
取的。


>-m选项，这个是指定初始化的时候使用 哪个xml文件。也就是-u决定manifest仓库路径地址，-b指定
manifest仓库分支，-m指定这个分支下的哪个xml文件。三个选项就决定了一个唯一的manifest.xml文件


注意：有的公司是使用不同的manifest分支来管理整套安卓源码的。有的公司是使用几个manifest分支，
每个分支下面使用不同的xml文件来管理整个安卓源码的。这2中方式各有利弊吧。谷歌是通过不同分支来管理。
高通是通过不同的xml文件来管理整套安卓源码的。这个明显的区别就是有没有使用-m选项。


```
#清华镜像下载aosp的repo init命令：
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest \
-b android-4.0.1_r1 \
--repo-url=https://aosp.tuna.tsinghua.edu.cn/tools/repo \
--repo-branch=stable
```
>--repo-url选项，这个指定repo工具仓库的地址，在repo init的时候就会通过这个地址来把repo仓库
下载下来。

>--repo-branch选项，这个指定repo仓库的分支，上面也说过，是个git仓库就有可能有不同的分支的。

#repo init的其他选项

>-q选项，就是不打印很多log，就是使repo init命令执行时候打印的日志少一些。

>--depth=DEPTH选项，仓库 git clone中的depth，就是下载git仓库的时候历史记录个数可以指定。
就是下载的git历史个数可以少一些，少的话下载速度就快了。不必拉取太多没用的历史提交记录。

>--archive选项，这个选项是在repo sync后不是把代码检出，而是打包，打包压缩。这个就是当你需要
下载代码然后 复制分享给谁的时候 最好打个包整体复制。

>-g,-p选项，-g意思是group，-p意思是platform。在manifest.xml中可以对各个git仓库进行分组，
分平台，然后只下这个指定的goup中的仓库，只下这个platform指定的仓库。可以让开发少下载代码。
这个分组到介绍manifest.xml 的时候在具体介绍。

这个几个选项是无关痛痒的，想用就用上。水平高，了解的话就用上。如果真是小白上面的不用也没关系的，
不用代码一样下载。


#建立本地mirror加快代码下载速度
安卓源码是几百个仓库组成的，代码量都是几十GB的。一个kernel都要有1个GB大小。怎么才能快速的下载
代码呢？那就是本地建个mirror。就是说你下载了好几套代码，每个代码都放在了不同的目录，这几套代码
很多都是重复的，我们把重复的共享使用一个目录，这个就是mirror。这个mirror主要是git裸仓库。

上面介绍repo init选项的时候漏了2个选项一个就是--mirror。通过名称我们就可以猜到这个是建立本地
mirror时候用的。

具体用法就是在repo init命令中加上这个选项--mirror就行了。初次创建需要在一个空目录下面执行命令。
一般的马哥会把本地mirror统一放到/home/mirror 路径下面。
```
# 命令如下：
mkdir /home/mirror && cd /home/mirror &&  rm -rf .repo

repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest \
-b android-4.0.1_r1 \
--repo-url=https://aosp.tuna.tsinghua.edu.cn/tools/repo \
--repo-branch=stable --mirror
```
这样就初始化好本地镜像了。然后执行repo sync命令。网络不好的情况请多执行 repo  sync  几次。



如果不是一个空目录是会报错的
```
fatal: --mirror is only supported when initializing a new workspace.
Either delete the .repo folder in this workspace, or initialize in another location.

```
#使用本地mirror加快代码下载速度
本地mirro创建完成后，以后的repo init命令中就可以使用了。具体用法是 repo init 命令中 加上--reference  /home/mirror
这样就指定使用/home/mirror路径下面的本地镜像仓库了。这样下代码会很快。

>注意：这个--reference 选项和--mirror选项是不能同时使用的。
>注意：/home/mirror下面 不用每次都取执行 repo sync。一个月执行一下都可以。
> 注意：使用了--reference  /home/mirror 下载代码会加快，然后.repo 的大小会很小，几套代码会公用这个mirro的。
```
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest \
-b android-4.0.1_r1 \
--repo-url=https://aosp.tuna.tsinghua.edu.cn/tools/repo \
--repo-branch=stable --reference  /home/mirror
```

#repo sync
 sync           Update working tree to the latest revision
执行完之前的repo init命令之后就可以下载代码了。下载代码的命令是repo sync。也就是同步服务器代码
到本地目录下面。这也是sync的英文单词的意思，同步。

#repo sync的选项
```
$ repo sync --help

Usage: repo sync [<project>...]

Options:
  -h, --help            show this help message and exit
  -f, --force-broken    continue sync even if a project fails to sync
  --force-sync          overwrite an existing git directory if it needs to
                        point to a different object directory. WARNING: this
                        may cause loss of data
  -l, --local-only      only update working tree, don't fetch
  -n, --network-only    fetch only, don't update working tree
  -d, --detach          detach projects back to manifest revision
  -c, --current-branch  fetch only current branch from server
  -q, --quiet           be more quiet
  -j JOBS, --jobs=JOBS  projects to fetch simultaneously (default 4)
  -m NAME.xml, --manifest-name=NAME.xml
                        temporary manifest to use for this sync
  --no-clone-bundle     disable use of /clone.bundle on HTTP/HTTPS
  -u MANIFEST_SERVER_USERNAME, --manifest-server-username=MANIFEST_SERVER_USERNAME
                        username to authenticate with the manifest server
  -p MANIFEST_SERVER_PASSWORD, --manifest-server-password=MANIFEST_SERVER_PASSWORD
                        password to authenticate with the manifest server
  --fetch-submodules    fetch submodules from server
  --no-tags             don't fetch tags
  --optimized-fetch     only fetch projects fixed to sha1 if revision does not
                        exist locally
  --prune               delete refs that no longer exist on the remote
  -s, --smart-sync      smart sync using manifest from the latest known good
                        build
  -t SMART_TAG, --smart-tag=SMART_TAG
                        smart sync using manifest from a known tag

  repo Version options:
    --no-repo-verify    do not verify repo source code
```
这个看着选项也是一大堆，其实基本上都是固定那几个选项的，经常用的就那几个，其他的可以不用关心。


```
repo sync -cdj 4 --no-tags  #一般是这样用的。
```
>-c选项，–current-branch：只sync，只下载当前分支，不加上会下载很多没用的分支的。。默认情况下，sync会同步所有的远程分支，当远程分支比较多的时候，下载的代码量就大。使用该参数，可以缩减下载时间，节省本地磁盘空间。

>-j选项，指定同时开启几个线程来下载代码，这个不要多，就使用4就行。使用的多不见得快。

>-d选项，detach是分离，剥离的意思，一般在git中用git status查看有时候能看到。
HEAD detached at 6beb886d10ec，这个是啥意思呢？这个指头指针剥离自一个revision，
也就是当前分支剥离自一个revision，也就是一个没用名字的分支，也就是当前分支是一个匿名分支。
这个选项在repo sync时候能有什么作用呢？我们接下来详细介绍。

首先我们新建一个目录进行repo init。然后repo sync device/qcom/msm8996，下载一个msm8996的仓库做个测试：

```
$ git br -vvv    
* (no branch) 5e51f0b Merge 6a461864882ef2fec0caab813044b2106bfa199d on remote branch
```
通过 git branch -vvv我们看到初始情况是在一个匿名分支上的。
这个时候往前回退几个提交：

```
$ git br -vv   
* (no branch) 48b4a9a Merge 6f578b199a8bbd8b9c053526a459db0600bf640c on remote branch

```
然后执行 repo sync .，发现有回到之前的那个提交了
```
* (HEAD detached at 5e51f0b) 5e51f0b Merge 6a461864882ef2fec0caab813044b2106bfa199d on remote branch

```

再做个实验，我们用之前的一个提交6b66e01 创建一个新分支test然后切换到test分支上。
```
$ git co 6b66e01 -b test     
Previous HEAD position was 5e51f0b... Merge 6a461864882ef2fec0caab813044b2106bfa199d on remote branch
Switched to a new branch 'test'

$ git st                          
On branch test
nothing to commit, working tree clean

$ git br -vvv          
* test 6b66e01 msm8996: Added Sustained and VR perf mode powerhints in xml
$ 
```
然后执行repo sync .         我们发现分支有切换到之前的那个5e51f0b点了，分支也离开了test分支了。
```
$ repo sync .      
Fetching project platform/vendor/qcom/thulium

device/qcom/msm8996/: leaving test; does not track upstream

$ git br -vvv           
* (HEAD detached at 5e51f0b) 5e51f0b Merge 6a461864882ef2fec0caab813044b2106bfa199d on remote branch
  test                       6b66e01 msm8996: Added Sustained and VR perf mode powerhints in xml

```
再做个实验，如果test分支有改动的状态，也就是git status看到有红色的修改的文件，这个使用执行repo sync .  有可能是会报错的：

```
$ repo sync .     
Fetching project platform/vendor/qcom/thulium
error: Your local changes to the following files would be overwritten by checkout:
	AndroidBoard.mk
Please commit your changes or stash them before you switch branches.
Aborting

device/qcom/msm8996/: leaving test; does not track upstream
error: device/qcom/msm8996/: platform/vendor/qcom/thulium checkout 5e51f0bbfc618dcc9518be616a2843f9c69dbfdf 

$ git st    
On branch test
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   AndroidBoard.mk

no changes added to commit (use "git add" and/or "git commit -a")
$
```

====================================================================================================

接下来我们继续做实验。我们使用repo start dev .        新建个本地分支，并且追踪远程分支

```
$ repo start dev .  

$ git br -vvv    
* dev  5e51f0b [shgit/qcom_o_dev_20171101] Merge 6a461864882ef2fec0caab813044b2106bfa199d on remote branch
  test fe80d2f test
```
这个时候执行repo sync .  不会有什么更新的，因为dev就是在最新的revision点上的。


```
$ git br -vvv        
* dev  07e64c0 [shgit/qcom_o_dev_20171101: ahead 1] add a commit for test      这个本地dev分支，追踪对应的远端分支
  test fe80d2f test                                                              这个是本地test分支


```

====================================================================================================

这个时候我们在dev分支上提交个commit，可以看到ahead 1 表示领先了一个提交，这个时候我们repo sync .
```
$ git br -vvv  
* dev  07e64c0 [shgit/qcom_o_dev_20171101: ahead 1] add a commit for test 这个本地dev分支，追踪对应的远端分支
  test fe80d2f test                                                          这个是本地test分支

```
分支领先的情况，通过repo sync 后我们发现dev分支并没有什么变化。

====================================================================================================

我们再看一种情况，我们分支落后的情况又如何呢？

```
$ git br -vvv   
* dev  48b4a9a [shgit/qcom_o_dev_20171101: behind 80] Merge 6f578b199a8bbd8b9c053526a459db0600bf640c on remote branch  这个本地dev分支，追踪对应的远端分支
  test fe80d2f test             这个是本地test分支
$
```
我们分支落后的情况，这个时候我们repo sync .
```
$ repo sync .                        
remote: Counting objects: 8198, done        
remote: Finding sources: 100% (6/6)           
remote: Total 6 (delta 0), reused 0 (delta 0)        
From ssh://gerrit-sh.blackshark.com:29418/git/android/platform/manifest
   6e376d832..0e5584474  bs_master  -> origin/bs_master

Fetching project platform/vendor/qcom/thulium

device/qcom/msm8996/: manifest switched refs/heads/qcom_o_dev_20171101...qcom_o_dev_20171101
project device/qcom/msm8996/
Updating 48b4a9a..5e51f0b
Fast-forward
 BoardConfig.mk                                     |   6 +-
 WCNSS_qcom_cfg.ini                                 |   6 +-
 android_filesystem_config.h                        |   6 +-
 fstab.qcom                                         |   4 +-
 init.target.rc                                     |  35 ++-
 media_codecs.xml                                   |  23 +-
 msm8996.mk                                         |  41 ++--
 .../frameworks/base/core/res/res/values/config.xml |  59 -----
 recovery_vendor_variant.fstab                      |   5 +-
 system.prop                                        |  10 +-
 vintf.xml                                          | 239 ++++++++++++++++++++-
 11 files changed, 295 insertions(+), 139 deletions(-)

$ git br -vvv   
* dev  5e51f0b [shgit/qcom_o_dev_20171101] Merge 6a461864882ef2fec0caab813044b2106bfa199d on remote branch  这个本地dev分支，追踪对应的远端分支
  test fe80d2f test   这个是本地test分支
$ 
```
分支落后的情况，通过repo sync 后我们发现dev分支发生了同步更新了。


====================================================================================================


接下来我们在新建个本地分支并设置为跟踪分支
```
$ git br -vvv 
  dev                       5e51f0b [shgit/qcom_o_dev_20171101] Merge 6a461864882ef2fec0caab813044b2106bfa199d on remote branch
* sdm660_nougat_20170213    385fc1e [shgit/sdm660_nougat_20170213] Merge dd9dcc7ddabcec42daee9473d8409aae82d6c692 on remote branch
  test                      fe80d2f test
  qcom_msm8953_apk_20170703 880dcbf [shgit/qcom_msm8953_apk_20170703] Merge e3ce047a16037379bfd859e14666a6c7eb2eae05 on remote branch

```
我们接着执行repo sync
```
$ repo sync . 
Fetching project platform/vendor/qcom/thulium
Fetching projects: 100% (1/1), done.  

device/qcom/msm8996/: manifest switched refs/heads/sdm660_nougat_20170213...qcom_o_dev_20171101
device/qcom/msm8996/: discarding 10 commits removed from upstream
$ git br -vvv       
  dev                       5e51f0b [shgit/qcom_o_dev_20171101] Merge 6a461864882ef2fec0caab813044b2106bfa199d on remote branch
* sdm660_nougat_20170213    5e51f0b [shgit/qcom_o_dev_20171101] Merge 6a461864882ef2fec0caab813044b2106bfa199d on remote branch
  test                      fe80d2f test
  qcom_msm8953_apk_20170703 880dcbf [shgit/qcom_msm8953_apk_20170703] Merge e3ce047a16037379bfd859e14666a6c7eb2eae05 on remote branch
$
```
我们发现本地分支发生变化了，原来跟踪的远程分支变掉了。变成qcom_o_dev_20171101了。这个分支是manifest.xml 对应的分支名称。


====================================================================================================

```
$ git br -vvv   
  dev                       5e51f0b [shgit/qcom_o_dev_20171101] Merge 6a461864882ef2fec0caab813044b2106bfa199d on remote branch
  sdm660_nougat_20170213    5e51f0b [shgit/qcom_o_dev_20171101] Merge 6a461864882ef2fec0caab813044b2106bfa199d on remote branch
  test                      fe80d2f test
* qcom_msm8953_apk_20170703 880dcbf [shgit/qcom_msm8953_apk_20170703] Merge e3ce047a16037379bfd859e14666a6c7eb2eae05 on remote branch
$ repo sync .     
Fetching project platform/vendor/qcom/thulium

device/qcom/msm8996/: manifest switched refs/heads/qcom_msm8953_apk_20170703...qcom_o_dev_20171101
device/qcom/msm8996/: discarding 37 commits removed from upstream
$ git br -vvv       
  dev                       5e51f0b [shgit/qcom_o_dev_20171101] Merge 6a461864882ef2fec0caab813044b2106bfa199d on remote branch
  sdm660_nougat_20170213    5e51f0b [shgit/qcom_o_dev_20171101] Merge 6a461864882ef2fec0caab813044b2106bfa199d on remote branch
  test                      fe80d2f test
* qcom_msm8953_apk_20170703 5e51f0b [shgit/qcom_o_dev_20171101] Merge 6a461864882ef2fec0caab813044b2106bfa199d on remote branch
$ 
```
同样的我们有试了另外一个本地分支，发现 原来跟踪的远程分支变掉了。变成qcom_o_dev_20171101了。这个分支是manifest.xml 对应的分支名称。

====================================================================================================


这时候，我们把这个2个本地分支的追踪跟踪改回去。我们发现有领先，也有落后的，然后执行repo sync
```
$ git br -vvv       
  dev                       5e51f0b [shgit/qcom_o_dev_20171101] Merge 6a461864882ef2fec0caab813044b2106bfa199d on remote branch
* sdm660_nougat_20170213    5e51f0b [shgit/sdm660_nougat_20170213: ahead 123, behind 10] Merge 6a461864882ef2fec0caab813044b2106bfa199d on remote branch
  test                      fe80d2f test
  qcom_msm8953_apk_20170703 5e51f0b [shgit/qcom_msm8953_apk_20170703: ahead 123, behind 37] Merge 6a461864882ef2fec0caab813044b2106bfa199d on remote branch
$

$ repo sync .     
Fetching project platform/vendor/qcom/thulium

$ git br -vvv   
  dev                       5e51f0b [shgit/qcom_o_dev_20171101] Merge 6a461864882ef2fec0caab813044b2106bfa199d on remote branch
* sdm660_nougat_20170213    5e51f0b [shgit/sdm660_nougat_20170213: ahead 123, behind 10] Merge 6a461864882ef2fec0caab813044b2106bfa199d on remote branch
  test                      fe80d2f test
  qcom_msm8953_apk_20170703 5e51f0b [shgit/qcom_msm8953_apk_20170703: ahead 123, behind 37] Merge 6a461864882ef2fec0caab813044b2106bfa199d on remote branch
$ 
```
我们发现没有什么变化的，因为此时这2个本地分支指向的都是5e51f0b 这个提交，都是qcom_o_dev_20171101这个分支最新的提交了。


====================================================================================================

我们继续，我们把这个2个本地分支的追踪跟踪改回去,我们在sdm660_o_bringup_20170719 分支上领先一个提交，然后repo sync
```
$ git br -vvv       
  dev                       5e51f0b [shgit/qcom_o_dev_20171101] Merge 6a461864882ef2fec0caab813044b2106bfa199d on remote branch
* sdm660_o_bringup_20170719 c7b1b47 [shgit/sdm660_o_bringup_20170719: ahead 1] test for shgit/sdm660_o_bringup_20170719
  qcom_msm8953_apk_20170703 880dcbf [shgit/qcom_msm8953_apk_20170703] Merge e3ce047a16037379bfd859e14666a6c7eb2eae05 on remote branch

$ repo sync .    
Fetching project platform/vendor/qcom/thulium
Fetching projects: 100% (1/1), done.  

device/qcom/msm8996/: manifest switched refs/heads/sdm660_o_bringup_20170719...qcom_o_dev_20171101
device/qcom/msm8996/: discarding 1 commits removed from upstream
project device/qcom/msm8996/
First, rewinding head to replay your work on top of it...

$ git br -vvvv    
  dev                       5e51f0b [shgit/qcom_o_dev_20171101] Merge 6a461864882ef2fec0caab813044b2106bfa199d on remote branch
* sdm660_o_bringup_20170719 5e51f0b [shgit/qcom_o_dev_20171101] Merge 6a461864882ef2fec0caab813044b2106bfa199d on remote branch
  qcom_msm8953_apk_20170703 880dcbf [shgit/qcom_msm8953_apk_20170703] Merge e3ce047a16037379bfd859e14666a6c7eb2eae05 on remote branch
$
```
发现 原来领先的那个提交也没了 原来跟踪的远程分支变掉了。变成qcom_o_dev_20171101了。这个分支是manifest.xml 对应的分支名称。


====================================================================================================


在这个基础上我们提交个新的，让其领先一个，也就是领先qcom_o_dev_20171101这个分支一个提交
```
$ git br -vvvv   
  dev                       5e51f0b [shgit/qcom_o_dev_20171101] Merge 6a461864882ef2fec0caab813044b2106bfa199d on remote branch
* sdm660_o_bringup_20170719 5e51f0b [shgit/qcom_o_dev_20171101] Merge 6a461864882ef2fec0caab813044b2106bfa199d on remote branch
  qcom_msm8953_apk_20170703 880dcbf [shgit/qcom_msm8953_apk_20170703] Merge e3ce047a16037379bfd859e14666a6c7eb2eae05 on remote branch


$ git br -vvv    
  dev                       5e51f0b [shgit/qcom_o_dev_20171101] Merge 6a461864882ef2fec0caab813044b2106bfa199d on remote branch
* sdm660_o_bringup_20170719 21badc9 [shgit/qcom_o_dev_20171101: ahead 1] test for
  qcom_msm8953_apk_20170703 880dcbf [shgit/qcom_msm8953_apk_20170703] Merge e3ce047a16037379bfd859e14666a6c7eb2eae05 on remote branch

$ repo sync .    
Fetching project platform/vendor/qcom/thulium
Fetching projects: 100% (1/1), done.  

$ git br -vvvv     
  dev                       5e51f0b [shgit/qcom_o_dev_20171101] Merge 6a461864882ef2fec0caab813044b2106bfa199d on remote branch
* sdm660_o_bringup_20170719 21badc9 [shgit/qcom_o_dev_20171101: ahead 1] test for
  qcom_msm8953_apk_20170703 880dcbf [shgit/qcom_msm8953_apk_20170703] Merge e3ce047a16037379bfd859e14666a6c7eb2eae05 on remote branch
 
```
我们发现领先qcom_o_dev_20171101分支的话，执行repo sync是没有什么变化的。
基本上我们可以看到，如果是领先qcom_o_dev_20171101分支的话之后更新是不会有什么改变的



====================================================================================================


切换到qcom_msm8953_apk_20170703 分支，让其落后几个提交，然后执行repo sync
```
$ git br -vvv   
  dev                       5e51f0b [shgit/qcom_o_dev_20171101] Merge 6a461864882ef2fec0caab813044b2106bfa199d on remote branch
  sdm660_o_bringup_20170719 21badc9 [shgit/qcom_o_dev_20171101: ahead 1] test for
* qcom_msm8953_apk_20170703 1c57473 [shgit/qcom_msm8953_apk_20170703: behind 7] Merge 47f59af3394f5966ddeb76c769d4a116d36885dc on remote branch

$ repo sync .    
Fetching project platform/vendor/qcom/thulium

device/qcom/msm8996/: manifest switched refs/heads/qcom_msm8953_apk_20170703...qcom_o_dev_20171101
device/qcom/msm8996/: discarding 30 commits removed from upstream

$ git br -vvvv     
  dev                       5e51f0b [shgit/qcom_o_dev_20171101] Merge 6a461864882ef2fec0caab813044b2106bfa199d on remote branch
  sdm660_o_bringup_20170719 21badc9 [shgit/qcom_o_dev_20171101: ahead 1] test for
* qcom_msm8953_apk_20170703 5e51f0b [shgit/qcom_o_dev_20171101] Merge 6a461864882ef2fec0caab813044b2106bfa199d on remote branch
$
```
我们发现落后分支的话，执行repo sync是会更改追踪的分支的。

====================================================================================================

以上测试都是没加-d选项的

====================================================================================================


4.最后来来试试加上-d后的效果

```
$ repo sync . --no-tags -d
Fetching project kernel/msm-4.9
remote: Counting objects: 31, done        
remote: Finding sources: 100% (16/16)           
remote: Total 16 (delta 0), reused 15 (delta 0)        
From ssh://gerrit.blackshark.com:29418/git/android/kernel/msm-4.9
   6beb886d10ec..b03e189a7d05  qcom_o_dev_20171101 -> shgit/qcom_o_dev_20171101
   
```

repo sync中我们看到有一行6beb886d10ec..b03e189a7d05 这个，这个表示前面的提交 更新到了后面的提交了。

也就是之前的提交revision是6beb886d10ec ，sync后 更新到的revision是 b03e189a7d05   。

6beb886d10ec我们之前看到是  qcom_o_dev_20171101 分支的最后一个提交。

从中也可以看到加了-d， 它更新的是qcom_o_dev_20171101分支，同时给你切换到一个匿名分支上，这就是detach的意思。

5.最后再来看看状态
```
$ git st
HEAD detached at b03e189a7d05
nothing to commit, working tree clean

```

执行完repo sync我们执行git status看到 当前是一个匿名分支了。这个就是manifest.xml中对应的revision值了。
这个就是-d --detach选项剥离选项的作用了。

注意这里需要是一个本地跟踪远端的分支，不是跟踪分支repo sync后会是在一个匿名分支上，剥离的分支。


>--no-tags选项，这个是不下载refs/tags/下面的标签。这个tag一般不需要下载，tag没什么用的。马哥就从来就用到过。建议加上的。尤其是高通的代码有很多tag的，完全没用的东西，下载也很浪费时间。


下面再介绍几个选项的

>-f 选项，同 –force-broken：当有git库sync失败了，不中断整个同步操作，继续同步其他的git库。

>–no-clone-bundle：```
在向服务器发起请求时，为了做到尽快的响应速度，会用到内容分发网络(CDN, Content Delivery Network)。同步操作也会通过CDN与就近的服务器建立连接， 使用HTTP/HTTPS的$URL/clone.bundle来初始化本地的git库，clone.bundle实际上是远程git库的镜像，通过HTTP直接下载，这会更好的利用网络带宽，加快下载速度。 当服务器不能正常响应下载$URL/clone.bundle，但git又能正常工作时，可以通过该参数，配置不下载$URL/clone.bundle，而是直接通过git下载远程git库。```

>  --force-sync    强制sync的意思，也就是当git仓库地址遍历会报个错误，错误中会提示你使用--force-sync，这个时候你就加上这个选项重新同步代码。
>   overwrite an existing git directory if it needs to  point to a different object directory. WARNING: this  may cause loss of data
类似的会报出如下错误：
```
error: Cannot fetch platform/vendor/qcom/proprietary (GitError: --force-sync not enabled; cannot overwrite a local work tree. If you're comfortable with the possibility of losing the work tree's git metadata, use `repo sync --force-sync vendor/qcom/proprietary` to proceed.)
Exception in thread Thread-85:
Traceback (most recent call last):
  File "/usr/lib/python2.7/threading.py", line 810, in __bootstrap_inner
    self.run()
  File "/usr/lib/python2.7/threading.py", line 763, in run
    self.__target(*self.__args, **self.__kwargs)
  File ".repo/repo/subcmds/sync.py", line 270, in _FetchProjectList
    success = self._FetchHelper(opt, project, *args, **kwargs)
  File ".repo/repo/subcmds/sync.py", line 314, in _FetchHelper
    prune=opt.prune)
  File ".repo/repo/project.py", line 1148, in Sync_NetworkHalf
    self._InitGitDir(force_sync=force_sync)
  File ".repo/repo/project.py", line 2177, in _InitGitDir
    raise e
GitError: --force-sync not enabled; cannot overwrite a local work tree. If you're comfortable with the possibility of losing the work tree's git metadata, use `repo sync --force-sync vendor/qcom/proprietary` to proceed.


```

>  -l, --local-only    只更新本地工作空间，不去远端服务器拉取代码。什么意思呢？repo sync在同步代码的时候是先把服务器上的代码更新（拉取，fetch）到.repo目录下面的，然后进行checkout操作。这里的--local-only就是只进行checkout操作，也就是把.repo 目录下面的代码更新到工作空间目录中。
  only update working tree, don't fetch

>  -n, --network-only   只更新，值拉取服务器代码到.repo 目录下。这个就是和上面的--local-only相对应的。 fetch only, don't update working tree。

上面这个2个选项什么时候会用到呢？
>好比你要给某人打包一套android的代码，带上全部git历史记录的。
>你首先想到的就是把整work目录打包发个他，这个操作是可行的。不过就是压缩包很大。
>另外一种就是值打包.repo 目录。在repo sync的时候加上--network-only ，更新完后打包.repo 目录。
>另一方解压完后，repo sync时候加上 --local-only  。这样压缩包会小，打包时间也快。

>  -m NAME.xml, --manifest-name=NAME.xml 这个选项和repo init中的-m类似，这里是更新代码时候临时使用-m指定的这个xml文件取更新代码。 temporary manifest to use for this sync。

>  --prune      删除远端已经不存在了，或者远端分支没权限看到了 的 本地分支。         delete refs that no longer exist on the remote

