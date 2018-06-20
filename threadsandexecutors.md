## wirich 项目

### 线程池处理无返回值\(newWorkStealingPool\(\) java8 最新\)

```
CountDownLatch latch = new CountDownLatch(fcsbList.size());//多个工人的协作
ExecutorService executorService = Executors.newWorkStealingPool(cpuCount);

//循环每一个然后分配线程去处理业务
fcsbList.forEach(fgbBean -> executorService.submit(new Thread(() -> {

            try {
                UserBankCardBean card = (UserBankCardBean) userBankCardSOAService.queryById(fgbBean.getDebitAccount());
                fundCastsurelySceneSOAService.changeAutoInvestPlan(fgbBean, currentSignFund, fundcode, sfyx, card.getChannelId(), card.getDepositAccount(), card.getTransactionAccountId(), card.getMoneyAccount());
            } catch (Exception e) {
                log.error("调用服务异常{} ,custNo:{}, appsheetserialno:{}", s, fgbBean.getUserId(), fgbBean.getAppsheetserialno(), e);
            } finally {
                latch.countDown();
            }

        }))
);

try {
    latch.await();//等待所有工人完成工作
} catch (InterruptedException e) {
    log.error("{} error~~", s, e);
} finally {
    //其实在使用Spring等框架来管理类的生命周期的条件下，也没有必要调用这两个方法来关闭线程池，
    //线程池的生命周期完全由该线程池所属的Spring管理的类决定
    executorService.shutdown(); 
}
```

### 线程池处理又返回值

```
CountDownLatch latch = new CountDownLatch(fundList.size());
ExecutorService exs = Executors.newFixedThreadPool(cpuCount);
List<BigDecimal> list = new ArrayList<>();//将每笔成功购买的金额存入list

fundList.forEach(groupFundBuyInfoDTO -> CompletableFuture.supplyAsync(() -> 
    calc(groupFundBuyInfoDTO, fundAmountMap, latch), exs).whenComplete((v, e) -> list.add(v)));

try {
    latch.await();
} catch (Exception e) {
    log.error("组合购买异步错误", e);
} finally {
    exs.shutdown();
}
return list;

public static BigDecimal calc(GroupFundBuyInfoDTO groupFundBuyInfoDTO, Map<String, BigDecimal> fundAmountMap
        , CountDownLatch latch, String custNo, String tradePassword, String moneyAccount, String transactionAccountId, String channelId, String merchantId, String groupCode, String groupName, String groupFundBuyId, String groupFundBuyInfoId
        , FundInfoSOAService fundInfoSOAService
        , SzKingWebService szKingWebService, TradeIdWorkerService tradeIdWorkerService
        , GroupFundBuySingleFundSOAService<GroupFundBuySingleFundBean> groupFundBuySingleFundSOAService
        , GroupFundMainInfoSOAService<GroupFundMainInfoBean> groupFundMainInfoSOAService
        , FundDividendService fundDividendService) {

    BigDecimal fundAmount = fundAmountMap.get(groupFundBuyInfoDTO.getFundId());
    try {
        // 业务逻辑
    } catch (Exception e) {
        log.error("组合购买异步错误", e);
    } finally {
        latch.countDown();
    }
    return fundAmount;
```



