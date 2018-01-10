awk学习之cookbook技巧.md


马哥的淘宝店:https://shop592330910.taobao.com/

1.每行后面都加上一个空行
```bash
awk '1; { print "" }'       #马哥的淘宝店:https://shop592330910.taobao.com/
```

awk 程序包含一系列的 parttern-action（模式-动作）的语句。类似这样‘pattern{action}’的结构。
这个例子中包含了2个语句，一个是 “1”，另一个是“{print “” }”，模式和动作都可以省略的其中一个的。
如果模式省略了，默认是对所有的输入内容按行来出来的。如果是动作省略的默认是“{print }”。

注意awk程序是面向行的。马哥的淘宝店:https://shop592330910.taobao.com/

上面的等同于下面这个程序
```bash
awk '1 { print } { print "" }'     #马哥的淘宝店:https://shop592330910.taobao.com/
```

动作只有在模式匹配的情况下才执行。这里例子中的模式是“1”，这个模式总是true，也就是每一行都会匹配，
每一行也都会执行动作的。所以也等价于下面的代码：
```bash

awk '{ print } { print "" }'   #马哥的淘宝店:https://shop592330910.taobao.com/

```
awk的print语句总是会在最后打印一个ORS（output record separator）变量的，这个值模式是换行，
也就是\n  #马哥的淘宝店:https://shop592330910.taobao.com/
这里例子中的第一个print语句后面没有接任何参数，所有默认等于```print $0```，也就是打印整行内容，
$0代表的是整行内容。    #马哥的淘宝店:https://shop592330910.taobao.com/
例子中的第二个print 打印一个空字符串，也是上面都不输出，打上print会在最后加上一个换行，所以呢
这里就打印出来了一个空行了。#马哥的淘宝店:https://shop592330910.taobao.com/

2.另外一个给每行后面加上空行的技巧
```bash
awk 'BEGIN { ORS="\n\n" }; 1' #马哥的淘宝店:https://shop592330910.taobao.com/
```
BEGIN是一个特殊的模式，表示在读取内容之前无条件执行后面的动作。
这个技巧中是在动作语句中对ORS变量重新赋值了，将一个换行变成两个换行了。
后面 的1 等价于 {print} ，这样就达到了每行后面都加上新的空行的目的。
```
$ awk 'BEGIN { ORS="\n\n" }; 1' /etc/passwd
root:x:0:0:root:/root:/bin/bash

daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin

bin:x:2:2:bin:/bin:/usr/sbin/nologin

```

3.在每个非空的行后面添加空行
```bash
awk 'NF { print $0 "\n" }' #马哥的淘宝店:https://shop592330910.taobao.com/

```  
这个技巧用到了awk中的另外一个特殊变量NF（number of fields）。这个表示每行被awk分割为多少个字段，
也就是多少列，默认是安装空格分割的。例如“this is a test”这行会被分割为4个字段。
如果是空行，是不会分割的，也就是NF的值是0。#马哥的淘宝店:https://shop592330910.taobao.com/
这里使用NF作为模式，只有NF值大于0条件才是真，后面的动作才执行。 
后面的动作是print $0，也就是打印整行内容，print第二个参数是个换行，也就是后面加个新的空行。

4、在每行后添加两个空行
```bash
awk '1; { print "\n" }' #马哥的淘宝店:https://shop592330910.taobao.com/

```
1是等于{print}的，所以上面的可以等价于下面这个  #马哥的淘宝店:https://shop592330910.taobao.com/
```bash
awk '{ print; print "\n" }'  #马哥的淘宝店:https://shop592330910.taobao.com/

``` 
首先是打印一整行内容，然后再打印一个换行。 #马哥的淘宝店:https://shop592330910.taobao.com/

5、为每个文件的内容添加行号
```bash
awk '{ print FNR "\t" $0 }' #马哥的淘宝店:https://shop592330910.taobao.com/

``` 
这则技巧是在没一行的开头加上FNR（file line number行号）然后是一个\t（ tab键），最后是整行内容。
FNR变量记录了当前的行号。FNR会每次重置为0的。
```bash
$ awk '{ print FNR "\t" $0 }' ffmpeg.sh  
1	#!/bin/bash #马哥的淘宝店:https://shop592330910.taobao.com/
2	
3	INPUT_FILE="$1"
4	ffmpeg -y -i "$INPUT_FILE" -vf drawtext="fontfile=fonts/simsun.ttc: text='欢迎光临 马哥的淘宝店<马哥私房菜> 地址是：shop592330910.taobao.com':fontcolor=red:fontsize=40:box=1:boxcolor=black@0:x=if(eq(mod(t\,3)\,0)\,rand(0\,(w-text_w))\,x):y=h-text_h" -codec:a copy "new.${INPUT_FILE}"
$ 

```   

