## wirich 项目

```
CountDownLatch latch = new CountDownLatch(fcsbList.size());//两个工人的协作
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
    executorService.shutdown();
}
```



