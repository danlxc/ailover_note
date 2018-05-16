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

#### 解释： #来源logs.zip 索引为：tutorialdata 源类型为：通用访问日志 搜索日志中IP为：127.0.0.1 关键字包括select 和 sleep
```
source="logs.zip:*" index="tutorialdata" sourcetype=access_common clientip="127.0.0.1" select sleep 
```
![](http://image.3001.net/images/20161214/14817236108359.png)


#### (select OR union) 逻辑或。满足一个即可。 关键字OR要大写
```
source="logs.zip:*" index="tutorialdata" (script OR select)

source="logs.zip:" index="tutorialdata" sele
```

### Splunk的搜索语言\(head&tail\)

#### 管道运算符(|)，将管道左边搜索产生的结果作为右边的输入 head, 返回前n 个（离现在时间最近的）结果 tail, 返回后n 个(离现在时间最后的)结果
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
![](http://image.3001.net/images/20161214/14817237571877.png)

```
source="tutorialdata.zip:*" index="tutorialdata" | top clientip|fields clientip count |rename clientip as “攻击源” |rename count as "攻击次数"  (删除最后一个percent百分比字段) 或者： 

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


## Splunk的搜索语言(table,sort)
#### table :返回仅由参数中指定的字段所形成的表。
*如：table _time，clientip，返回的列表中只有这两个字段,多个字段用逗号隔开基于某个字段排序（升序、降序)，降序的字段前面要使用-号，升序的使用+号.sort -clientip, +status, 先基于clientip降序排列之后，再对这个结果基于status升序*



