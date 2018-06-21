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

1:

```
CountDownLatch latch = new CountDownLatch(fundList.size());
ExecutorService exs = Executors.newWorkStealingPool(cpuCount);
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

public static BigDecimal calc(GroupFundBuyInfoDTO groupFundBuyInfoDTO
        , Map<String, BigDecimal> fundAmountMap
        , CountDownLatch latch) {

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

2:

```
ExecutorService exs = Executors.newWorkStealingPool(cpuCount);
List<Callable<BigDecimal>> callables = null;
for (GroupFundBuyInfoDTO g: fundList) {
    callables = Arrays.asList(() -> calc(g, fundAmountMap, latch));
}

try {
    List<BigDecimal> collect = new ArrayList<>();
    for (Future<BigDecimal> v : exs.invokeAll(callables)) {
        collect.add(v.get());
    }
} catch (InterruptedException e) {
    e.printStackTrace();
}
```

#### Callbale

Callbale也可以像runnbales一样提交给 executor services。但是callables的结果怎么办？因为`submit()`不会等待任务完成，executor service不能直接返回callable的结果。不过，executor 可以返回一个`Future`类型的结果，它可以用来在稍后某个时间取出实际的结果。

```
ExecutorService executor = Executors.newFixedThreadPool(1);
Future<Integer> future = executor.submit(task);

System.out.println("future done? " + future.isDone());

Integer result = future.get();

System.out.println("future done? " + future.isDone());
System.out.print("result: " + result);
```



