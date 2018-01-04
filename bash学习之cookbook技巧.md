马哥的淘宝店:https://shop592330910.taobao.com/

bash cookbook 技巧

来自http://www.catonmat.net/blog


Part I: Working With Files  第一部分  文件处理

1.清空文件内容  Empty a file (truncate to 0 size)

```
$ > file
```

这一行命令用到了输出重定向操作符`>`。输出重定向发生时，文件会被打开准备写入。如果此时文件不存在则先创建，存在则将其大小截取为0（truncate to 0）。这里我们并没有重定向写任何内容到文件中，所以文件依然保持为空。
马哥的淘宝店:https://shop592330910.taobao.com/

如果你想替换文件的内容，或者创建一个包含指定内容的文件，可以运行下面的命令：

```
$ echo "some string" > file
```

如果你想往文件里面写入多行内容可以运行下面的命令：
```
cat << EOF > file
some string 1 line
stome string 
.....
EOF

```

2.追加内容到文件

```
$ echo "foo bar baz" >> file     马哥的淘宝店:https://shop592330910.taobao.com/

```

这个命令用到了另外一个输出重定向操作符`>>`，该操作符将内容追加到文件。同样地，如果文件不存在则先创建它。追加的内容之后，紧跟着换行符。如果你不想要追加换行符，在执行echo命令时可以指定-n选项：

```
$ echo -n "foo bar baz" >> file
```

在使用输出重定向的时候，bash可以设置个 `set -C` 来禁止覆盖已经存在的文件

```
$ set -C
$ echo "foo bar baz" > file 
bash: file: cannot overwrite existing file     马哥的淘宝店:https://shop592330910.taobao.com/


```
此时如果确实想覆盖此文件可以使用`>|` 符号。




3.读取文件的首行并赋值给变量

```
$ read -r line < file
```

这一行命令用到了 Bash 的内置命令read，和输入重定向操作符`<`。read命令从标准输入中读取一行，并将内容保存到变量line中。在这里，-r选项保证读入的内容是原始的内容（raw），意味着反斜杠转义的行为不会发生。输入重定向操作符`< file`打开并读取文件file，然后将它作为read命令的标准输入。

记住，read命令会删除包含在IFS变量中出现的所有字符，IFS 的全称是 Internal Field Separator（内部字段分隔符），Bash 根据 IFS 中定义的字符来分隔单词。在这里，read命令读入的行被分隔成多个单词。默认情况下，IFS包含空格，制表符和回车，这意味着开头和结尾的空格和制表符都会被删除。如果你想保留这些符号，可以通过设置IFS为空来完成：

```
$ IFS= read -r line < file       马哥的淘宝店:https://shop592330910.taobao.com/

```

IFS 的变化仅会影响当前的命令，这行命令可以保证读入原始的首行内容到变量line中，同时行首与行尾的空白字符被保留。

另外一种读取文件首行内容，并赋值给变量的方法是:

```
$ line=$(head -1 file)
```

这里用到了命令替换操作符`$(...)`，它运行括号里的命令并且将输出返回。 这个例子中，命令是`head -1 file`，输出的内容是文件的首行。输出然后通过等号赋值给变量`line`。`$(...)`的等价写法是`...`，所以也可以换成下面这样：

```
$ line=`head -1 file`
```

不过，在 Bash 中`$(...)`用法更加推荐，因为它看起来更加整洁，并且容易嵌套使用。

4.依次读入文件每一行

```
$ while read -r line; do
    # do something with $line 马哥的淘宝店:https://shop592330910.taobao.com/

done < file
```

这是一种正确的读取文件内容的做法，read命令放在while循环中。当read命令遇到文件结尾时（EOF），它会返回一个正值，导致循环判断失败终止。

记住，read命令会删除首尾多余的空白字符，所以如果你想保留，请设置 IFS 为空值:

```
$ while IFS= read -r line; do
    # do something with $line  马哥的淘宝店:https://shop592330910.taobao.com/

done < file
```

如果你不想将< file放在最后，可以通过管道将文件的内容输入到 while 循环中：

```
$ cat file | while IFS= read -r line; do
    # do something with $line    马哥的淘宝店:https://shop592330910.taobao.com/

done
```

对这里有个说明：对于循环读取标准输入的操作，很多程序内置有自己的标准输入链接到相同的输入源上面，和read命令一样。它偶尔会产生扭曲的结果。
 另一种方法是从不同的文件描述符读取数据。 
```
exec 3< input_file.txt                # open input_file.txt on fd 3

while read -u 3 -r line ; do
    # do stuff here     马哥的淘宝店:https://shop592330910.taobao.com/

done

exec 3<&-                             # close fd 3
```

对于这个通过管道传给while循环的这个例子需要特别注意一点，这个while循环将会创建一个subshell子shell，之前的一些变量在这个while中是不存在的。

```
i=0
cat foo.txt | while read line; do ((i++)); done
echo "$i"

#will print 0. 马哥的淘宝店:https://shop592330910.taobao.com/
```

```
 i=0;cat foo.txt | (while read line; do ((i++)); done; echo "$i")
```

对于bash4.X可以使用mapfile内置命令来读取文件的一行

```
mapfile ARRAY < file
```
这个命令也有一些选项
-s count - skip the first count lines
-n count - read in at most count lines
-c quanta - set a quanta for the -C option
-C command - run command every quanta lines passing the index of the array about to be assigned
-0 index - start assigning at array[index] instead of 0
-t - strip trailing newline
-u fd - read from file descriptor fd instead of stdin




5.随机读取一行并赋值给变量

```
$ read -r random_line < <(shuf file)      马哥的淘宝店:https://shop592330910.taobao.com/
```

Bash 中并没有提供一种直接的方法来随机读取文件的某一行内容，所以这里需要利用外部程序。在最新的一些 Linux 系统上，GNU Coreutils 包中提供的`shuf`命令可以满足我们的需求。

这一行命令中用到了进程替换（process substitution）操作符`<(...)`。进程替换操作会创建一个匿名的管道文件，并将进程命令的标准输出连接到管道的写一端。然后 Bash 开始执行进程替换中的命令，然后将整个进程替换的表达式替换成匿名管道的文件名。

当 Bash 看到`<(shuf file)`时，它首先打开一个特殊的文件`/dev/fd/n`，这里的n是一个空闲的文件描述符，然后执行`shuf file`命令，将标准输出连接到`/dev/fd/n`，并且替换`<(shuf file)` 为`/dev/fd/n`，因此实际的命令会变成:

```
$ read -r random_line < /dev/fd/n               马哥的淘宝店:https://shop592330910.taobao.com/
```

结果会读取洗牌后的文件的第一行内容。

另外一种做法是，使用 `GNU sort` 命令，它提供的-R选项可以随机排序文件：

```
$ read -r random_line < <(sort -R file)            马哥的淘宝店:https://shop592330910.taobao.com/
```

或者，同前面一样，将结果赋值给变量：

```
$ random_line=$(sort -R file | head -1)         马哥的淘宝店:https://shop592330910.taobao.com/
```

这里，我们首先通过sort -R随机排序文件，然后通过head -1 读取文件的第一行。


6.读取文件首行前三个字段并赋值给变量

```
$ while read -r field1 field2 field3 throwaway; do
    # do something with $field1, $field2, and $field3
done < file
```

如果在read命令中指定多个变量名，它会将读入的内容分隔成多个字段，然后依次赋值给对应的变量，第一个字段赋值给第一个变量，第二个字段赋值给第二个变量，等等，最后将剩余的所有字段赋值给最后一个变量。这也是为什么在上面的例子中，我们加了一个throwaway变量，否则的话，当文件的一行大于三个字段时，第三个变量的内容会包含所有剩余的字段。

有时候，为了书写方便，可以简单地用`_`来替换`throwaway`变量：

```
$ while read -r field1 field2 field3 _; do
    # do something with $field1, $field2, and $field3       马哥的淘宝店:https://shop592330910.taobao.com/
done < file
```

又或者，如果你的文件确实只有三个字段，那可以忽略它：

```
$ while read -r field1 field2 field3; do
    # do something with $field1, $field2, and $field3           马哥的淘宝店:https://shop592330910.taobao.com/
done < file
```

下面是一个例子，假如你想知道一个文件到底包含多少行，多少个单词以及多少个字节。当你执行wc命令时，你会得到3个数字加上文件名，文件名在最后：

```
$ cat file-with-5-lines
x 1
x 2
x 3
x 4
x 5
```
```
$ wc file-with-5-lines
 5 10 20 file-with-5-lines
```

