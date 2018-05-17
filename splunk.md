### splunk 笔记

##### java 调用接口 By LuYu \(查询注册用户数\)

```
|dbxquery query="SELECT DATE_FORMAT($args.fieldName$, $args.timeType$ ) date, count(1) count FROM cj_user WHERE merchant_id = $args.bankFlag$ AND DATE_FORMAT($args.fieldName$, $args.timeType$ ) BETWEEN $args.beginTime$ AND $args.endTime$ GROUP BY  date" connection="prod"
```

#### 统计用户资产 By HaiLong

    |dbxquery query="SELECT * FROM `wirich2`.`cjqfweb_user_asset_one`" connection="prod"

    |stats count as 持有基金数   sum(fundmarketvalue) as 总市值   sum(floatprofit) as 总收益   sum(costmoney) as 总本金 by custno

    |lookup username.csv custno output name as 姓名

    |rename custno  as 客户号

    |table 客户号 姓名 持有基金数 总市值 总收益 总本金

    |eval 总收益率=round('总收益'/'总本金'*100,2)."%"

#### 统计

```
index="prod01"  source!=/www/logs/redis/* source!=/www/logs/nginx/*   source!=/www/logs/zookeeper/* source!=*stdout.log  fundAmountMap OR 购买状态 OR 支付状态 OR 组合购买异步结束

|rex "^\[[^\]]*\] \[(?<loglevel>[^\]]*)\] \[(?<msgtype>.*)\] \[(?<userid>[^\]]*)\] \[(?<traceid>[^\]]*)\]"

|rex "(?<fundecode>\w*)\.OF=(?<jine>[^,}]*)[,}]" max_match=0

|eventstats sum(jine) as totalbuy by traceid

|rex "组合购买异步结束，成功购买总金额为：(?<shijibuy>.*)"

|rex "基金: (?<fundcode>\w*) 购买状态: (?<buystatus>[^,]*),retCode：(?<buyreturncode>\d*)，retMsg：(?<buyretrunmsg>[^,]*), 耗时: (?<buyspentime>\w*) ms"

|rex "基金 (?<fundcode>\w*) 支付状态: (?<paystatus>[^,]*),返回码:(?<payreturncode>\d*) ,消息: (?<payretrunmsg>[^,]*), 耗时: (?<payspentime>\w*) ms"

|fields - _raw

|fields + fundcode shijibuy totalbuy traceid fundcode buystatus buyreturncode buyretrunmsg buyspentime paystatus payreturncode payretrunmsg payspentime userid

|stats values(*) as * max(_time) as _time by traceid

|eval vresult=case(buyretrunmsg=="购买成功!" AND payretrunmsg=="支付成功" AND shijibuy==totalbuy,"足额购买成功",buyretrunmsg=="购买成功!" AND payretrunmsg=="支付成功" AND shijibuy!=totalbuy,"部分购买成功",buyretrunmsg=="购买成功!" AND payretrunmsg!="支付成功","购买成功支付失败",buyretrunmsg!="购买成功!","购买失败")

| timechart span=1d sum(shijibuy) as 实际购买金额 cont=false
```

##### 上面的知识点

```
max_match
语法：max_match=<int>
描述：控制正则表达式的匹配次数。如果大于 1，则生成的字段为多值字段。
默认值：1，值 0 表示无限制。
```

## 网络教程

### 基础搜索语言

#### 解释： \#来源logs.zip 索引为：tutorialdata 源类型为：通用访问日志 搜索日志中IP为：127.0.0.1 关键字包括select 和 sleep

```
source="logs.zip:*" index="tutorialdata" sourcetype=access_common clientip="127.0.0.1" select sleep
```

