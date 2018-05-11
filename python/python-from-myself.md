[TOC]

##### 当前时间
```
import datetime
datetime.now()

import time
time.strftime("%Y-%m-%d %H:%M:%S")
time.asctime()
```
** **
##### 线程睡眠
`time.sleep(5)`
** **
##### 时间差以秒表示,并且表刘两位小数
```
i = datetime.now()
j = datetime.now()
round((j-i).seconds/60,2)
```
** **