6、为所有文件的所有行统一添加行号
```bash

awk '{ print NR "\t" $0 }'      #马哥的淘宝店:https://shop592330910.taobao.com/

```
这个和上面的那个例子类似，只是这里使用NR（line number）变量了，这个不会重置为0的。
```bash

$ awk '{ print FNR "\t" $0 }' ffmpeg.sh ffmpeg.sh                                                                                                           
1	#!/bin/bash  #马哥的淘宝店:https://shop592330910.taobao.com/
2	
3	INPUT_FILE="$1"
4	ffmpeg -y -i "$INPUT_FILE" -vf drawtext="fontfile=fonts/simsun.ttc: text='欢迎光临 马哥的淘宝店<马哥私房菜> 地址是：shop592330910.taobao.com':fontcolor=red:fontsize=40:box=1:boxcolor=black@0:x=if(eq(mod(t\,3)\,0)\,rand(0\,(w-text_w))\,x):y=h-text_h" -codec:a copy "new.${INPUT_FILE}"
1	#!/bin/bash   #马哥的淘宝店:https://shop592330910.taobao.com/
2	
3	INPUT_FILE="$1"
4	ffmpeg -y -i "$INPUT_FILE" -vf drawtext="fontfile=fonts/simsun.ttc: text='欢迎光临 马哥的淘宝店<马哥私房菜> 地址是：shop592330910.taobao.com':fontcolor=red:fontsize=40:box=1:boxcolor=black@0:x=if(eq(mod(t\,3)\,0)\,rand(0\,(w-text_w))\,x):y=h-text_h" -codec:a copy "new.${INPUT_FILE}"






                                                                                                                                                            
$ awk '{ print NR "\t" $0 }' ffmpeg.sh ffmpeg.sh                                                                                                             
1	#!/bin/bash  #马哥的淘宝店:https://shop592330910.taobao.com/
2	
3	INPUT_FILE="$1"
4	ffmpeg -y -i "$INPUT_FILE" -vf drawtext="fontfile=fonts/simsun.ttc: text='欢迎光临 马哥的淘宝店<马哥私房菜> 地址是：shop592330910.taobao.com':fontcolor=red:fontsize=40:box=1:boxcolor=black@0:x=if(eq(mod(t\,3)\,0)\,rand(0\,(w-text_w))\,x):y=h-text_h" -codec:a copy "new.${INPUT_FILE}"
5	#!/bin/bash  #马哥的淘宝店:https://shop592330910.taobao.com/
6	
7	INPUT_FILE="$1"
8	ffmpeg -y -i "$INPUT_FILE" -vf drawtext="fontfile=fonts/simsun.ttc: text='欢迎光临 马哥的淘宝店<马哥私房菜> 地址是：shop592330910.taobao.com':fontcolor=red:fontsize=40:box=1:boxcolor=black@0:x=if(eq(mod(t\,3)\,0)\,rand(0\,(w-text_w))\,x):y=h-text_h" -codec:a copy "new.${INPUT_FILE}"
$

```
我们可以看到当作用到2个文件来显示行号的时候 FNR是会在第二个文件重置为0 的。 
NR变量则不会重置为0 的。


