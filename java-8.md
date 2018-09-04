## 时间类

```
明天的日期
LocalDateTime.now().plusDays(1).format(DateTimeFormatter.ofPattern(DateUtil.YYYYMMDD))
```

## 过滤+聚合

```
// 查询所有用户
List<FundCastsurelySingleForAutoInvestDTO> List = fundCastsurelySingleSOAService.querySumCostmoneyByCustno();

//筛选出触发止损的用户
List<FundCastsurelySingleForAutoInvestDTO> fcsbList = List.stream().filter(l -> {
    BigDecimal fundMarketValue = new BigDecimal(l.getFundMarketValue());
    BigDecimal costMoney = new BigDecimal(l.getCostMoney());
    BigDecimal stopLoss = l.getStopLoss();
    return (fundMarketValue.subtract(costMoney).divide(costMoney).compareTo(stopLoss) <= 0);
}).collect(Collectors.toList());

List<SmartBetaProportionBeanBase> newBeanBaseList = beanBaseList.stream()
    .filter(ov->"smartbeta".equals(ov.getSmartbetaName()))
    .collect(Collectors.toList());
```

#### Collect

As you can see it's very simple to construct a list from the elements of a stream. Need a set instead of list - just use `Collectors.toSet().`

```
List<Person> filtered =
    persons
        .stream()
        .filter(p -> p.name.startsWith("P"))
        .collect(Collectors.toList()); //

System.out.println(filtered);    // [Peter, Pamela]
```

fdsf 