![](http://image.3001.net/images/20161214/14817236108359.png)

#### \(select OR union\) 逻辑或。满足一个即可。 关键字OR要大写

```
source="logs.zip:*" index="tutorialdata" (script OR select)

source="logs.zip:" index="tutorialdata" sele
```

### Splunk的搜索语言\(head&tail\)

#### 管道运算符\(\|\)，将管道左边搜索产生的结果作为右边的输入 head, 返回前n 个（离现在时间最近的）结果 tail, 返回后n 个\(离现在时间最后的\)结果

```
index="tutorialdata" sourcetype="access_common" select | head 2
```

![](http://image.3001.net/images/20161214/14817236419831.png)

### Splunk的搜索语言\(top、rare、rename as \)

_top, 显示字段最常见/出现次数最多的值  
rare, 显示字段出现次数最少的值  
limit，限制查询，如：limit 5，限制结果的前5条  
rename xx as zz : 为xx字段设置别名为zz,多个之间用 ，隔开  
fields ：保留或删除搜索结果中的字段。fiels – xx 删除xx字段，保留则不需要 – 符号_

#### 获取出现次数最多的IP，降序排列

```
source="tutorialdata.zip:*" index="tutorialdata" | top clientip
```

![](http://image.3001.net/images/20161214/1481723737559.png)

#### 在上方结果中限制显示前5条

```
source="tutorialdata.zip:*" index="tutorialdata" | top clientip limit=5
```

![](http://image.3001.net/images/20161214/148172374734.png)

#### 为两个字段设置别名

```
source="tutorialdata.zip:*" index="tutorialdata" | top clientip |rename clientip as “攻击源” |rename count as "攻击次数"
```

#### ![](http://image.3001.net/images/20161214/14817237571877.png)

#### 删除最后一个percent百分比字段

```
source="tutorialdata.zip:*" index="tutorialdata" | top clientip|fields clientip count |rename clientip as “攻击源” |rename count as "攻击次数"  
 或者： 
source="tutorialdata.zip:*" index="tutorialdata" | top clientip|fields - percent |rename clientip as “攻击源” |rename count as "攻击次数" | fields
```

![](http://image.3001.net/images/20161214/14817237838432.png)

#### 可以保存为饼状图的仪表盘

![](http://image.3001.net/images/20161214/14817237958177.png)

#### 返回clientip最少的10个，升序排序

```
source="tutorialdata.zip:*" index="tutorialdata" | rare clientip
```

![](http://image.3001.net/images/20161214/14817238094190.png)

### Splunk的搜索语言\(table,sort\)

#### table :返回仅由参数中指定的字段所形成的表。

_如：table \_time，clientip，返回的列表中只有这两个字段,多个字段用逗号隔开基于某个字段排序（升序、降序\)，降序的字段前面要使用-号，升序的使用+号.sort -clientip, +status, 先基于clientip降序排列之后，再对这个结果基于status升序_

```
source="tutorialdata.zip:*" index="tutorialdata" host="www1" | table _time,clientip,status
```

![](http://image.3001.net/images/20161214/14817238249508.png)

#### 针对上述中先基于clientip降序排列之后，再对这个结果基于status升序

```
source="tutorialdata.zip:*" index="tutorialdata" host="www1" | table _time,clientip,status|sort -clientip,+status
```

![](http://image.3001.net/images/20161214/14817238396881.png)

### Splunk的搜索语言\(stats）

#### 对满足条件的事件进行统计

_stats count\(\) ：括号中可以插入字段，主要作用对事件进行计数  
stats dc\(\)：distinct count，去重之后对唯一值进行统计  
stats values\(\)，去重复后列出括号中的字段内容  
stats list\(\)，未去重之后列出括号指定字段的内容  
stats avg\(\)，求平均值_

```
source="tutorialdata.zip:*" index="tutorialdata" host="www1"|stats count(clientip)
 [统计clientip数量]

图片 http://image.3001.net/images/20161214/14817238617654.png

index="tutorialdata" sourcetype="access_combined_wcookie" status=200 "action=purchase" |stats dc(clientip) 
[dc去重复之后再进行统计]

图片 http://image.3001.net/images/20161214/14817242495436.png

可视化可以使用“径向仪表”，对满足一定数量进行不同颜色标记，可存为现有的仪表盘面板。

图片 http://image.3001.net/images/20161214/14817242599180.png

index="tutorialdata" sourcetype="access_combined_wcookie" |stats values(host) as "主机列表" 
[去除重复后列出字段的内容]

图片 http://image.3001.net/images/20161214/14817242662553.png

 index="tutorialdata" sourcetype="access_combined_wcookie" status=200 "action=purchase" |stats list(host) 
[未去除重复列出括号中的内容]

图片 http://image.3001.net/images/20161216/14818732606406.png
```

### Splunk的搜索语言\(chart\)

#### 在用于制作图表的表格输出中返回结果

```
chart count():

index="tutorialdata" sourcetype="access_combined_wcookie" status=200 "action=purchase" | chart count by host
[统计字段status=200以及action=purchase的事件，并且以host字段来进行排列显示]

图片

图片

chart max()

index="tutorialdata" sourcetype="access_combined_wcookie" status=200 "action=purchase" | chart count by host|chart max(count)
 [求出最大值]

图片

chart min()

index="tutorialdata" sourcetype="access_combined_wcookie" status=200 "action=purchase" | chart count by host|chart min(count) 
 [求出最小值]

图片

chart avg()

index="tutorialdata" sourcetype="access_combined_wcookie" status=200 "action=purchase" | chart count by host|chart avg(count)
[根据第一次的结果求出平均值]

图片
```