7、 格式化行号
```bash

awk '{ printf("%5d : %s\n", NR, $0) }'  #马哥的淘宝店:https://shop592330910.taobao.com/

```
这个例子使用到了printf 函数，这个类似bash/c语言中的printf（）函数。这个函数是不会在每行结尾追加打印ORS，也就是不会打印换行的。
```bash

$ awk '{ printf("%5d : %s\n", NR, $0) }' ffmpeg.sh ffmpeg.sh      #马哥的淘宝店:https://shop592330910.taobao.com/
    1 : #!/bin/bash #马哥的淘宝店:https://shop592330910.taobao.com/
    2 : 
    3 : INPUT_FILE="$1"
    4 : ffmpeg -y -i "$INPUT_FILE" -vf drawtext="fontfile=fonts/simsun.ttc: text='欢迎光临 马哥的淘宝店<马哥私房菜> 地址是：shop592330910.taobao.com':fontcolor=red:fontsize=40:box=1:boxcolor=black@0:x=if(eq(mod(t\,3)\,0)\,rand(0\,(w-text_w))\,x):y=h-text_h" -codec:a copy "new.${INPUT_FILE}"
    5 : #!/bin/bash #马哥的淘宝店:https://shop592330910.taobao.com/
    6 : 
    7 : INPUT_FILE="$1"
    8 : ffmpeg -y -i "$INPUT_FILE" -vf drawtext="fontfile=fonts/simsun.ttc: text='欢迎光临 马哥的淘宝店<马哥私房菜> 地址是：shop592330910.taobao.com':fontcolor=red:fontsize=40:box=1:boxcolor=black@0:x=if(eq(mod(t\,3)\,0)\,rand(0\,(w-text_w))\,x):y=h-text_h" -codec:a copy "new.${INPUT_FILE}"


```


8.非空行前面添加行号
```bash
awk 'NF { $0=++a " :" $0 }; { print }' #马哥的淘宝店:https://shop592330910.taobao.com/
```
awk中是可以使用变量的，当然也可以使用自定义的变量，和bash是类似的，不需要先定义在使用，可以直接使用的。
之前我们用到的那些个变量都是awk自己定义的。#马哥的淘宝店:https://shop592330910.taobao.com/

这个例子怎么理解呢？ 
首先 这里是2个模式动作语句。第一个是 ```NF { $0=++a " :" $0 }``` ，第二个是``` { print } ```
第一个里面 ++a是定义了一个变量a，这里初始就是0（数字零），而不是什么空或者空字符串。
为啥是数字也是因为这个++操作符是作用到数字上面的。#马哥的淘宝店:https://shop592330910.taobao.com/
然后执行++操作（类似c语言里面的前置++）a第一次执行就会变为1了。
什么时候第一次执行呢也就是第一个不是空行的时候才是首次执行。因为NF模式匹配非空行。上面有个例子讲过。

然后是```  ++a  “ ：” $0 ``` 这3个值按照字符串拼接那样拼接起来，然后一起重新复制给变量$0了，$0表示整行内容，
这里就实现了给非空行重新设置行号的目的了。注意拼接字符串不能使用加号，加号只能作用到数字上面。

第一个模式动作语句是不打印内容的。#马哥的淘宝店:https://shop592330910.taobao.com/

然后是第二个模式动作语句，里面只有一个print，默认就是打印$0，默认就是打印整行内容。
这里不管是空行，还是非空行都统统的打印出来，非空行因为是之前在第一个模式动作中重新被
赋值了，这里也就实现了打印非空行行号的目的了。#马哥的淘宝店:https://shop592330910.taobao.com/

同时我们可以看到这个awk程序作用到2个文件的时候，变量a是不会被重置的。#马哥的淘宝店:https://shop592330910.taobao.com/


```bash

$ awk 'NF { $0=++a " :" $0 }; { print }' ffmpeg.sh ffmpeg.sh     #马哥的淘宝店:https://shop592330910.taobao.com/
1 :#!/bin/bash #马哥的淘宝店:https://shop592330910.taobao.com/

2 :INPUT_FILE="$1"
3 :ffmpeg -y -i "$INPUT_FILE" -vf drawtext="fontfile=fonts/simsun.ttc: text='欢迎光临 马哥的淘宝店<马哥私房菜> 地址是：shop592330910.taobao.com':fontcolor=red:fontsize=40:box=1:boxcolor=black@0:x=if(eq(mod(t\,3)\,0)\,rand(0\,(w-text_w))\,x):y=h-text_h" -codec:a copy "new.${INPUT_FILE}"
4 :#!/bin/bash #马哥的淘宝店:https://shop592330910.taobao.com/

5 :INPUT_FILE="$1"
6 :ffmpeg -y -i "$INPUT_FILE" -vf drawtext="fontfile=fonts/simsun.ttc: text='欢迎光临 马哥的淘宝店<马哥私房菜> 地址是：shop592330910.taobao.com':fontcolor=red:fontsize=40:box=1:boxcolor=black@0:x=if(eq(mod(t\,3)\,0)\,rand(0\,(w-text_w))\,x):y=h-text_h" -codec:a copy "new.${INPUT_FILE}"


```



