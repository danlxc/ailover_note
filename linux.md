# linux 常用命令

## 安装源

```
yum install epel-release
```

## 查看命令目录

```
which ipython
whereis ipython
```

## 查询命令历史

```
history  | grep aa
```

## 查询进程

```
ps -ef | grep aa
```



tail

1. head
2. cat
3. tac
4. echo

## 1. tail {#1-tail}

```
tail -f test.log
```

你会看到屏幕不断有内容被打印出来. 这时候中断第一个进程Ctrl-C。

## 2. head {#2-head}

linux 如何显示一个文件的某几行\(中间几行\)

从第3000行开始，显示1000行。即显示3000~3999行

```
cat test.log | tail -n +3000 | head -n 1000
```

显示1000行到3000行

```
cat test.log | head -n 3000 | tail -n +1000
```

\*注意两种方法的顺序

分解：

```
tail -n 1000：显示最后1000行
tail -n +1000：从1000行开始显示，显示1000行以后的
head -n 1000：显示前面1000行
```

用sed命令

```
sed -n '5,10p' test.log 这样你就可以只查看文件的第5行到第10行。
```

例：

```
cat test.
log
 | tail -n 
1000
 # 输出mylog.
log
 文件最后一千行

```

## 3. cat {#3-cat}

cat主要有三大功能：  
 1.一次显示整个文件。

```
$ 
cat filename

```

2.从键盘创建一个文件。

```
$ 
cat 
>
 filename 

```

只能创建新文件,不能编辑已有文件.  
 3.将几个文件合并为一个文件：

```
$cat
 file1 file2 
>
file

```

参数：

```
-n
 或 
--
number 由 
1
 开始对所有输出的行数编号

-b
 或 
--
number
-nonblank
 和 
-n
 相似，只不过对于空白行不编号

-s
 或 
--
squeeze
-blank
 当遇到有连续两行以上的空白行，就代换为一行的空白行

-v
 或 
--
show
-nonprinting

```

例：

把 textfile1 的档案内容加上行号后输入 textfile2 这个档案里

```
cat 
-n
 textfile1 
>
 textfile2

```

把 textfile1 和 textfile2 的档案内容加上行号（空白行不加）之后将内容附加到 textfile3 里。

```
cat 
-b
 textfile1 textfile2 
>
>
 textfile3

```

把test.txt文件扔进垃圾箱，赋空值test.txt

```
cat /dev/
null
>
/etc/
test.txt 

```

注意：&gt;意思是创建，&gt;&gt;是追加。千万不要弄混了。

## 4. tac {#4-tac}

tac \(反向列示\)

tac 是将 cat 反写过来，所以他的功能就跟 cat 相反。

cat 是由第一行到最后一行连续显示在萤幕上，

而 tac 则是由最后一行到第一行反向在萤幕上显示出来！

## 5. echo {#5-echo}

在Linux中echo命令用来在标准输出上显示一段字符，比如：

```
echo
"the echo command test!"

```

这个就会输出“the echo command test!”这一行文字！

```
echo
"the echo command test!"
>
a.sh

```

这个就会在a.sh文件中输出“the echo command test!”这一行文字！

该命令的一般格式为： echo \[ -n \] 字符串其中选项n表示输出文字后不换行；字符串能加引号，也能不加引号。

用echo命令输出加引号的字符串时，将字符串原样输出；

用echo命令输出不加引号的字符串时，将字符串中的各个单词作为字符串输出，各字符串之间用一个空格分割。

# grep 同时满足多个关键字和满足任意关键字 {#grep-同时满足多个关键字和满足任意关键字}

## 1.满足任意条件 {#1满足任意条件}

```
grep -
E
"word1|word2|word3"
   file.txt

```

满足任意条件（word1、word2和word3之一）将匹配。

## 2.满足所有条件 {#2满足所有条件}

```
grep word1 file.txt 
| grep word2 |grep word3

```

必须同时满足三个条件（word1、word2和word3）才匹配。

多管道，多次筛选。

## 3.使用正则表达式 -E 选项 {#3使用正则表达式-e-选项}

```
grep -
E
"[1-9]+"

```

或

```
egrep 
"[1-9]+"

```

# 常用命令 {#常用命令}

## 1.查看日志常用命令 {#1查看日志常用命令}

```
tail:  
   -n  是显示行号；相当于nl命令；例子如下：
        tail -100f      test.log        实时监控100行日志
        tail  -n  10    test.log        查询日志尾部最后10行的日志;
        tail -n +10     test.log        查询10行之后的所有日志;
head:  
    跟tail是相反的，tail是看后多少行日志；例子如下：
        head -n 10      test.log   查询日志文件中的头10行日志;
        head -n -10     test.log   查询日志文件除了最后10行的其他所有日志;
cat： 
    tac是倒序查看，是cat单词反写；例子如下：
        cat -n test.log |grep "debug"   查询关键字的日志


```

## 1. 按行号查看—过滤出关键字附近的日志 {#1-按行号查看过滤出关键字附近的日志}

1）cat -n test.log \| grep “debug”

得到关键日志的行号

2）cat -n test.log \| tail -n +92 \| head -n 20

选择关键字所在的中间一行. 然后查看这个关键字前10行和后10行的日志:

tail -n +92 表示查询92行之后的日志

head -n 20 则表示在前面的查询结果里再查前20条记录

## 2. 根据日期查询日志 {#2-根据日期查询日志}

```
sed 
-n
'/2017-08-23 00:05:00,446/,/2017-08-23 00:05:00,546/p'
  test
.
log

```

特别说明:上面的两个日期必须是日志中打印出来的日志,否则无效；

先 grep ‘2014-12-17 16:17:20’ test.log 来确定日志中是否有该 时间点

## 3. 日志内容特别多，打印在屏幕上不方便查看 {#3-日志内容特别多打印在屏幕上不方便查看}

1.使用 more 和 less 命令,

```
cat -n test.
log
 |
grep
"debug"
 |more     

```

这样就分页打印了,通过点击空格键翻页

2.使用 &gt;xxx.txt 将其保存到文件中,到时可以拉下这个文件分析

```
cat -n test.
log
 |
grep
"debug"
>
debug.txt

```

# more命令 {#more命令}

more命令是一个基于vi编辑器文本过滤器，它以全屏幕的方式按页显示文本文件的内容，支持vi中的关键字定位操作。

该命令一次显示一屏文本，满屏后停下来，并且在屏幕的底部出现一个提示信息，给出至今己显示的该文件的百分比：–More–（XX%）

可以用下列不同的方法对提示做出回答：

| 按键 | 说明 |
| :--- | :--- |
| 按Space键： | 显示文本的下一屏内容。 |
| 按B键： | 显示上一屏内容。 |
| 按Enter键： | 只显示文本的下一行内容。 |
| 按斜线符： | 接着输入一个模式，可以在文本中寻找下一个相匹配的模式。 |
| 按H键： | 显示帮助屏，该屏上有相关的帮助信息。 |
| 按Q键： | 退出more命令 |



