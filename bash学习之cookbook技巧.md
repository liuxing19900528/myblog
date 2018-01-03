马哥的淘宝店:https://shop592330910.taobao.com/

bash cookbook 技巧

1.清空文件内容  Empty a file (truncate to 0 size)

```
$ > file
```

这一行命令用到了输出重定向操作符`>`。输出重定向发生时，文件会被打开准备写入。如果此时文件不存在则先创建，存在则将其大小截取为0（truncate to 0）。这里我们并没有重定向写任何内容到文件中，所以文件依然保持为空。

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
$ echo "foo bar baz" >> file
```

这个命令用到了另外一个输出重定向操作符`>>`，该操作符将内容追加到文件。同样地，如果文件不存在则先创建它。追加的内容之后，紧跟着换行符。如果你不想要追加换行符，在执行echo命令时可以指定-n选项：

```
$ echo -n "foo bar baz" >> file
```

在使用输出重定向的时候，bash可以设置个 `set -C` 来禁止覆盖已经存在的文件

```
$ set -C
$ echo "foo bar baz" > file 
bash: file: cannot overwrite existing file

```
此时如果确实想覆盖此文件可以使用`>|` 符号。




3.读取文件的首行并赋值给变量

```
$ read -r line < file
```

这一行命令用到了 Bash 的内置命令read，和输入重定向操作符`<`。read命令从标准输入中读取一行，并将内容保存到变量line中。在这里，-r选项保证读入的内容是原始的内容（raw），意味着反斜杠转义的行为不会发生。输入重定向操作符`< file`打开并读取文件file，然后将它作为read命令的标准输入。

记住，read命令会删除包含在IFS变量中出现的所有字符，IFS 的全称是 Internal Field Separator（内部字段分隔符），Bash 根据 IFS 中定义的字符来分隔单词。在这里，read命令读入的行被分隔成多个单词。默认情况下，IFS包含空格，制表符和回车，这意味着开头和结尾的空格和制表符都会被删除。如果你想保留这些符号，可以通过设置IFS为空来完成：

```
$ IFS= read -r line < file
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
    # do something with $line
done < file
```

这是一种正确的读取文件内容的做法，read命令放在while循环中。当read命令遇到文件结尾时（EOF），它会返回一个正值，导致循环判断失败终止。

记住，read命令会删除首尾多余的空白字符，所以如果你想保留，请设置 IFS 为空值:

```
$ while IFS= read -r line; do
    # do something with $line
done < file
```

如果你不想将< file放在最后，可以通过管道将文件的内容输入到 while 循环中：

```
$ cat file | while IFS= read -r line; do
    # do something with $line
done
```

对这里有个说明：对于循环读取标准输入的操作，很多程序内置有自己的标准输入链接到相同的输入源上面，和read命令一样。它偶尔会产生扭曲的结果。
 另一种方法是从不同的文件描述符读取数据。 
```
exec 3< input_file.txt                # open input_file.txt on fd 3

while read -u 3 -r line ; do
    # do stuff here
done

exec 3<&-                             # close fd 3
```

对于这个通过管道传给while循环的这个例子需要特别注意一点，这个while循环将会创建一个subshell子shell，之前的一些变量在这个while中是不存在的。

```
i=0
cat foo.txt | while read line; do ((i++)); done
echo "$i"

#will print 0.
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
$ read -r random_line < <(shuf file)
```

Bash 中并没有提供一种直接的方法来随机读取文件的某一行内容，所以这里需要利用外部程序。在最新的一些 Linux 系统上，GNU Coreutils 包中提供的`shuf`命令可以满足我们的需求。

这一行命令中用到了进程替换（process substitution）操作符`<(...)`。进程替换操作会创建一个匿名的管道文件，并将进程命令的标准输出连接到管道的写一端。然后 Bash 开始执行进程替换中的命令，然后将整个进程替换的表达式替换成匿名管道的文件名。

当 Bash 看到`<(shuf file)`时，它首先打开一个特殊的文件`/dev/fd/n`，这里的n是一个空闲的文件描述符，然后执行`shuf file`命令，将标准输出连接到`/dev/fd/n`，并且替换`<(shuf file)` 为`/dev/fd/n`，因此实际的命令会变成:

```
$ read -r random_line < /dev/fd/n
```

结果会读取洗牌后的文件的第一行内容。

另外一种做法是，使用 `GNU sort` 命令，它提供的-R选项可以随机排序文件：

```
$ read -r random_line < <(sort -R file)
```

或者，同前面一样，将结果赋值给变量：

```
$ random_line=$(sort -R file | head -1)
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
    # do something with $field1, $field2, and $field3