9.计算文件行数（模拟 wc -l 命令）
```bash

awk 'END { print NR }'  #马哥的淘宝店:https://shop592330910.taobao.com/

```
这里又用到了一个特殊的模式END，这个表示所有行处理完，最后在执行这个模式后面的动作。
这个例子就是打印NR的值，也就是最终的行数。 #马哥的淘宝店:https://shop592330910.taobao.com/
```bash
$ awk 'END { print NR }' ffmpeg.sh  #马哥的淘宝店:https://shop592330910.taobao.com/
11

$ awk 'END { print NR }' ffmpeg.sh ffmpeg.sh ffmpeg.sh     #马哥的淘宝店:https://shop592330910.taobao.com/
33

```

10、对每行的所有的列求和

```bash
awk '{ s = 0; for (i = 1; i <= NF; i++) s = s+$i; print s }'  #马哥的淘宝店:https://shop592330910.taobao.com/

#或分开多行书写：

awk '{  #马哥的淘宝店:https://shop592330910.taobao.com/
s = 0; 
for (i = 1; i <= NF; i++) 
    s = s+$i; 
print s 
}'

```
这个例子使用到了awk编程中的for循环，这个和c语言的for循环类似。
这个例子是把每行的所有列都加到一起 复制给变量s，然后打印总和。
每次计算一行的总和，然后就输出。而不是把所有行加起来。下面例子11才是把所有行加起来的。
特别注意这里是``` s + $i ``` ，要特别注意那个美元符，这里是需要美元符号的，有
美元符号表示的列的值，没有的表示的循环变量 i的值，i的值会是1,2,3,4,5,6等。但是列的值可不一定是这样的。

下面我们看执行结果，直接复制代码执行后面不跟文件，会停留等待用户输入，这个时候你可以输入一行数字，
每个数字可以空格分割，每输入一行（按完回车键）就会执行一次计算，然后打印。
结束输入可以按ctrl+D键。
```bash

$ awk '{  #马哥的淘宝店:https://shop592330910.taobao.com/
s = 0;
for (i = 1; i <= NF; i++)
    s = s+$i;
print s
}'          
1 2 3 4 5              # 这行是输入内容
15
1 2 3 4 5 6 7 8 9 10   # 这行是输入内容
55


```

11.对所有的行所有的列求和
```bash

awk '{ for (i = 1; i <= NF; i++) s = s+$i }; END { print s+0 }'  #马哥的淘宝店:https://shop592330910.taobao.com/

#或分开多行书写： #马哥的淘宝店:https://shop592330910.taobao.com/
awk '{ 
for (i = 1; i <= NF; i++) 
    s = s+$i 
}; 

END { print s+0 }' #马哥的淘宝店:https://shop592330910.taobao.com/


```
注意这个例子11和例子10的区别，例子10 有个 s = 0的语句，使得每次计算新的一行都会重新初始化为0.
这个例子与上一个基本一致，除了输出的是所有行所有字段的和。由于变量会被自动定义，s只需要定义一次，
故而不需要把s定义成0。另外需要注意的是，最后在END模式里面它输出{print s+0}而非{print s}，
这是因为如果文件为空，s不会被定义就不会有任何输出了，输出s+0可以保证在这种情况下也会输出更有意义的0。
或者我们可以写一个BEGIN{s = 0 }，来一个s初始值的设置。这样最后可以不用打印s + 0啦。

下面我们看执行结果，直接复制代码执行后面不跟文件，会停留等待用户输入，这个时候你可以输入一行数字，
每个数字可以空格分割，每输入一行就会执行一次计算/或打印，这个例子是最后才打印结果。
结束输入可以按ctrl+D键。
```bash

$ awk '{    #马哥的淘宝店:https://shop592330910.taobao.com/
for (i = 1; i <= NF; i++)
    s = s+$i
};

END { print s }' #马哥的淘宝店:https://shop592330910.taobao.com/ 
1 2 3 4 5 6  # 这行是输入内容
21

```
```bash

$ awk '{   #马哥的淘宝店:https://shop592330910.taobao.com/
for (i = 1; i <= NF; i++)
    s = s+$i
};

END { print s }' #马哥的淘宝店:https://shop592330910.taobao.com/
10 9 8 7 6      # 这行是输入内容
5 4 3 2 1 0     # 这行是输入内容
-1 -2 -3 -4 -5  # 这行是输入内容   
40
 

```

