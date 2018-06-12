## 时间类

```
明天的日期
LocalDateTime.now().plusDays(1).format(DateTimeFormatter.ofPattern(DateUtil.YYYYMMDD))
```

## 过滤+聚合

```
//筛选出触发止损的用户
List<FundCastsurelySingleForAutoInvestDTO> fcsbList = List.stream().filter(l -> {
    BigDecimal fundMarketValue = new BigDecimal(l.getFundMarketValue());
    BigDecimal costMoney = new BigDecimal(l.getCostMoney());
    BigDecimal stopLoss = l.getStopLoss();
    return (fundMarketValue.subtract(costMoney).divide(costMoney).compareTo(stopLoss) <= 0);
}).collect(Collectors.toList());
```



