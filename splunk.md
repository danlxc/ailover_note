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

图片 http://image.3001.net/images/20161214/14817242832838.png

图片 http://image.3001.net/images/20161214/14817242961994.png

chart max()

index="tutorialdata" sourcetype="access_combined_wcookie" status=200 "action=purchase" | chart count by host|chart max(count)
 [求出最大值]

图片 http://image.3001.net/images/20161214/1481724306496.png

chart min()

index="tutorialdata" sourcetype="access_combined_wcookie" status=200 "action=purchase" | chart count by host|chart min(count)
 [求出最小值]

图片 http://image.3001.net/images/20161214/14817243154180.png

chart avg()

index="tutorialdata" sourcetype="access_combined_wcookie" status=200 "action=purchase" | chart count by host|chart avg(count)
[根据第一次的结果求出平均值]

图片 http://image.3001.net/images/20161214/14817243232072.png
```

### Splunk的搜索语言\(timechart\)

#### 使用相应的统计信息创建时间系列图表

```
index="tutorialdata" sourcetype="access_combined_wcookie" status=200 "action=purchase" | timechart count by host
[可以看到以每天作为时间分隔统计，在每24小时中满足条件的通过host字段进行统计]

图片 http://image.3001.net/images/20161214/14817243325870.png

index="tutorialdata" sourcetype="access_combined_wcookie" status=200 "action=purchase" | timechart span=8h count by host
[加入span参数来定义时间间隔为8h一次分隔统计]

图片 http://image.3001.net/images/20161214/14817243465759.png
```

### 子搜索\(\[search \]\)

#### 子搜索包含在方括号\[\]中

```
注:以下字段中含义：action=purchase代表成功购买产品 status表示状态为200

index="tutorialdata" sourcetype="access_combined_wcookie" status=200 "action=purchase" | top clientip limit=1
(搜索满足成功购买产品、状态为200的，出现数量最多的IP，只取最高的那个)

图片 http://image.3001.net/images/20161214/14817243631172.png

index="tutorialdata" sourcetype="access_combined_wcookie" "action=purchase" status=200 clientip="87.194.216.51"|stats count dc(productId),values(productId) by clientip
（搜成功购买，状态为200，IP为:87.194.216.51,统计购买产品的数量，并且去重复地列出具体的名称，最后通过clientip排序显示）

图片 http://image.3001.net/images/20161214/14817243724305.png

合并上面两个语句，子搜索放在[]中

index="tutorialdata" sourcetype="access_combined_wcookie" action="purchase" status=200 [search index="tutorialdata" sourcetype="access_combined_wcookie" status=200 action="purchase" | top clientip limit=1 |table clientip]|stats count dc(productId),values(productId) by clientip
(上面的clientip是通过子搜索 search 后面的结果，最后使用了“|table clientip”来只显示clientip字段，最后再进行如上次的统计数量和明细)

图片 http://image.3001.net/images/20161214/14817243908217.png

可视化后添加到仪表盘，可将现有仪表盘生成PDF。

图片  http://image.3001.net/images/20161214/14817244018045.png

还可以通过“PDF计划交付”来定时通过邮箱将报表发送给指定用户。

图片 http://image.3001.net/images/20161214/14817244162831.png
```

### eval命令和if函数 eval-对表达式进行计算并将结果存储在某个字段中

#### if \(条件,True的结果，False的结果\)

```
index="apachedata" sourcetype="access_combined_wcookie" | eval success=if(status==200,"成功","错误")| timechart count by sucess
解释：if函数判断status状态如果等于200则标记为成功字段，否则标记为错误字段，通过eval统计这些结果存储在sucess字段中，通过sucess字段排列，显示出成果与错误的数量

图片 http://image.3001.net/images/20161214/14817246605235.png

制作每一个主机的200、400和500事件数的对比图

200标记为“成功”，400标记为“客户端错误”，500标记为“服务器错误”,保存为column chart可视化图，另存现有仪表面板

index="apachedata" sourcetype="access_combined_wcookie" | chart count(eval(status==200)) as "成功", count(eval((400500 OR status==500)) as "服务器错误" by host
解释： 统计status状态码等于200的别名则为成功，状态码大于400或者等于400，并且状态码要小于500则为 客户端错误，状态码大于500或者等于500的则为服务器错误，最后通过host字段排列

58.png http://image.3001.net/images/20161214/14817248054795.png

61.png http://image.3001.net/images/20161214/14817249315735.png
```

#### 通过IP地址获取地区、国家、城市等信息

```
iplocation: 使用3rd-party数据库解析IP地址的位置信息

index="apachedata" sourcetype="access_combined_wcookie" | top 10 clientip|iplocation clientip
解释：获取前十的IP，并且对前十IP所在地区进行解析显示

图片 http://image.3001.net/images/20161214/1481725000233.png!small

来自中国的IP有多少

where:条件查询

index="apachedata" sourcetype="access_combined_wcookie"|iplocation clientip | where Country="China"|stats count by Country|rename Country as "国家"
```



