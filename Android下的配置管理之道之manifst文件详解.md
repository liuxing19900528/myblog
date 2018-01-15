Android使用repo来管理多个Git项目。它需要一个manifest  XML文件来指示这些git项目的属性。
 
这里对manifest 做个介绍：
repo manifest XML可以包含下面的元素。

1.manifest: 最顶层的XML元素。
```xml
<?xml version="1.0" encoding="UTF-8"?>

```
 

2.remote元素: 设置远程git服务器的属性，包括下面的属性
```xml
<remote fetch=".." name="zsgit" review="gerrit.xxxxx.com"/>
```

    name: 远程git服务器的名字，直接用于git fetch, git remote 等操作
    alias: 远程git服务器的别名，如果指定了，则会覆盖name的设定。在一个manifest中，
    name不能重名，但alias可以重名。
    fetch: 所有projects的git URL 前缀
    review: 指定Gerrit的服务器名，用于repo upload操作。如果没有指定，则repo upload没有效果


3.default元素：设定所有projects的默认属性值，如果在project元素里没有指定一个属性，则使用default元素的属性值。
```xml
   <default remote="zsgit" revision="msm8996_nougat_r01055_20161021"/>
```
    remote: 之前定义的某一个remote元素中name属性值，用于指定使用哪一个远程git服务器。
    revision: git分支的名字，例如master或者refs/heads/master
    sync_j: 在repo sync中默认并行的数目。
    sync_c: 如果设置为true，则只同步指定的分支(revision 属性指定)，而不是所有的ref内容。
    sync_s: 如果设置为true，则会同步git的子项目
 

4.project元素：指定一个需要clone的git仓库。
```xml
<project groups="zs_amss" name="AMSS/adsp_proc" path="AMSS/adsp_proc"  />
```

    name: 唯一的名字标识project，同时也用于生成git仓库的URL。格式如下：
          ${remote_fetch}/${project_name}.git
    path: 可选的路径。指定git clone出来的代码存放在本地的子目录。如果没有指定，则以name作为子目录名。
    remote: 指定之前在某个remote元素中的name。
    revision: 指定需要获取的git提交点，可以是master, refs/heads/master, tag或者SHA-1值。
    groups: 列出project所属的组，以空格或者逗号分隔多个组名。所有的project都自动属于"all"组。每一个project自动属于
    name:'name' 和path:'path'组。例如<project name="monkeys" path="barrel-of"/>，它自动属于default, name:monkeys, and path:barrel-of组。如果一个project属于notdefault组，则，repo sync时不会下载。
    sync_c: 如果设置为true，则只同步指定的分支(revision 属性指定)，而不是所有的ref内容。
    sync_s: 如果设置为true，则会同步git的子项目。
    upstream: 在哪个git分支可以找到一个SHA1。用于同步revision锁定的manifest(-c 模式)。该模式可以避免同步整个ref空间。
    annotation: 可以有多个annotation，格式为name-value pair。在repo forall 命令中这些值会导入到环境变量中。
    remove-project: 从内部的manifest表中删除指定的project。经常用于本地的manifest文件，用户可以替换一个project的定义。
 
 
 
 
5.include

通过name属性可以引入另外一个manifest文件(路径相对与当前的manifest.xml 的路径)

    name ：另一个需要导入的manifest文件名字

EXAMPLE ：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<manifest>

  <remote  name="shift"
           fetch="git://git.mygit.com/" />
  <default revision="kk-shift"
           remote="shift"
           sync-j="1" />

  <project path="packages/shift/VideoPlayer" name="platform/packages/shift/VideoPlayer" />
  <include name="another_manifest.xml" />
</manifest>
```
可以在当前的路径下添加一个another_manifest.xml，这样可以在另一个xml中添加或删除project
 
 
 
 
6.remove-project
从内部的manifest表中删除指定的project。经常用于本地的manifest文件，用户可以替换一个project的定义
 
