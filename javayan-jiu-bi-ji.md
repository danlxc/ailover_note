#### redis 的动态代理对象,防止缓存击穿,缓存雪崩等...
```
@PostMapping("tacticsChart")
    public Object queryTacticsChart(@CurrentLocale Locale locale, @Validated FundChartRequest fundChartRequest) {

        RedisProxy redisProxy = new RedisProxy(tacticsIncomeSOAService);
        try {
            //redis 代理对象 获取收益走势图
            PrivateFundChartDTO privateFundChartDTO = (PrivateFundChartDTO) redisProxy.invoke("queryTacticsChart", RedisConst.TACTICS_CHART, fundChartRequest.getTacticsCode(), fundChartRequest.getStartType());
            if (privateFundChartDTO == null) {
                return this.genErrorResult("202002", locale);
            }
            return this.genSuccessResult(privateFundChartDTO, locale);
        } catch (Exception e) {
            return this.genErrorResult("202002", locale);
        }
    }


    package com.wirich.trade.service.proxy;


import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;

/**
 * @Author DannyLiu(全全)
 * @ClassName: redis 的动态代理对象,防止缓存击穿,缓存雪崩等...
 * @Description:
 * @Date: Created in 1:52 PM of 1/2/2018
 */
public class RedisProxy {
    private Object obj;
    private final static Logger log = LoggerFactory.getLogger(RedisProxy.class);

    @Autowired
    private RedisUtil redis;

    public RedisProxy(Object obj){
        this.obj = obj;
    }

    public Object invoke(String methodName, String... args ) throws Exception {

        String key = null;
        for (String arg : args) {
            key += arg+"_";
        }

        obj = redis.getObject(key.substring(0,args.length-1), obj);

        if (obj == null) {
            synchronized (this) {
                if (obj == null) {

                    obj = obj.getClass().getMethod(methodName,String.class).invoke(args);

                    if (obj == null) {
                        redis.saveObject(key, 60, obj.getClass().newInstance());
                    }

                    log.info("proxy调用方法{},并将key{}放入redis中", methodName ,key);

                    redis.saveObject(key, obj);
                }
            }
        }

        return obj;
    }
}


```

#### 防止重复请求
```
//如下是处理重复请求的代码
String firstRequest = redis.getFirstRequest(uniqueId, UserController.class);
//表示同时有两个请求过来
if (StringUtils.isBlank(firstRequest)) {
    return null;
}
//表示一个请求处理完成了,然后又收到一个相同的请求
if ("false".equalsIgnoreCase(firstRequest)) {
    return this.genErrorResult("990108", locale);//请求重复提交错误
}

@Override
    public String getFirstRequest(String uniqueId, Class<?> lockOfClass) {
        synchronized (lockOfClass) {
            Boolean isExists = exists(uniqueId);
            if (isExists && RedisConst.UNIQUEID_NOT_REQUEST.equals(get(uniqueId))) {
                setex(uniqueId, RedisConst.THIRTY_MINUTES_TO_SECONDS, RedisConst.UNIQUEID_ONE_REQUEST);
            } else if (isExists && RedisConst.UNIQUEID_ONE_REQUEST.equals(get(uniqueId))) {
                return null;
            } else {
                return "false";//请求重复提交错误
            }
        }
        return "true";
    }
```

#### 多线程获取返回值
```
//此为controller 也就是dubbo服务的消费者
//多线程获取返回值Demo
    @PostMapping("/testT")
    public Object tacticsTransferByHands(@CurrentLocale Locale locale,HttpSession session) {
        Long start = System.currentTimeMillis();

        //定长10list
        List<Integer> taskList = Lists.newArrayList(2, 1, 3, 4, 5, 6, 7, 8, 9, 11);
        //结果集
        List<String> list = fundGroupTransferSOAService.testT(taskList);

        System.out.println(taskList);
        System.out.println("list=" + list + "耗时 = " + (System.currentTimeMillis() - start));

        return this.genSuccessResult(locale);
    }

//此为service 也就是dubbo服务的服务者
    public List<String> testT(List<Integer> ll) throws WebServiceException {
        //此为获取cpu的核数量
        private static final int cpuCount = Runtime.getRuntime().availableProcessors();
        //此为线程池
        ExecutorService exs = Executors.newFixedThreadPool(cpuCount);

        List<String> list2 = new ArrayList<>();

        try {
            ll.forEach(j -> CompletableFuture.supplyAsync(() -> calc(j), exs).whenComplete((v, e) -> list2.add(v)));
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            exs.shutdown();
        }

//此方法是保证子线程都执行完毕了,再执行主线程
        while (true) {
            if (exs.isTerminated()) {
                break;
            }
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        return list2;
    }

    public static String calc(Integer i) {
        return (i += i);
    }
```

#### 线程中放入uuid  ThreadLocal
```
public class RpcContext {
    public static ThreadLocal<String> traceId = new ThreadLocal<String>(){
        @Override
        public String get() {
            if(null ==  super.get()){
                super.set(UUID.randomUUID().toString());
            }
            return super.get();
        }
    };
}
```

#### spring boot 单元测试 dao
```
@RunWith(SpringJUnit4ClassRunner.class) // SpringJUnit支持，由此引入Spring-Test框架支持！
// 指定我们SpringBoot工程的Application启动类
//  指定的话，就只会初始化指定的bean，速度快，推荐
//  这里指定的classes是可选的。如果不指定classes，则spring boot会启动整个spring容器，很慢（比如说会执行一些初始化，ApplicationRunner、CommandLineRunner等等）。不推荐
// @SpringBootTest(classes = {TradeServiceApplication.class, DataSourceWirichConfig.class, DataSourceAutoConfiguration.class, MybatisAutoConfiguration.class})
@SpringBootTest(classes = TradeServiceApplication.class)
@WebAppConfiguration // 由于是Web项目，Junit需要模拟ServletContext，因此我们需要给我们的测试类加上@WebAppConfiguration。
public class SmartBetaProportionSOAServiceImplTest {

    @Autowired
    private SmartBetaProportionSOAService smartBetaProportionSOAService;

    @Test
    public void addTransferInfo2() {
        smartBetaProportionSOAService.addTransferInfo();
    }

}
```