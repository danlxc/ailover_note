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
##### 定时器apscheduler
- coalesce：当由于某种原因导致某个job积攒了好几次没有实际运行（比如说系统挂了5分钟后恢复，有一个任务是每分钟跑一次的，按道理说这5分钟内本来是“计划”运行5次的，但实际没有执行），如果coalesce为True，下次这个job被submit给executor时，只会执行1次，也就是最后这次，如果为False，那么会执行5次（不一定，因为还有其他条件，看后面misfire_grace_time的解释）
- max_instance: 就是说同一个job同一时间最多有几个实例再跑，比如一个耗时10分钟的job，被指定每分钟运行1次，如果我们max_instance值为5，那么在第6~10分钟上，新的运行实例不会被执行，因为已经有5个实例在跑了
- misfire_grace_time：设想和上述coalesce类似的场景，如果一个job本来14:00有一次执行，但是由于某种原因没有被调度上，现在14:01了，这个14:00的运行实例被提交时，会检查它预订运行的时间和当下时间的差值（这里是1分钟），大于我们设置的30秒限制，那么这个运行实例不会被执行。

```
from apscheduler.schedulers.blocking import BlockingScheduler
    # 线程池 和 cpu使用数量
    executors = {
        'default': ThreadPoolExecutor(7),
        'processpool': ProcessPoolExecutor(2)
    }
    job_defaults = {
        'coalesce': True, # 错过了多少定时,是否重新跑多少次,还是只跑一次
        'max_instances': 7, # 最大的同时启动定时实例数量
        'misfire_grace_time': 1800 #(missed)错过多久可以再跑
    }
    jobstores = {
        #'mongo': MongoDBJobStore(collection='job', database='test', client=client), #
        'default': MemoryJobStore() #默认使用系统内存存储定时任务
    }
    scheduler=BlockingScheduler(jobstores=jobstores, executors=executors, job_defaults=job_defaults)
    # 'cron 表示不阻塞'
    scheduler.add_job(job,'cron',day_of_week='mon-fri',hour='9',minute="27")
    scheduler.add_listener(my_listener, events.EVENT_JOB_MISSED | events.EVENT_JOB_EXECUTED )
    print('Press Ctrl+{0} to restart'.format('Break' if os.name == 'nt' else 'C'))
    #print('如果成功了,请一定不要按Ctrl+c,如果missed,则按Ctrl+c重启任务')
    try:
        scheduler.start()
    except (KeyboardInterrupt):
        if isFlag:
            print('The job start again````By KeyboardInterrupt')
            start_again()
    except (SystemExit):
        scheduler.shutdown(wait=False)
        print('Have SystemExit so Exit The Job!')


```
** **