done < file
```

又或者，如果你的文件确实只有三个字段，那可以忽略它：

```
$ while read -r field1 field2 field3; do
    # do something with $field1, $field2, and $field3
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
$ printf "%c" {a..z} $'\n'
```
另外一个在结尾添加换行的方法是使用echo命令：

```
$ echo $(printf "%c" {a..z})
```
这个技巧使用了命令替换（substitution），里面执行的命令是`printf "%c" {a..z}`  然后替换命令的输出，最后echo打印这个输出，并且加上换行。


下面的又会输出什么呢？

```
$ printf "%c\n" {a..z}
```
这个会每个字符后面都加上一个换行的。

如果想要快速地将 printf 的结果保存到变量中，可以使用-v选项：

```
$ printf -v alphabet "%c" {a..z}
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
$ printf "%02d " {0..9}
```
这里我们又用到了printf的循环输出功能，这一次的输出格式为`"%02d "`，意思是在输出数字的时候，如果不满两位就用0补齐。同时，输出的元素是 0 到 9的列表（括号展开后的结果）。

如果你使用的是bash 4版本的，可以使用花括号展开的方式得到同样的结果：

```
$ echo {00..09}
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
$ echo {a,b,c}{1,2,3}
```
上面的命令会生成以下结果：`a1 a2 a3 b1 b2 b3 c1 c2 c3`。首先，它取出第一个括号中的第一个元素a，然后依次与第二个括号{1,2,3}的所有元素组合，生成a1 a2 a3，依此类推。

5.同一个字符串重复输出 10 次

```
$ echo foo{,,,,,,,,,,}
```
这个技巧也是使用到了花括号展开。这里foo要和后面的10个空的字符串进行组合，空的当然组合出来
还是本身啦。这样就是重复的得到了10个自身相同的字符串啦。

6.拼接字符串.

```
$ echo "$x$y"
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
$ IFS=- read -ra parts <<< "foo-bar-baz"
```
在这里，-a 选项告诉read命令将分割后的元素保存到数组parts中。随后，你可以通过`${parts[0]}, ${parts[1]}和${parts[2]}`  来访问数组的各个元素，或者通过`${parts[@]}`来访问所有元素。

8.逐个字符方式处理字符串

```
$ while IFS= read -rn1 c; do
    # do something with $c
done <<< "$str"
```
这里我们使用到了read命令的`-n1`  参数，它让read命令依次读入一个字符。类似我们可以使用`-n2`  参数来使read命令依次读取2个字符串。

9.字符串替换

```
$ echo ${str/foo/bar}
```
这个技巧用到了参数的展开。一个通用的格式是：`${var/find/replace}` 找到$var变量中的find字符串，并将它替换成bar。

10.检查字符串是否匹配某个模式

```
$ if [[ $file = *.zip ]]; then
    # do something
fi
```

这个小技巧用到了通配符匹配：如果`$file`的值匹配`*.zip`，则执行if语句里的命令。这种语法下的模式是最简单的通配符（glob pattern）匹配，通配符包括`* ? [...]`。其中，`*`可以匹配一个或者多个字符， `?`只能匹配单个字符，`[...]`能够匹配任意出现在中括号里面的字符或者一类字符集。



下面是另外一个例子，用来判断回答是否匹配 Y 或者 y:er is Y or y:

```
$ if [[ $answer = [Yy]* ]]; then
    # do something
fi
```
注意这个和正则表达式还是不太一样的，虽然很类似，但是不是一回事啊。


11.检查字符串是否匹配某个正则表达式

```
$ if [[ $str =~ [0-9]+\.[0-9]+ ]]; then
    # do something
fi
```
这一行命令检查`$str`是否能够匹配正则表达式`[0-9]+\.[0-9]+`，即两个数字中间包含一个点号。正则表达式的规范可以通过 man 手册查询: man 7 regex

```
$ str='foo12345bar67890'
$ re='[^0-9]+([0-9]+)[^0-9]+([0-9]+)'

$ [[ $str =~ $re ]] && x=${BASH_REMATCH[1]} y=${BASH_REMATCH[2]}

$ echo "$x/$y
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
$ str="hello world"
$ echo ${str:6}
```
这一小技巧通过子串提取操作，从字符串hello world中取到了子串world。子串提取操作的语法格式为`${var:offset:length}`，它的意思是说从变量var中，提取第offset个位置（下标从0开始计算）开始的总共length个数的字符。在我们这个例子中，忽略了length，默认会返回所有剩余的字符。
下面是另外一个例子，返回$str变量中第7、8位置的两个字符：

```
$ echo ${str:7:2}
```

14.转换成大写

```
$ declare -u var
$ var="foo bar"
```
Bash 中的内置命令 declare 可以用于声明一个变量，或者设置变量的属性。在这个例子中，通过指定-u选项，使得变量$var在赋值时，就会自动地将内容转换成大写的格式。现在你 echo 它，可以看到所有内容已经变成大写了：

```
$ echo $var
FOO BAR
```
注意，-u选项也是在 Bash 4 新版本中引入的功能，在低版本下是没有的。类似地，你还可以使用 Bash 4 提供的另外一种参数展开语法`${str^^}`，也可以将字符串转换成太写的格式：

```
$ str="zoo raw"
$ echo ${str^^}
```

15.转换成小写

```
$ declare -l var
$ var="FOO BAR"
```

同上面一条类似，-l选项声明变量的小写属性，使得其值转换成小写的格式：

```
$ echo $var
foo bar
```

同样，只有 Bash 4 以及以上的版本才支持-l选项。另外一种方式是使用参数展开语法:

```
$ str="ZOO RAW"
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
$ echo "${str~} ${str~~}"
fOObar fooBAR
```

```
$ echo "${str~F} ${str~[a-f]} ${str~~[bBfF]} ${str~~[^a-f]}"
fOObar FOObar fOOBar foobaR
```