所以，这个文件包含5行，10个单词，以及20个字符。我们接下来，可以通过read命令将这些信息保存到变量中：
```
$ read lines words chars _ < <(wc file-with-5-lines)

$ echo $lines
5
$ echo $words
10
$ echo $chars
20
```
类似地，你也可以使用 here-strings 将字符串分隔并保存到变量中。假设你有一个字符串变量$info，内容为"20 packets in 10 seconds"，然后你想要将从中获取20和10。在不久之前，我是这样来完成的：
```
$ packets=$(echo $info | awk '{ print $1 }')
$ time=$(echo $info | awk '{ print $4 }')
```
然而，得益于read命令的强大和对 Bash 的了解，我们可以这样做：

```
$ read packets _ _ time _ <<< "$info"
```

这里，<<< 就是 here-string 的语法，它允许你直接传递字符串给标准输入。


7.保存文件的大小到变量

```
$ size=$(wc -c < file)
```

这一行命令中用到了第3点中介绍的命令替换操作$(...)，它运行里面的命令并将结果获取回来。在这个例子中，命令是`wc -c < file`，它输出文件的字节数。这个结果最终会赋值给变量size。
这里如果直接执行`wc -c   file` 的话结果会带个文件名称的，这里使用输入重定向就不会带文件名称了。

8.从文件路径中获取文件名

假设，你有一个文件，它的路径为/path/to/file.ext，然后你要从中获取文件名，在这里是file.ext。你要怎么做？ 一个好的方法是通过参数展开（parameter expansion）功能：

```
$ filename=${path##*/}
```

这一行命令使用了参数展开的语法：`${var##pattern}`，它从`$var`字符串开始处开始匹配pattern。如果能够匹配成功，将最长匹配的内容删除后再返回。

在这个例子中，匹配的模式是*/，它尝试匹配/path/to/file.ext的开始部分，正如前面所说，这里是贪婪匹配，所以它能够匹配到最后一个斜杠为止，即匹配的内容是/path/to/。所以当把匹配的内容删除后，返回的内容就是文件名file.ext。
这里的井在键盘的左边，表示从左边开始匹配，2个井号表示尽可能多的匹配。

对此也可以使用basename命令

```
filename=$(basename $path)
```

9.从文件路径中获取目录名

和上面一样类似，这次你要从路径/path/to/file.txt中获取目录名/path/to。你可以继续通过参数展开功能来完成这个任务：

```
$ dirname=${path%/*}
```

这次的用法是`${var%pattern}`，它从`$var`的结尾处匹配`/*`。如果能够成功匹配，将最短匹配的内容删除再返回。