12、将所有字段替换为其绝对值
```bash

awk '{ for (i = 1; i <= NF; i++) if ($i < 0) $i = -$i; print }'   #马哥的淘宝店:https://shop592330910.taobao.com/


awk '{     #马哥的淘宝店:https://shop592330910.taobao.com/
  for (i = 1; i <= NF; i++) {
    if ($i < 0) {
      $i = -$i;
    }
  }
  print
}'    #马哥的淘宝店:https://shop592330910.taobao.com/

```
这条语句用了if语句，和C语言类似。
它对每一行检查，检查每个字段的值是否小于0，如果值小于0，则将其改为正数，然后重新赋值给这个字段。
字段名可以间接地用变量的形式引用，如i=5;$i='hello'会将第5个字段的内容置为hello。

13、计算文件中的总字段（单词）数
```bash

awk '{ total = total + NF }; END { print total+0 }' #马哥的淘宝店:https://shop592330910.taobao.com/

```
这个例子是对每行的NF（NF就是字段数，列数，或者说是单词个数）值进行累加，最后打印输出
这里最后为什么是``` print total+0 ``` 可以参考例子11的说明。

14、输出含有单词“马哥私房菜”的行的数目
```bash

awk '/马哥私房菜/ { n++ }; END { print n+0 }' #马哥的淘宝店:https://shop592330910.taobao.com/

```
这个例子含有两个语句。第一句找出匹配/马哥私房菜/的行，并对变量n进行累加。在/…/之间的内容为正则表达式，
/马哥私房菜/匹配所有含有“马哥私房菜”的单词。
第二句在文件处理完成后输出n的数值。这里用n+0是为了让n为空的情况下输出0而不是一个空行。

15、寻找第一个字段为数字且最大的行
```bash

awk '$1 > max { max=$1; maxline=$0 }; END { print max, maxline }' #马哥的淘宝店:https://shop592330910.taobao.com/

```
这个例子用变量max记录第一个字段的最大值，并把第一个字段最大的行的内容存在变量maxline中。
在循环终止后，输出max和maxline的内容。
马哥的淘宝店:https://shop592330910.taobao.com/

注意：如果在数字都为负数的情况下，这个例子就不能用了，下面的是修改过的版本
```bash

awk 'NR == 1 { max = $1; maxline = $0; next; } $1 > max { max=$1; maxline=$0 }; END { print max, maxline }'

```

16、在每一行前添加该行的字段数，并打印
```bash

awk '{ print NF ":" $0 } ' #马哥的淘宝店:https://shop592330910.taobao.com/

```
这个例子仅仅是在逐行输出字段数NF，一个冒号，以及该行的内容。


17、输出每行的最后一个字段
```bash

awk '{ print $NF }' #马哥的淘宝店:https://shop592330910.taobao.com/

```
awk里面的字段可以用变量的形式引用。这一句输出第NF个字段的内容，而NF就是该行的字段数。


18、打印最后一行的最后一个字段
```bash
awk '{ field = $NF }; END { print field }' #马哥的淘宝店:https://shop592330910.taobao.com/
```

这个例子用field记录最后一个字段的内容，并在循环后输出field的内容。

这里是一个更好的版本。它更常用、更简洁也更高效：
```bash

awk 'END {print $NF}' #马哥的淘宝店:https://shop592330910.taobao.com/

```
19、输出所有字段数大于4的行
```bash

awk 'NF > 4'         #马哥的淘宝店:https://shop592330910.taobao.com/

awk 'NF > 4{print}'  #马哥的淘宝店:https://shop592330910.taobao.com/

```

这个例子省略了要执行的动作。如前所述，省略动作等价于{print}。


20、输出所有最后一个字段大于4的行
```bash

awk '$NF > 4'  #马哥的淘宝店:https://shop592330910.taobao.com/

```
这个例子用$NF引用最后一个字段，如果它的数值大于4，那么就输出。















