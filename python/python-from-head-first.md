[TOC]
## 特殊小记
### 安装模块
`pip install DBUtils`

### 数组
```
movies = ["The Holy Grail", 1975,"Terry Jone & Terryt Gilliam", 91,
          ["Graham Chapman",
           ["Michael palin", "John Cleese", "Terry Gilliam", "Eric Idle",
            ["Terry Jones"]]]]
```
** **
##### 数组的长度
`len(movies)`

`movies.append("Danny")`
** **

##### 移除最后一个
`movies.pop()`
** **

##### 尾部
`movies.extend()`
** **

##### 移除特定
`movies.remove("Danny")`
** **

##### 插入自当索引的
`movies.insert(0,"Chapman")`
** **

##### 遍历list
```
for each_item in movies:
	print(each_item)
```
** **

##### 定义一个函数(递归调用)
```
def print_lol(the_list):

	"""This function takes one positional argument called "the_list", which is
		any python list(of - possibly - nested lists). Each data item in the
		provided list id (recursively) printed to the screen on it's own line."""

	for each_item in the_list:
		if isinstance (each_item, list):
			print_lol(each_item)
		else:
			print (each_item)
```
```
import nester
from nester import print_lol

for num in range(4):
	print(num)
```
** **

##### BIF( 命名空间 `_builtins_` )
`buit-in functions`
** **

### 文件以异常

##### 读取一个文件(Open a named file, assign the file to a file object called “data”.)
`data = open('sketch.txt')`
** **

##### 获取当前目录(What’s the current working directory?)
`os.getcwd()`
** **

##### 改变为某个目录(Change to the folder that contains your data file)
`os.chdir`
** **

##### 打印一行(“readline()” method to grab a line from the file)
*end 表示在打印的结尾处取消默认的换行,而是以空格替代*
`print(data.readline(), end='')`
** **

##### 重置打印文件的索引到开头(“seek()” method to return to the start of the file.)
*And yes, you can use “tell()” with Python’s files, too.*
`data.seek()`
** **

#####  循环打印每行文件(it’s a standard iteration using the file’s data as input)
```
for each_line in data:
	print(each_line,end='')
```
** **

##### 关闭文件(Since you are now done with the file, be sure to close it.)
`data.close()`
** **

##### 文件分割(associated with the “each_line” string and break the string whenever a “:” is found)
`each_line.split(":")`
** **

#####  文件分割的返回值(returns a list of strings, which are assigned to a list of target identifiers. multiple assignment)
`(role, line_spoken)= each_line.split(":")`

*S.split([sep[, maxsplit]]) -> list of strings Return a list of the words in S, using sep as the delimiter string. If maxsplit is given, at most maxsplit splits are done. If sep is not specified or is None, any whitespace string is a separator and empty strings are removed from the result.如下表示只切分一次*

`(role, line_spoken)= each_line.split(":",1)`
** **

##### 在一个字符串中找了一个字符串( find() to try and locate a substring in another string)
`each_line.split(":")`

*if it can’t be found, the find() method returns the value -1. If
the method locates the substring, find() returns the index position of the
substring in the string.*
** **


##### 异常处理(which exists to provide you with a way to systematically handle exceptions and errors at runtime)
```
try:
    your code (which might cause a runtime error)
except:
    your error-recovery code
```
** **

##### pass 关键字

*When you have a situation where you might be expected to provide code, but
don’t need to, use Python’s pass statement (which you can think of as the
empty or null statement.)*

`pass`
****

##### 新知识,代码实例(异常时的两种处理逻辑)
```
import os

if os.path.exists('sketch.txt'):
    data = open('sketch.txt')

    for each_line in data:
        if not each_line.find(":") == -1 :
            (role, line_spoken) = each_line.split(":", 1)
            print(role, end='')
            print(' said: ', end='')
            print(line_spoken, end='')
        else:
            pass

    data.close()
else:
    print('The data file is missing!')
```

```
try:
    data = open('sketch.txt')

    for each_line in data:
        try:
            (role, line_spoken) = each_line.split(":", 1)
            print(role, end='')
            print(' said: ', end='')
            print(line_spoken, end='')
        except ValueError:
            pass

    data.close()
except IOError:
    print('The data file is missing!')

```
** **

### 保存文件

##### 去除空白字符 (remove unwanted whitespace)
`line_spoken.strip()`

##### 获取数据文件对象(w)
*When you use the open() BIF to work with a disk file, you can specify an
access mode to use. By default, open() uses mode r for reading, so you don’t
need to specify it. To open a file for writing, use mode w*

`out = open('data.out', 'w')`
** **

##### 打印数据到文件(use the file argument to specify the data file object to use)
`print('Norwegian blues stun easily.', file=out)`

*When you’re done, be sure to close the file to ensure all of your data is written
to disk. This is known as flushing and is very important*

`out.close()`

*When you use access mode  w , Python opens your named file
for writing. If the file already exists, it is cleared of its contents, or
clobbered. To append to a file, use access mode  a , and to open a
file for writing and reading (without clobbering), use  w+ . If you
try to open a file for writing that does not already exist, it is first
created for you, and then opened for writing.*
** **