在这个例子中，匹配的模式是/*，它能够匹配/file.ext部分，删除这部分内容后返回的就是目录名称。
通用的百分号在键盘的右边，表示从右边开始匹配，一个百分号表示尽可能少的匹配的。

对此也可以使用dirname命令
```
dirname=$(dirname $path)
```

注意：basename 和 dirname不是bash的内置命令，这个2个是个外部命令。

10.快速拷贝文件

假设你要将文件 `/path/to/file`  拷贝到`/path/to/file_copy`，一般情况下会这么来写：
```
$ cp /path/to/file /path/to/file_copy
```
不过，你可以利用花括号展开（brace expansion）`{...}`功能:

```
$ cp /path/to/file{,_copy}
```

花括号展开可以生成任意字符串的组合，在这个例子中，`/path/to/file{,_copy}`最
终生成`/path/to/file /path/to/file_copy`。所以上面这行命令最终发型成:

```
$ cp /path/to/file /path/to/file_copy
```

或者是文件带扩展名的
```
cp /long/path/to/file{,_bk}.ext
```


类似地，你可以执行下面的命令快速的移动文件：

```
$ mv /path/to/file{,_old}
```

这行命令展开后就变成了：

```
$ mv /path/to/file /path/to/file_old
```

```
function bk {
 if [[ -z $1 ]]; then
    echo "Usage: bk <file>"
    return
 fi

 file=$(basename $1)
 bk=_bk

 while :; do
  case $file in
   *.*)
    newfile=$(echo $file | sed 's/\(.*\)\.\(.*\)/\1'$bk'.\2/')
    ;;
   *)
    newfile=${file}$bk
    ;;
  esac
  if [[ ! -e $newfile ]]; then
   break
  fi
  bk=${bk}_bk
 done

 cp "$1" "$(dirname $1)/$newfile"
 if [[ $? -eq 0 ]]; then
    vim "$1"
 fi
}
```




----------
Part II: Working With Strings  第二部分 字符串处理

1.生成alphabet字母表

```
$ echo {a..z}  
```
这个技巧中又用到了花括号展开。利用它我们可以生成任意的字符串，例如{x..y}，其中 x 和 y 都是单个字符，这个表达式展开后包含 x 与 y 之间的所有字符。注意里面是2个点号。

```
$ echo {a..z}
a b c d e f g h i j k l m n o p q r s t u v w x y z

$ echo {x..y}
x y

$ echo {x..z}
x y z

```

2.生成不包含空格的字母表字符串

```
$ printf "%c" {a..z}
abcdefghijklmnopqrstuvwxyz$ 


```
这个技巧非常棒。如果你在printf命令之后指定一个列表，最终它会循环依次打印每个元素，直到完成为止。printf就像一个循环。
这个技巧中printf的格式是`"%c"`意思就是字符，代表一个字符（character）。后面的参数是从 a 到 z 的字符列表。所以，当printf执行时，它依次输出每个字符直到所有字符全被处理完成为止。

输出的结果最后不包含换行符，因为printf的输出格式是"%c"，其中并没有包含\n。如果你想输出完整的一行，可以简单地在字符列表后面增加一个$'\n'：

```
$ printf "%c" {a..z} $'\n'             马哥的淘宝店:https://shop592330910.taobao.com/
```
另外一个在结尾添加换行的方法是使用echo命令：

```
$ echo $(printf "%c" {a..z})           马哥的淘宝店:https://shop592330910.taobao.com/
```
这个技巧使用了命令替换（substitution），里面执行的命令是`printf "%c" {a..z}`  然后替换命令的输出，最后echo打印这个输出，并且加上换行。


下面的又会输出什么呢？

```
$ printf "%c\n" {a..z}              马哥的淘宝店:https://shop592330910.taobao.com/
```
这个会每个字符后面都加上一个换行的。

如果想要快速地将 printf 的结果保存到变量中，可以使用-v选项：

```
$ printf -v alphabet "%c" {a..z}       马哥的淘宝店:https://shop592330910.taobao.com/
```
这个会把 abcdefghijklmnopqrstuvwxyz i存放到 $alphabet 变量中.

类似的你也可以生成1到100的数字序列

```
$ echo {1..100}

1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 54 55 56 57 58 59 60 61 62 63 64 65 66 67 68 69 70 71 72 73 74 75 76 77 78 79 80 81 82 83 84 85 86 87 88 89 90 91 92 93 94 95 96 97 98 99 100

```
或者，如果你忘记这种方法，可以使用 seq 命令来做这个事情：

```
$ seq 1 100
```

3.输出数字0到9，开头补齐0

```
$ printf "%02d " {0..9}            马哥的淘宝店:https://shop592330910.taobao.com/
```
这里我们又用到了printf的循环输出功能，这一次的输出格式为`"%02d "`，意思是在输出数字的时候，如果不满两位就用0补齐。同时，输出的元素是 0 到 9的列表（括号展开后的结果）。

如果你使用的是bash 4版本的，可以使用花括号展开的方式得到同样的结果：

```
$ echo {00..09}                   马哥的淘宝店:https://shop592330910.taobao.com/
```
旧的bash版本不支持这个功能的。


4.生成 30 个英文单词

```
$ echo {w,t,}h{e{n{,ce{,forth}},re{,in,fore,with{,al}}},ither,at}
```
这里还是用到了花括号展开。这是一个滥用括号展开的例子，看看最终输出的结果是什么：

```
when whence whenceforth where wherein wherefore wherewith wherewithal whither what then thence thenceforth there therein therefore therewith therewithal thither that hen hence henceforth here herein herefore herewith herewithal hither hat

```


你可以通过括号展开生成一组单词或者符号的排列。就是排列组合，例如：

```
$ echo {a,b,c}{1,2,3}           马哥的淘宝店:https://shop592330910.taobao.com/
```
上面的命令会生成以下结果：`a1 a2 a3 b1 b2 b3 c1 c2 c3`。首先，它取出第一个括号中的第一个元素a，然后依次与第二个括号{1,2,3}的所有元素组合，生成a1 a2 a3，依此类推。

5.同一个字符串重复输出 10 次

```
$ echo foo{,,,,,,,,,,}          马哥的淘宝店:https://shop592330910.taobao.com/
```
这个技巧也是使用到了花括号展开。这里foo要和后面的10个空的字符串进行组合，空的当然组合出来
还是本身啦。这样就是重复的得到了10个自身相同的字符串啦。

6.拼接字符串.

```
$ echo "$x$y"                 马哥的淘宝店:https://shop592330910.taobao.com/
```
这小技巧简单地将两个变量的值连接在一起，所以如果x变量的值为foo，而y的值为bar，则结果为foobar。

注意，这里的`"$x$y"`是加了双引号的，如果忘记加了，echo会将`$x$y`当成常规的命令行参数去解析。所以，如果`$x`在开头包含`-`，它就变成一个命令行参数，而不是被 echo 输出的内容。

```
x=-n
y=" foo"
echo $x$y

输出:
 foo
```
相反，正确书写的方式执行后的结果如下所示：
```
x=-n
y=" foo"
echo "$x$y"

输出：
-n foo
```

如果你需要把2个字符串拼接后赋值给一个变量，你可以这样做

```
var=$x$y
```

The quoting only ensures that -n foo is treated as one string and not split on the space. echo will still see the - and test whether it is a valid flag/options. It just happens to be that "n foo" is not a valid flag/option to echo, and so it is treated as a regular string.

The quoting is only known to Bash, which is in charge of parsing the command line into strings and doing variable substitution. Bash (generally, echo just happens to be a built in function in bash) knows nothing about what is regular argument to a command and what is a flag/an option.

Hopefully, the following examples will illustrate the differences:

```
$ ls
bar  bas  foo

$ ls --reverse
foo  bas  bar

$ ls "--reverse"
foo  bas  bar

$ ls --rev erse 
ls: cannot access erse: No such file or directory

$ ls "--rev erse"
ls: unrecognized option '--rev erse'
Try `ls --help' for more information.
```

7.按照指定字符分割字符串
假设`str=foo-bar-baz` 。现在想分割它，并且遍历它。你可以简单的使用IFS和raed来完成：

```
$ IFS=- read -r x y z <<< "$str"
```
这里我们使用read 命令从标准输入读取内容，分割后并依次保存到x y z 三个变量中。其中，`$x` 为 foo, `$y` 为 bar, `$z` 为 baz。我们开始设置IFS是-，IFS用来分割输入的每一行，这里是按照-符号来分割上面的那个字符串，所以最后得到的是每个单一的字段了。

另外要留意的 是here-string操作符<<<，可以很方便地将字符串传递给命令的标准输入。在这个例子中，$str的内容传给 read 命令的标准输入。

你也可以将分割后的几个字段保存到数组类型的变量中：

```
$ IFS=- read -ra parts <<< "foo-bar-baz"         马哥的淘宝店:https://shop592330910.taobao.com/
```
在这里，-a 选项告诉read命令将分割后的元素保存到数组parts中。随后，你可以通过`${parts[0]}, ${parts[1]}和${parts[2]}`  来访问数组的各个元素，或者通过`${parts[@]}`来访问所有元素。

8.逐个字符方式处理字符串

```
$ while IFS= read -rn1 c; do
    # do something with $c       马哥的淘宝店:https://shop592330910.taobao.com/
done <<< "$str"
```
这里我们使用到了read命令的`-n1`  参数，它让read命令依次读入一个字符。类似我们可以使用`-n2`  参数来使read命令依次读取2个字符串。

9.字符串替换

```
$ echo ${str/foo/bar}     马哥的淘宝店:https://shop592330910.taobao.com/
```
这个技巧用到了参数的展开。一个通用的格式是：`${var/find/replace}` 找到$var变量中的find字符串，并将它替换成bar。

10.检查字符串是否匹配某个模式

```
$ if [[ $file = *.zip ]]; then
    # do something      马哥的淘宝店:https://shop592330910.taobao.com/
fi
```

这个小技巧用到了通配符匹配：如果`$file`的值匹配`*.zip`，则执行if语句里的命令。这种语法下的模式是最简单的通配符（glob pattern）匹配，通配符包括`* ? [...]`。其中，`*`可以匹配一个或者多个字符， `?`只能匹配单个字符，`[...]`能够匹配任意出现在中括号里面的字符或者一类字符集。



下面是另外一个例子，用来判断回答是否匹配 Y 或者 y:er is Y or y:

```
$ if [[ $answer = [Yy]* ]]; then
    # do something   马哥的淘宝店:https://shop592330910.taobao.com/
fi
```
注意这个和正则表达式还是不太一样的，虽然很类似，但是不是一回事啊。


11.检查字符串是否匹配某个正则表达式

```
$ if [[ $str =~ [0-9]+\.[0-9]+ ]]; then
    # do something   马哥的淘宝店:https://shop592330910.taobao.com/
fi
```
这一行命令检查`$str`是否能够匹配正则表达式`[0-9]+\.[0-9]+`，即两个数字中间包含一个点号。正则表达式的规范可以通过 man 手册查询: man 7 regex

```
$ str='foo12345bar67890'
$ re='[^0-9]+([0-9]+)[^0-9]+([0-9]+)'

$ [[ $str =~ $re ]] && x=${BASH_REMATCH[1]} y=${BASH_REMATCH[2]}

$ echo "$x/$y       马哥的淘宝店:https://shop592330910.taobao.com/
12345/67890
```


12.计算字符串的长度

```
$ echo ${#str}
```
这里我们又用到了参数展开（也可以叫参数替换）的语法: `${#str}`，它返回$str变量值的长度。
计算数组长度也是类似的。

13.从字符串中提取子串

```
$ str="hello world"     马哥的淘宝店:https://shop592330910.taobao.com/
$ echo ${str:6}
```
这一小技巧通过子串提取操作，从字符串hello world中取到了子串world。子串提取操作的语法格式为`${var:offset:length}`，它的意思是说从变量var中，提取第offset个位置（下标从0开始计算）开始的总共length个数的字符。在我们这个例子中，忽略了length，默认会返回所有剩余的字符。
下面是另外一个例子，返回$str变量中第7、8位置的两个字符：

```
$ echo ${str:7:2}     马哥的淘宝店:https://shop592330910.taobao.com/
```

14.转换成大写

```
$ declare -u var        马哥的淘宝店:https://shop592330910.taobao.com/
$ var="foo bar"
```
Bash 中的内置命令 declare 可以用于声明一个变量，或者设置变量的属性。在这个例子中，通过指定-u选项，使得变量$var在赋值时，就会自动地将内容转换成大写的格式。现在你 echo 它，可以看到所有内容已经变成大写了：

```
$ echo $var         马哥的淘宝店:https://shop592330910.taobao.com/
FOO BAR
```
注意，-u选项也是在 Bash 4 新版本中引入的功能，在低版本下是没有的。类似地，你还可以使用 Bash 4 提供的另外一种参数展开语法`${str^^}`，也可以将字符串转换成太写的格式：

```
$ str="zoo raw"        马哥的淘宝店:https://shop592330910.taobao.com/
$ echo ${str^^}
```

15.转换成小写

```
$ declare -l var       马哥的淘宝店:https://shop592330910.taobao.com/
$ var="FOO BAR"
```

同上面一条类似，-l选项声明变量的小写属性，使得其值转换成小写的格式：

```
$ echo $var        马哥的淘宝店:https://shop592330910.taobao.com/
foo bar
```

同样，只有 Bash 4 以及以上的版本才支持-l选项。另外一种方式是使用参数展开语法:

```
$ str="ZOO RAW"      马哥的淘宝店:https://shop592330910.taobao.com/
$ echo ${str,,}
```


注意，如果是 Bash 4 以下，还是老老实实地用tr命令就可以了。
对于低版本的bash唯一正确安全的做法是：
```
$ echo "$BASH_VERSION" | tr '[[:lower:]]' '[[:upper:]]'
```

补充几个大小写转换的
```
$ str=FOObar
$ echo "${str~} ${str~~}"    马哥的淘宝店:https://shop592330910.taobao.com/
fOObar fooBAR
```

```
$ echo "${str~F} ${str~[a-f]} ${str~~[bBfF]} ${str~~[^a-f]}"
fOObar FOObar fOOBar foobaR
```

----------
Part III: Redirections 第三部分  重定向

在bash中处理重定向是很简单的一件事，如果你意识到它是处理的文件描述符的。这个重定向其实是由bash来解析执行处理的。
当你打开bash（也就是我们的shell，终端，模拟终端等）它会自动的打开3个文件描述符：
标准输入stdin  (文件描述符0 file descriptor 0)，
标准输出stdout (文件描述符1 file descriptor 1)，和 
标准错误stderr (文件描述符2 file descriptor 2)。
当然你可以打开更多个文件描述符（例如 文件描述符3， 文件描述符4，文件描述符5,。。。。），当然你也能关闭这些文件描述符
你可以复制这些文件描述符。你可以读取或者写入这些文件描述符。
文件描述符总是指向某个文件的，除非你关闭了这个文件描述符。通常的bash启动时候为打开3个文件描述符，stdin，stdout和
stderr，他们都会指向你的终端（terminal）。输入会读取终端中的内容，一般也就是你键盘敲入的内容。并且会把标准输出和标准错误送到终端上显示。

假设你的终端设备是/dev/tty0，下面的截图解释了bash启动时候文件描述符的样子：
![这里写图片描述](https://github.com/mageSFC/myblog/blob/master/images/bash%E5%AD%A6%E4%B9%A0%E4%B9%8Bcookbook%E6%8A%80%E5%B7%A7_1.png)

当bash执行一个命令的时候，他会fork一个子进程（fork查看帮助 man 2 fork），子进程会从父进程继承所有的文件描述符，
设置好指定的重定向，最后执行该命令（查看man 3 exec）。

下面我们会用图片的方式来让大家更清楚的认识重定向。

1.重定向一命令的标准输出到一个文件
```
$ command >file         马哥的淘宝店:https://shop592330910.taobao.com/

```
大于号```>```是输出重定向操作符。Bash 首先会打开文件准备写入，如果文件打开成功，则将命令command的 stdout 
指向之前打开的文件。如果打开失败，则不会继续执行命令。command >file的写法和command 1>file的写法是一样的，
1是 stdout 对应的文件描述符。

bash打开文件 ‘file’， 并且把文件描述符1 指向这个文件‘file’。所有要写入到文件描述符1的内容从现在开始都会被
写入到文件‘file’里面了。

下面图片展示了文件描述符的改变：

![这里写图片描述](https://github.com/mageSFC/myblog/blob/master/images/bash%E5%AD%A6%E4%B9%A0%E4%B9%8Bcookbook%E6%8A%80%E5%B7%A7_2.png)

通常情况下 命令 ```command n>file``` 将会把文件描述符n 重定向到文件‘file’的。

举例：
```
$ ls > file_list             马哥的淘宝店:https://shop592330910.taobao.com/
```
重定向命令ls的输出到 文件‘$ ls > file_list’

2.重定向一命令的标准错误到一个文件
```
$ command 2> file            马哥的淘宝店:https://shop592330910.taobao.com/
```
这里bash重定向标准错误到文件‘file’，这个数字2代表的就是标准错误。

下面图片展示了文件描述符的改变。

![这里写图片描述](https://github.com/mageSFC/myblog/blob/master/images/bash%E5%AD%A6%E4%B9%A0%E4%B9%8Bcookbook%E6%8A%80%E5%B7%A7_3.png)

bash打开文件‘file’来写入，得到这个文件描述符，然后把 文件描述符2 指向 这个文件 ‘file ’，所有 此刻开始所有
的错误输出都会写入到这个文件‘file’。

3.同时重定向标准输出和标准错误到一个文件
```
$ command &>file             马哥的淘宝店:https://shop592330910.taobao.com/
```
使用符号 ```&>``` 来同时 重定向标准输出和标准错误  到一个文件。它将命令command的 stdout 和 stderr 
都重定向到文件file中。这个是bash的一个简写形式来快速的 同时 重定向标准输出和标准错误 到一个目标文件。

下面图片展示了文件描述符的改变。

![这里写图片描述](https://github.com/mageSFC/myblog/blob/master/images/bash%E5%AD%A6%E4%B9%A0%E4%B9%8Bcookbook%E6%8A%80%E5%B7%A7_4.png)

你可以看到 标准输出 和标准错误 此刻都指向文件‘file’了。所以呢 此刻所有的写入到stdout和stderr内容都会保存到文件‘file’中了。


除此之外，还有几种方法可以将 stdout 和 stderr 同时重定向到同一个文件中。你可以依次重定向每个输出。
```
$ command >file 2>&1              马哥的淘宝店:https://shop592330910.taobao.com/
```
上面是一种更加常见的方法，首先重定向 stdout 到文件file，然后将 stderr 重定向到和 stdout 同样的文件中。
当 Bash 在命令中遇到多个重定向操作时，它会从左到右依次处理。我们通过图表来依次推导这整个过程。初始时文件描述符表的样子：

![这里写图片描述](https://github.com/mageSFC/myblog/blob/master/images/bash%E5%AD%A6%E4%B9%A0%E4%B9%8Bcookbook%E6%8A%80%E5%B7%A7_5.png)

现在 Bash 处理第一组重定向```>file```，我们之前已经解释过，它将使得 stdout 指向文件file：

![这里写图片描述](https://github.com/mageSFC/myblog/blob/master/images/bash%E5%AD%A6%E4%B9%A0%E4%B9%8Bcookbook%E6%8A%80%E5%B7%A7_6.png)

接下来，Bash 开始处理第二组重定向```2>&1```，它会把 stderr 重定向到 stdout 所指向的文件：

![这里写图片描述](https://github.com/mageSFC/myblog/blob/master/images/bash%E5%AD%A6%E4%B9%A0%E4%B9%8Bcookbook%E6%8A%80%E5%B7%A7_7.png)

这里要注意不要错误的写成：  ```    command >file 2>&1  ``` 这个命令和 ```    $ command 2>&1 >file  ```      是不一样的。千万注意。

重定向的顺序是很重要的，这行命令只会把 stdout 重定向到文件，而 stderr 会继续输出到终端屏幕上。为了理解原因，我们同样来推导依次整个处理过程。
在执行这个命令初始的时候文件描述符看起来像下面这个图片：

![这里写图片描述](https://github.com/mageSFC/myblog/blob/master/images/bash%E5%AD%A6%E4%B9%A0%E4%B9%8Bcookbook%E6%8A%80%E5%B7%A7_8.png)


当 Bash 遇到```2>&1```时，它会把 stderr 指向 stdout 对应的文件（这里是终端）：

![这里写图片描述](https://github.com/mageSFC/myblog/blob/master/images/bash%E5%AD%A6%E4%B9%A0%E4%B9%8Bcookbook%E6%8A%80%E5%B7%A7_9.png)

紧接着，Bash 看到>file，按照之前我们解释的，它会把 stdout 重定向到文件file：

![这里写图片描述](https://github.com/mageSFC/myblog/blob/master/images/bash%E5%AD%A6%E4%B9%A0%E4%B9%8Bcookbook%E6%8A%80%E5%B7%A7_10.png)


从上面的图片中可以看出，stdout 指向了文件file，但是 stderr 依然指向终端。所以，一定要注意重定向的书写顺序。


最后注意在bash中写成这样子：``` $ command &>file  ``` （推荐这个写法）     和写成这样  ``` $ command >&file  ```  是应用的，也就是这个 ``` & ``` 符号可以放到大于号前面也可以放到大于号后面，注意中间是没空格的。


4.丢弃命令的输出
```
 $ command > /dev/null           马哥的淘宝店:https://shop592330910.taobao.com/
```
/dev/null是一个特殊的文件，任何写入到该文件的内容都会被丢弃。所以，我们需要做的就是把 stdout 重定向到文件/dev/null。  文件描述符看起来像下面这个图片：

![这里写图片描述](https://github.com/mageSFC/myblog/blob/master/images/bash%E5%AD%A6%E4%B9%A0%E4%B9%8Bcookbook%E6%8A%80%E5%B7%A7_11.png)


类似的，基于前一条命令，我们可以做到把输出到 stdout 和 stderr 的内容都丢弃：
```
$ command >/dev/null 2>&1          马哥的淘宝店:https://shop592330910.taobao.com/
```
或者简单的写成：
```
$ command &>/dev/null         马哥的淘宝店:https://shop592330910.taobao.com/
```
文件描述符看起来像下面这个图片：

![这里写图片描述](https://github.com/mageSFC/myblog/blob/master/images/bash%E5%AD%A6%E4%B9%A0%E4%B9%8Bcookbook%E6%8A%80%E5%B7%A7_12.png)


5.重定向文件到命令的 stdin
```
$ command <file            马哥的淘宝店:https://shop592330910.taobao.com/
```
Bash 在执行命令之前，打开文件file准备读入。如果打开文件出错，Bash 会直接返错，不会继续执行命令。
相反如果打开成功，Bash 会使用打开的文件的文件描述符作为命令的标准输入。此时，文件描述符表的样子为：

![这里写图片描述](https://github.com/mageSFC/myblog/blob/master/images/bash%E5%AD%A6%E4%B9%A0%E4%B9%8Bcookbook%E6%8A%80%E5%B7%A7_13.png)

下面是一个例子，假如你想把文件的第一行读入到变量中：
```
$ read -r line < file        马哥的淘宝店:https://shop592330910.taobao.com/
```
bash的内置命令read 读取一行，从标准输入中读取一行，通过使用重定向输入符号 小于号 ``` < ``` 来使得read 命令最终是从文件‘file’中读取一行。


6.重定向多行文本到一个命令stdin
```
$ command <<EOL               马哥的淘宝店:https://shop592330910.taobao.com/
your
multi-line
text
goes
here
EOL
```
这里用到了 here document 的语法```<<MARKER ```。当 Bash 遇到该操作符是，它会从标准输入读取每一行，
直到遇到一行以MARKER开头为止。这个例子中，Bash 读取到所有内容并传给command的 stdin。
这里有个例子，假设你想去除一堆 URL 地址中的http://部分，可以用下面的一行命令：
```
$ sed 's|http://||' <<EOL        马哥的淘宝店:https://shop592330910.taobao.com/
http://url1.com
http://url2.com
http://url3.com
EOL
```
这里输入一些url地址列表，然后重定向输入到命令sed，这个sed命令 删除了 字符串 ‘http://’。
这个例子结果如下：
```
url1.com                      马哥的淘宝店:https://shop592330910.taobao.com/
url2.com
url3.com
``` 

下面是关于 Here Documents 的介绍
```
   Here Documents
       This  type  of  redirection  instructs the shell to read input from the current source until a line containing only delimiter (with no trailing blanks) is seen.  All of the lines read up to that
       point are then used as the standard input for a command.

       The format of here-documents is:  马哥的淘宝店:https://shop592330910.taobao.com/

              <<[-]word
                      here-document
              delimiter

       No parameter and variable expansion, command substitution, arithmetic expansion, or pathname expansion is performed on word.  If any characters in word are quoted, the delimiter is the result of
       quote  removal  on word, and the lines in the here-document are not expanded.  If word is unquoted, all lines of the here-document are subjected to parameter expansion, command substitution, and
       arithmetic expansion, the character sequence \<newline> is ignored, and \ must be used to quote the characters \, $, and `.   马哥的淘宝店:https://shop592330910.taobao.com/

       If the redirection operator is <<-, then all leading tab characters are stripped from input lines and the line containing delimiter.  This  allows  here-documents  within  shell  scripts  to  be
       indented in a natural fashion.

```


7.重定向一行文本到命令的stdin

这里也是一个Here Strings语法。
```
$ command <<< "foo bar baz"   马哥的淘宝店:https://shop592330910.taobao.com/
```
举个例子，如果你想快速的 传递一些文件作为命令的参数，像下面这样：
```
$ echo "clipboard contents" | command         马哥的淘宝店:https://shop592330910.taobao.com/
```
这个时候你可以这样来写了：
```
$ command <<< "clipboard contents"       马哥的淘宝店:https://shop592330910.taobao.com/
```

下面是关于 Here Strings 的介绍
```
   Here Strings
       A variant of here documents, the format is:

              <<<word   马哥的淘宝店:https://shop592330910.taobao.com/

       The  word  undergoes brace expansion, tilde expansion, parameter and variable expansion, command substitution, arithmetic expansion, and quote removal.  Pathname expansion and word splitting are
       not performed.  The result is supplied as a single string to the command on its standard input.


```

8.重定向所有命令的 stderr 到文件中
```
$ exec 2>file    马哥的淘宝店:https://shop592330910.taobao.com/
$ command1
$ command2
$ ...
```
这一行命令中使用了 Bash 的内置命令exec。如果你在它之后指定重定向操作，重定向的效果为一直持续到显示改变或者脚本
退出为止。
在这个例子中，2>file处理之后，随后所有命令的 stderr 都会重定向到文件file中。通过这种方法，你可以很方便的把脚
本中所有命令的 stderr 都汇总到一个文件，同时又不用每一个命令之后都指定2>file。

9.打开文件并通过特定文件描述符读
```
$ exec 3<file    马哥的淘宝店:https://shop592330910.taobao.com/
```
上面我们再次用到了exec命令，3<file告诉它以只读方式打开文件 file，并将文件描述符 3 指向打开的文件：
此时，文件描述符表的样子为：

![这里写图片描述](https://github.com/mageSFC/myblog/blob/master/images/bash%E5%AD%A6%E4%B9%A0%E4%B9%8Bcookbook%E6%8A%80%E5%B7%A7_14.png)

现在你可以从这个文件描述符3 来读取数据了：
```
$ read -u 3 line       马哥的淘宝店:https://shop592330910.taobao.com/
```
一些常规的命令，例如 grep，还可以这么用：
```
$ grep "foo" <&3       马哥的淘宝店:https://shop592330910.taobao.com/
```
执行了上面的命令后，grep 命令的 stdin 指向了之前打开的文件，看起来好像将文件描述符 3 复制成了 0。
当你使用完成后，通过下面的方法关闭该文件：
```
$ exec 3>&-           马哥的淘宝店:https://shop592330910.taobao.com/
```
这里文件描述符 3 指向&-，就意味着关闭改文件描述符。


10.通过特定文件描述符打开文件并执行写操作
```
$ exec 4>file         马哥的淘宝店:https://shop592330910.taobao.com/
```
这里我们简单的告诉bash 来 打开一个文件，赋值一个文件描述符 4 ，此时，文件描述符表的样子为：

![这里写图片描述](https://github.com/mageSFC/myblog/blob/master/images/bash%E5%AD%A6%E4%B9%A0%E4%B9%8Bcookbook%E6%8A%80%E5%B7%A7_15.png)


你可以看到，你并不需要按顺序使用文件描述符，可以任意挑选从 0 到 255 之内的所有未被使用的描述符。

接下来，我们可以很方便的通过描述符 4 来写文件：
```
$ echo "foo" >&4      马哥的淘宝店:https://shop592330910.taobao.com/
```
或者关闭描述符：
```
$ exec 4>&-          马哥的淘宝店:https://shop592330910.taobao.com/
```
11.通过特定文件描述符打开文件并读写操作
```
$ exec 3<>file       马哥的淘宝店:https://shop592330910.taobao.com/
```
这里我们用到了菱形操作符（diamond operator) <>，该操作符表示打开的文件既可以用于读也可以用于写。例如：
```
$ echo "foo bar" > file   # write string "foo bar" to file "file".
$ exec 5<> file           # open "file" for rw and assign it fd 5.
$ read -n 3 var <&5       # read the first 3 characters from fd 5. 从文件描述符5中读取
$ echo $var               # 这里会打印出 foo
```
这个会先输出foo ，这个是我们先前写入到文件‘file’中的头3个字符。现在我们在写入一些内容：
```
$ echo -n + >&5           # write "+" at 4th position.  这里会在第四个位置添加符号 ‘+’， 因为之前打开这个文件 已经读取了3个字符指针现在停留在第4个位置上了。
$ exec 5>&-               # close fd 5.
$ cat file                # 这个会输出‘foo+bar’
```
这个会输出‘foo+bar’，我们接着在后写入了+。

12.重定向一组命令的 stdout 到文件中
```
$ (command1; command2) >file    马哥的淘宝店:https://shop592330910.taobao.com/
```
这一行命令使用(commands)语法，commands 会在一个子 shell（sub-shell） 中执行。所以在这里，
command1和command2会在子 shell 中运行，然后 Bash 将子 shell 的 stdout 重定向到文件中。


13.在 Shell 中通过文件执行的命令
打开2个shell，在shell  1中执行：
```
mkfifo fifo             马哥的淘宝店:https://shop592330910.taobao.com/
exec < fifo
```
在shell 2 中执行：
```
exec 3> fifo;           马哥的淘宝店:https://shop592330910.taobao.com/
echo 'echo test' >&3
```
回头你会发现在第一个 shell 中会输出 test，你可以继续不断地往文件 fifo 中输入命令，第一个 shell 中 会一直执行这些命令。

我们来解释下这里的原理。

在第一个 shell 中，我们使用 mkfifo 命令创建了一个命名管道fifo。命名管道（也可以叫做 FIFO）类似之前提到的管道（匿名管道），
除了前者是以文件系统上的文件的方式存在（标识一条特殊的进程通信的内核通道）。命名管道可以被多个进程打开同时读写，当多个进程通
过 FIFO 交换数据时，内核并没有写到文件系统中，而是自己私下里传递了这些数据。所以，FIFO 这种特殊的文件，它在文件系统中是没
有存放数据块的。文件系统只是通过文件名的形式提供标识，以便进程间可以利用这个标识来访问管道。
接下来，我们通过exec < fifo命令，使用 fifo 作为当前 shell 的标准输入。

现在，我们在第二个 shell 中以写的方式打开命名管道，并将文件描述符 3 指向它。接下来，我们只要简单地把 echo test写到文件描
述描述符 3，最终会写到管道 fifo 中。因为第一个 shell 的标准输入连接到管道的读的一段，它会接受到传递过来的内容并执行。

14.通过 Bash 访问 Web 站点
```
$ exec 3<>/dev/tcp/www.google.com/80
$ echo -e "GET / HTTP/1.1\n\n" >&3
$ cat <&3
```
Bash 将/dev/tcp/host/port当作一种特殊的文件，它并不需要实际存在于系统中，这种类型的特殊文件是给 Bash 建立 tcp 连接用的。

在这个例子中，我们首先以读写的方式打开文件描述符 3，并把它指向/dev/tcp/www.google.com/80，后者是一个连接，表示连接到 www.google.com 的 80端口。

接下来，我们往文件描述符 3 写 GET / HTTP/1.1\n\n 。完成之后，我们使用cat命令从同样的地方读取返回内容。

类似的，你也可以通过/dev/udp/host/port 来创建一个 UDP 连接。   使用/dev/tcp/host/port，你甚至可以使用 Bash 写一个端口扫描程序。




15.重定向输出时防止覆盖已有的文件
```
$ set -o noclobber     马哥的淘宝店:https://shop592330910.taobao.com/
```
这行命令将当前 shell 的noclobber选项打开，这个选项的作用是，防止>重定向操作符覆盖已有的文件内容。

这时如果你重定向写入到一个文件，会返回一个错误：
```
$ program > file      马哥的淘宝店:https://shop592330910.taobao.com/
bash: file: cannot overwrite existing file
```
如果你100%确定你要覆盖一个文件，可以使用>|重定向操作符：
```
$ program >| file       马哥的淘宝店:https://shop592330910.taobao.com/
```
上面的命令会正确的执行，因为它覆盖了noclobber选项。




16.重定向标准输出到文件，同时打印到标准输出
```
$ command | tee file    马哥的淘宝店:https://shop592330910.taobao.com/
```
tee是一个很方便的命令，它并不是 Bash 的一部分，但是你会经常用到这个命令。它将接收到的输入，同时打印到标准输出和一个文件中。

下面的图片描述了上面命令执行的过程：

![这里写图片描述](https://github.com/mageSFC/myblog/blob/master/images/bash%E5%AD%A6%E4%B9%A0%E4%B9%8Bcookbook%E6%8A%80%E5%B7%A7_16.png)


17.重定向进程的标准输出到另外一个进程的标准输入
```
$ command1 | command2   马哥的淘宝店:https://shop592330910.taobao.com/
```
这个就是简单的管道的使用，管道是链接了command1的stdout 到command2的 stdin。

![这里写图片描述](https://github.com/mageSFC/myblog/blob/master/images/bash%E5%AD%A6%E4%B9%A0%E4%B9%8Bcookbook%E6%8A%80%E5%B7%A7_17.png)


你可以从中看到 所有   来自命令1的文件描述符1（stdout） 内容 都重定向到了 来自命令2的文件描述符0（stding）

18.重定向进程的标准输出和标准错误到另外一个进程的标准输入
```
$ command1 |& command2      马哥的淘宝店:https://shop592330910.taobao.com/
```
以上用法只在 Bash 4.0 以后的版本才能使用，对于老的版本，比较通用的做法是：
```
$ command1 2>&1 | command2   马哥的淘宝店:https://shop592330910.taobao.com/
```
下面的图片描述了上面命令执行的过程：
 
![这里写图片描述](https://github.com/mageSFC/myblog/blob/master/images/bash%E5%AD%A6%E4%B9%A0%E4%B9%8Bcookbook%E6%8A%80%E5%B7%A7_18.png)

首先是命令1的stderr重定向到stdout， 然后是通过管道 一起 发送给命令2 的stdin。


19.给文件描述符 命名
```
$ exec {filew}>output_file    马哥的淘宝店:https://shop592330910.taobao.com/
```
命名文件描述符是bash 4.1的 的特性。命名文件描述符看起来像变量 {varname}。


20.重定向 在命令行上的顺序
```
$ echo hello >/tmp/example      马哥的淘宝店:https://shop592330910.taobao.com/

$ echo >/tmp/example hello

$ >/tmp/example echo hello

```
这3个写法都是一样的，都是正确的。


21.交换标准输出与标准错误输出
```
$ command 3>&1 1>&2 2>&3        马哥的淘宝店:https://shop592330910.taobao.com/

```
在这里，我们首先让文件描述符3指向 stdout，然后将 stdout（文件描述符1）指向 stderr（文件描述符2）。
最后有把 stderr（文件描述符2）指向文件描述符3，即 stdout。最终，我们交换了 stdout 与 stderr。

下面我们通过图来展示以上过程，初始的时候是这样的：

![这里写图片描述](https://github.com/mageSFC/myblog/blob/master/images/bash%E5%AD%A6%E4%B9%A0%E4%B9%8Bcookbook%E6%8A%80%E5%B7%A7_19.png)


首先，执行了3>&1之后，文件描述符3指向 stdout：

![这里写图片描述](https://github.com/mageSFC/myblog/blob/master/images/bash%E5%AD%A6%E4%B9%A0%E4%B9%8Bcookbook%E6%8A%80%E5%B7%A7_20.png)

接下来，执行1>&2，文件描述符1指向了 stderr：

![这里写图片描述](https://github.com/mageSFC/myblog/blob/master/images/bash%E5%AD%A6%E4%B9%A0%E4%B9%8Bcookbook%E6%8A%80%E5%B7%A7_21.png)

最后，执行2>&3，文件描述符2执向了 stdout：

![这里写图片描述](https://github.com/mageSFC/myblog/blob/master/images/bash%E5%AD%A6%E4%B9%A0%E4%B9%8Bcookbook%E6%8A%80%E5%B7%A7_22.png)


如果你是一个追求完美的人，可以将文件描述符3关闭：
```
$ command 3>&1 1>&2 2>&3 3>&-        马哥的淘宝店:https://shop592330910.taobao.com/
```
最终的文件描述符图会是这样的：

![这里写图片描述](https://github.com/mageSFC/myblog/blob/master/images/bash%E5%AD%A6%E4%B9%A0%E4%B9%8Bcookbook%E6%8A%80%E5%B7%A7_23.png)

最终你可以发现 文件描述符1 和文件描述符2 已经交换了。


22.重定向标准输出和标注错误输出给不同的进程
```
$ command > >(stdout_cmd) 2> >(stderr_cmd)    马哥的淘宝店:https://shop592330910.taobao.com/
```

这一行命令用到了进程替换（Process Substitution）语法。>(...)操作符的执行过程是，运行里面的命令，
同时将命令的标准输入连接到一个命名管道的读段。Bash 随后会用命名管道的实际文件名替换这个操作符。

例如，假设第一个替换操作>(stdout_cmd)返回/dev/fd/60，而后一个返回/dev/fd/61。替换后，最初的命令变成以下形式：
```
$ command > /dev/fd/60 2> /dev/fd/61
```
从上面可以看出，标准输出重定向到了/dev/fd/60，而标准错误输出则重定向到了/dev/fd/61。

当命令执行是输出内容到 stdout，则管道/dev/fd/60后面的进程（stdout_cmd）会从另外一侧读取到数据。
同样的，进程stderr_cmd也能从命令的 stderr 输出中读取。


23.获取管道流中的所有命令执行退出码
假设你用管道流执行多个命令：
```
$ cmd1 | cmd2 | cmd3 | cmd4        马哥的淘宝店:https://shop592330910.taobao.com/
```
然后你想获取所有命令的退出码，但是这里并没有一种简单的做法可以实现，因为 Bash 只会返回最后一个命令的退出码。

Bash 的开发者同样思考了这个问题，他们添加了PIPESTATUS数组，这个数组中存放了管道流中所有命令的退出码。

下面是一个简单的例子：
```
$ echo 'pants are cool' | grep 'moo' | sed 's/o/x/' | awk '{ print $1 }'
$ echo ${PIPESTATUS[@]}       马哥的淘宝店:https://shop592330910.taobao.com/
0 1 0 0
```
这个例子中 grep ‘moo’ 命令失败， 这个数组  PIPESTATUS 中的第二个元素保存的这个失败的状态。




----------


Part IV: Working with history 第四部分 使用命令行历史

1.清除命令行历史
```
$ rm ~/.bash_history     马哥的淘宝店:https://shop592330910.taobao.com/


```
Bash 将历史执行的命令都保存在一个隐藏文件.bash_history中，该文件位于你的家目录下。
为了清除命令行历史，只要把这个文件删除即可。

注意，当你执行完退出后，最后一个rm ~/.bash_history命令依然会被记录下来。
如果你想隐藏清除的操作命令，请看下一条。

2.停止记录当前会话下命令行历史
```
$ unset HISTFILE    马哥的淘宝店:https://shop592330910.taobao.com/
```

环境变量HISTFILE指向命令行执行历史保存的目标文件路径，如果你重置了该变量，Bash 就不会保存历史。

另外一种方法是将它指向/dev/null：
```
$ HISTFILE=/dev/null    马哥的淘宝店:https://shop592330910.taobao.com/
```

3.不要记录当前执行的命令
```
$  command          马哥的淘宝店:https://shop592330910.taobao.com/
```
如果命令以多余的空格开始，就不会被记录到命令行历史中。
注意这个只在 变量 HISTIGNORE被正确的配置情况下才正常工作的。这个变量包含了 冒号 ： 在命令的前面包含冒号，这样
就是的此命令不会被历史记录。

举例，忽略空格的设置：
```
HISTIGNORE="[ ]*"     马哥的淘宝店:https://shop592330910.taobao.com/
``` 
这里有个我的这个变量的设置：
```
HISTIGNORE="&:[ ]*"        马哥的淘宝店:https://shop592330910.taobao.com/
```
这个&表示上个命令的意思，这里就表示忽略上次的命令，也就是忽略重复的命令，也就是重复执行的命令只会记录一次，
多个直接用冒号分割。


这个是忽略某些命令的意思，例如：
```
export HISTIGNORE=”pwd:ls:history”   马哥的淘宝店:https://shop592330910.taobao.com/
```
这个将会忽略命令pwd，ls，history这3个命令，ls -l这个命令不会被忽略。

这里有个关于环境变量HISTIGNORE的介绍：
```
       HISTIGNORE   马哥的淘宝店:https://shop592330910.taobao.com/
              A  colon-separated list of patterns used to decide which command lines
              should be saved on the history list. Each pattern is anchored at the
              beginning of the line and must match the complete line (no implicit `*' is
              appended).  Each pattern is tested against the line after the checks speci‐
              fied  by  HISTCONTROL are applied.  In addition to the normal shell pattern
              matching characters, `&' matches the previous history  line.   `&'  may  be
              escaped  using  a  backslash;  the backslash is removed before attempting a
              match.  The second and subsequent lines of a  multi-line  compound  command
              are  not  tested,  and  are added to the history regardless of the value of
              HISTIGNORE.
```

默认的，以空格开始的命令是会被记录的，我们可以配置```  export HISTCONTROL="ignorespace"  ```

4.更改保存命令行历史的目标文件
```
$ HISTFILE=~/docs/shell_history.txt     马哥的淘宝店:https://shop592330910.taobao.com/
```
这里我们简单的改变环境变量   HISTFILE 的值，让它指向一个新的文件~/docs/shell_history.txt，这样之后
所有的命令历史就会保存到这个新的文件里了。

5.命令行历史记录中增加时间戳 
```
$ HISTTIMEFORMAT="%Y-%m-%d %H:%M:%S"   马哥的淘宝店:https://shop592330910.taobao.com/
```
如果你给环境变量HISTTIMEFORMAT设置了正确的时间日期格式（可以参考 man 3 strftime），这样bash就会包时间日期记录到命令历史中。
同时你通过命令history查看的时候也会看到时间日期。

6.查看命令历史
```
history                马哥的淘宝店:https://shop592330910.taobao.com/
```
命令history将会显示命令行历史，每一行都会但是一个编号，如果你设置了环境变量HISTTIMEFORMAT，这样同时也会显示时间。

7.显示最后50条命令行历史
```
history 50            马哥的淘宝店:https://shop592330910.taobao.com/
```
命令history后面跟上1个数字，例如50，表示的是显示最后多少条历史记录，这里是显示最后50条命令行历史记录。

8.显示命令历史中执行次数最多的前10条命令
```
$ history |                 马哥的淘宝店:https://shop592330910.taobao.com/
    sed 's/^ \+//;s/  / /' |
    cut -d' ' -f2- |
    awk '{ count[$0]++ } END { for (i in count) print count[i], i }' |
    sort -rn |
    head -10
```
这条技巧结合使用了命令sed，cut，awk，sort，head等，这是一个完美组合，我们来依次看看这个是怎么工作的。
首先是history命令，我们看history命令的输出：
```
$ history                马哥的淘宝店:https://shop592330910.taobao.com/
    1  rm .bash_history 
    2  dmesg
    3  su -
    4  man cryptsetup
    5  dmesg
```
首先我们通过管道 使用sed命令 删除 开头的空格，然后替换 行号数字 后面 2个空格 为 1个空格：
```
$ history | sed 's/^ \+//;s/  / /'
1 rm .bash_history 
2 dmesg
3 su -
4 man cryptsetup
5 dmesg
```
之后是使用cut命令删除第一列的数字（这一列是history命令显示出来的）
```
$ history |                 马哥的淘宝店:https://shop592330910.taobao.com/
    sed 's/^ \+//;s/  / /' |
    cut -d' ' -f2-

rm .bash_history 
dmesg
su -
man cryptsetup
dmesg
```
再之后是使用awk命令记录统计每个出现的命令次数：
```
$ history |                 马哥的淘宝店:https://shop592330910.taobao.com/
    sed 's/^ \+//;s/  / /' |
    cut -d' ' -f2- |
    awk '{ count[$0]++ } END { for (i in count) print count[i], i }'

1 rm .bash_history 
2 dmesg
1 su -
1 man cryptsetup
```
然后我们使用sort命令进行按数字，逆序的排序操作：
```
$ history |                 马哥的淘宝店:https://shop592330910.taobao.com/
    sed 's/^ \+//;s/  / /' |
    cut -d' ' -f2- |
    awk '{ count[$0]++ } END { for (i in count) print count[i], i }' |
    sort -rn

2 dmesg
1 rm .bash_history 
1 su -
1 man cryptsetup
```
最后我们取结果中的前10行，使用head命令
```
$ history |                 马哥的淘宝店:https://shop592330910.taobao.com/
    sed 's/^ \+//;s/  / /' |
    cut -d' ' -f2- |
    awk '{ count[$0]++ } END { for (i in count) print count[i], i }' |
    sort -rn |
    head -10
    
    
```
下面这个是我的系统上的统计结果
```
2172 ls          马哥的淘宝店:https://shop592330910.taobao.com/
1610 gs         这里的gs是 git status的别名
252 cd ..
215 gp          这个的gp是 git push的别名
213 ls -las
197 cd projects
155 gpu         这里的gpu是git pull的别名
151 cd
119 gl           这里的gl是git log的别名
119 cd tests/
```
不过我觉得应该改进一下：
那个cut命令就不要了，这里只统计命令的名字，命令选项不统计，大部分情况下都是命令一样，选项是不一样的。
```
 history |                  马哥的淘宝店:https://shop592330910.taobao.com/
    sed 's/^ \+//;s/  / /' |
    awk '{print $2}' |
    awk '{ count[$0]++ } END { for (i in count) print count[i], i }' |
    sort -rn |
    head -10
    
5 git
4 history
3 man
2 echo
1 read
1 help
```

下面是一个perl版本的
```
history | perl -lne 's/\d+//;s/^\s+//;s/\s\s/ /; $count{$_}++; END { print "$count{$_} $_" for (sort { $count{$a} <= $count{$b} } keys %count )[0..10] }'
```


```
declare -A aA; while read -r _ _ _ cmd; do ((aA["${cmd}"]++)); done < <(history)

for cmd in "${!aA[@]}"; do if (( ${aA[$cmd]} > 1 )); then printf -- '%3d %-.50s\n' ${aA[$cmd]} "${cmd}"; fi; done | sort -rn


```

8.快速执行先前的一个命令
```
$ !!
```
使用2个感叹号（感叹号也叫bang，英文bang），第一个感叹号表示开始历史命令替换，而第二个感叹号表示上一次执行的命令。例如：
```
$ echo foo                马哥的淘宝店:https://shop592330910.taobao.com/
foo
$ !!
foo
```
这里 echo foo 命令被重复执行了
这个很有用，当你输入一个命令执行后，发现忘记使用sudo了，这个时候就就可以这样操作：
```
$ rm /var/log/something
rm: cannot remove `/var/log/something': Permission denied
$
$ sudo !!   # executes `sudo rm /var/log/something`
```

还有一个快捷键的方式执行某个历史命令：
```
CTRL+R                      马哥的淘宝店:https://shop592330910.taobao.com/
```


10.快速执行某个字符串开始的一个历史命令
```
$ !foo
```
上一个命令中，第一个感叹号表示开始历史命令替换，后面的内容表示最近一次执行的以foo开头的命令。例如：
```
$ echo foo                             马哥的淘宝店:https://shop592330910.taobao.com/
foo
$ ls /                                马哥的淘宝店:https://shop592330910.taobao.com/
/bin /boot /home /dev /proc /root /tmp
$ awk -F: '{print $2}' /etc/passwd
...
$ !ls                                  马哥的淘宝店:https://shop592330910.taobao.com/
/bin /boot /home /dev /proc /root /tmp
```
这里我们执行了命令echo，ls，awk，最后我们使用 ！ls来快速的执行历史命令 ls / 

11.使用文本编辑器打开上一次执行的命令
```
$ fc                       马哥的淘宝店:https://shop592330910.taobao.com/
```
当fc命令执行后，会用文本编辑器打开上一个命令。当你想要编辑一个很长并且复杂的命令式，这个功能会帮你省下不少功夫。

例如，你输入了下面一行错误的命令:
```
$ for wav in wav/*; do mp3=$(sed 's/\.wav/\.mp3/' <<< "$wav"); ffmpeg -i "$wav" "$m3p"; done
```
当你输完命令后，因为内容过长，你找不出错误的地方。这中情况下，你可以使用fc命令加载该命令到文本编辑器中，然后错误的地方（最后的 mp3 单词拼错）就一目了然了。

当然还有个技巧：
```
!f:s/m3p/mp3       马哥的淘宝店:https://shop592330910.taobao.com/
```
这个直接利用感叹 加是 f 然后是替换拼写错误的m3p为mp3

后面的替换也可以写为：```  ^m3p^mp3  ```


----------

Part V: Navigating around in emacs mode 第五部分 行编辑模式
 
在这一部分，我会教你如何快速在 Bash 命令行中使用 Emacs 风格的键盘导航快捷键。

0.行编辑模式介绍
Bash 使用 GNU readline 库来提供行编辑特性。readline 库同时支持 Emacs 风格和 Vi 风格的快捷键绑定，
也支持用户去做自定义绑定。默认情况下，readline 会使用 Emacs 风格的键绑定，不过你可以很方便的切换到 Vi
 风格，或者自定义设置。
执行set -o emacs命令切换到 Emacs 风格，set -o vi则会切换到 Vi 风格。
除此之外，你仍可以通过~/.inputrc或者bind命令来自定义快捷键绑定。例如，bind '"\C-f": "ls\n"'将CTRL+F
绑定为执行ls命令。你可以通过查阅 Bash 手册中的 readline 一节来更多地了解 readline 的快捷键绑定语法。



1.移动光标到行首

```
ctrl  +  a     马哥的淘宝店:https://shop592330910.taobao.com/
```

2.移动光标到行尾
```
ctrl + e       马哥的淘宝店:https://shop592330910.taobao.com/
```

3.光标往后（向左）移动一个单词
```
ESC + f 或者 ALT + f       马哥的淘宝店:https://shop592330910.taobao.com/
```
4.光标往前（向右）移动一个单词
```
ESC + b 或者 ALT + b      马哥的淘宝店:https://shop592330910.taobao.com/
```

5.删除上一个单词
```
CTRL + w                 马哥的淘宝店:https://shop592330910.taobao.com/
```
删除一个单词也被称为"killing a word"，每个被删除的单词都被保存在缓存中，可以按下CTRL + y将其粘贴回来，
这个操作被称为"yanking"。

6.粘贴上一次被删除的内容
```
CTRL + y              马哥的淘宝店:https://shop592330910.taobao.com/
```

7.光标往后（向左）移动一个字符
```
CTRL + b               马哥的淘宝店:https://shop592330910.taobao.com/

```
8.光标往前（向右）移动一个字符
```
CTRL + f               马哥的淘宝店:https://shop592330910.taobao.com/
```

9.删除光标前的字符
```
CTRL + u              马哥的淘宝店:https://shop592330910.taobao.com/
```
删除光标前的字符，删除的内容被保存到缓存中，同样可以用CTRL + y粘贴回来。        

10.反向历史搜索        
```
CTRL + r              马哥的淘宝店:https://shop592330910.taobao.com/
```
这可能是 Bash 中最常用的快捷键，当你按下CTRL + r时，会开始反向搜索命令行执行历史。
你只要输入之前执行的命令中的少许字符就可以很快地从历史记录中找到该命令。

11.正向历史搜索
```
CTRL + s               马哥的淘宝店:https://shop592330910.taobao.com/
```
如果你按下CTRL + s，终端会停止屏幕刷新，因为默认情况下，你的终端将它解释成停止输出流的信号。
当我是新手时，这种情况快把我逼疯了。每次我不小心按下CTRL + s后，屏幕就冻结了，然后我就不知道发生了什么。
之后，我才学会用CTRL + q键来恢复终端。

正确的方式应该是通过stty命令来更改终端对于  CTRL + s  按下后采取的行为：
```
$ stty stop 'undef'     马哥的淘宝店:https://shop592330910.taobao.com/
```
这样会取消默认的停止信号的快捷键绑定，然后你可以开始使用 Bash 的CTRL + s功能。
CTRL + s在 Bash 中的作用和CTRL + r相反，是执行正向历史搜索。

12.交换相邻两个字符的位置
```
CTRL + t               马哥的淘宝店:https://shop592330910.taobao.com/

```

13.交换相邻两个单词的位置
```
ESC + t 或者 ALT + t    马哥的淘宝店:https://shop592330910.taobao.com/
```

14.将光标开始到单词结尾的字符转换成大写
```
ESC + u 或者 ALT + u    马哥的淘宝店:https://shop592330910.taobao.com/
```




15.将光标开始到单词结尾的字符转换成小写
```
ESC + l 或者 ALT + l     马哥的淘宝店:https://shop592330910.taobao.com/
```
16.单词首字符大写
```
ESC + c 或者 ALT + c     马哥的淘宝店:https://shop592330910.taobao.com/
```
在单词的首字符下按下，可以将首字符转换成大写的形式。


17.输入特殊字符
```
CTRL + v                  马哥的淘宝店:https://shop592330910.taobao.com/
```
按下CTRL + v之后，会取消下一个输入字符的特殊含义，例如CTRL + v后按下TAB键，可以在命令行下输入一个制表符，
或者之后按下CTRL + m会输入一个 Windows 下的回车符（注: ^M）。


18.注释当前输入的命令（在开头添加#号)
```
ESC + # 或者 ALT + #       马哥的淘宝店:https://shop592330910.taobao.com/
```

19.在文本编辑器中快速打开当前命令
```
CTRL + x CTRL + e        马哥的淘宝店:https://shop592330910.taobao.com/
```
按下以上快捷键可以将当前输入的命令用你最喜欢的文本编辑器打开，当退出编辑器后，该命令会被自动执行。

注：设置默认的编辑器方法，例如 vim：
```
export EDITOR='vim'       马哥的淘宝店:https://shop592330910.taobao.com/
```


20.删除光标左侧的字符
```
CTRL + h                  马哥的淘宝店:https://shop592330910.taobao.com/
```


21.删除光标所在处的字符
```
CTRL + d                马哥的淘宝店:https://shop592330910.taobao.com/
```
注：相当于 delete 键。



22.撤销上一次编辑操作（undo）
```
CTRL + x CTRL + u       马哥的淘宝店:https://shop592330910.taobao.com/
```


23.插入上一个命令的最后一个参数
```
ESC + . 或者 ALT + .     马哥的淘宝店:https://shop592330910.taobao.com/
```
在当前位置下，按下该建后可以快速插入上一个命令中的最后一个参数。


24.撤销对当前行的所有编辑操作
```
ESC + r 或者 ALT + r     马哥的淘宝店:https://shop592330910.taobao.com/
```


25.清除屏幕内容
```
CTRL + l               马哥的淘宝店:https://shop592330910.taobao.com/
```


26.切换成 vi 编辑风格
```
$ set -o vi          马哥的淘宝店:https://shop592330910.taobao.com/
```


马哥的淘宝店:https://shop592330910.taobao.com/


awk
http://www.catonmat.net/blog/awk-book/



























