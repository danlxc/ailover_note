<!-- toc -->


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
##### 事件监听
- 事件异常
Constant    Event class Triggered when...
EVENT_SCHEDULER_START   SchedulerEvent  The scheduler is started
EVENT_SCHEDULER_SHUTDOWN    SchedulerEvent  The scheduler is shut down
EVENT_JOBSTORE_ADDED    JobStoreEvent   A job store is added to the scheduler
EVENT_JOBSTORE_REMOVED  JobStoreEvent   A job store is removed from the scheduler
EVENT_JOBSTORE_JOB_ADDED    JobStoreEvent   A job is added to a job store
EVENT_JOBSTORE_JOB_REMOVED  JobStoreEvent   A job is removed from a job store
EVENT_JOB_EXECUTED  JobEvent    A job is executed successfully
EVENT_JOB_ERROR JobEvent    A job raised an exception during execution
EVENT_JOB_MISSED    JobEvent    A job’s execution time is missed

```
def my_listener(event):
# 如下为missed异常
    if event.exception:
        print('The job EVENT_JOB_MISSED :(')
        print('The job start again````For missed')
# 如下为正常值执行
    else :
        print('The job worked :)')

scheduler=BlockingScheduler()
scheduler.add_job(job,'cron',second='*/7', day_of_week='0-4', hour='9-23')
scheduler.add_listener(my_listener, events.EVENT_JOB_MISSED | events.EVENT_JOB_EXECUTED )
```
** **
##### 定时器apscheduler 完整代码
```
import datetime
import time
import logging
import apscheduler
import apscheduler.events as events.events as events
from apscheduler.schedulers.blocking import BlockingScheduler

isFlag = False
def job():
    start_time = datetime.now()
    print ('job start at', start_time)
    i = datetime.now()
    print ('a simple cron job start at', i)

    time.sleep(5)

    j = datetime.now()
    print ("The work took ", round((j-i).seconds / 60,2), 'minutes')

def my_listener(event):
    if event.exception:
        print('The job EVENT_JOB_MISSED :(')
        print('The job start again````For missed')

    else :
        print('The job worked :)')

scheduler=BlockingScheduler()
scheduler.add_job(job,'cron',second='*/7', day_of_week='0-4', hour='9-23')
scheduler.add_listener(my_listener, events.EVENT_JOB_MISSED | events.EVENT_JOB_EXECUTED )

try:
    scheduler.start()
# 如下表示当被键盘打断的时候,这再一次执行代码
except (KeyboardInterrupt, SystemExit):
    print('The job by KeyboardInterrupt :)')
    job()
```

#### 导入异常
```
try:
    import asyncio
except ImportError:
    import trollius as asyncio
```

#### 全局变量
```
global isFlag
isFlag = False
```

#### 定时器apscheduler 之 守护线程(两个线程同时进行)
```
        year (int|str) – 4-digit year
        month (int|str) – month (1-12)
        day (int|str) – day of the (1-31)
        week (int|str) – ISO week (1-53)
        day_of_week (int|str) – number or name of weekday (0-6 or mon,tue,wed,thu,fri,sat,sun)
        hour (int|str) – hour (0-23)
        minute (int|str) – minute (0-59)
        second (int|str) – second (0-59)

        start_date (datetime|str) – earliest possible date/time to trigger on (inclusive)
        end_date (datetime|str) – latest possible date/time to trigger on (inclusive)
        timezone (datetime.tzinfo|str) – time zone to use for the date/time calculations (defaults to scheduler timezone)

        *    any    Fire on every value
        */a    any    Fire every a values, starting from the minimum
        a-b    any    Fire on any value within the a-b range (a must be smaller than b)
        a-b/c    any    Fire every c values within the a-b range
        xth y    day    Fire on the x -th occurrence of weekday y within the month
        last x    day    Fire on the last occurrence of weekday x within the month
        last    day    Fire on the last day within the month
        x,y,z    any    Fire on any matching expression; can combine any number of any of the above expressions
```

```
# coding=utf-8
"""
Demonstrates how to use the background scheduler to schedule a job that executes on 3 second
intervals.
"""

from datetime import datetime
import time
import os

from apscheduler.schedulers.background import BackgroundScheduler


def tick():
    print('Tick! The time is: %s' % datetime.now())


if __name__ == '__main__':
    scheduler = BackgroundScheduler()
    #scheduler.add_job(tick, 'interval', seconds=3)
    #scheduler.add_job(tick, 'date', run_date='2016-02-14 15:01:05')
    scheduler.add_job(tick, 'cron', day_of_week='6', second='*/5')
    
    scheduler.start()    #这里的调度任务是独立的一个线程
    print('Press Ctrl+{0} to exit'.format('Break' if os.name == 'nt' else 'C'))

    try:
        # This is here to simulate application activity (which keeps the main thread alive).
        while True:
            time.sleep(2)    #其他任务是独立的线程执行
            print('sleep!')
    except (KeyboardInterrupt, SystemExit):
        # Not strictly necessary if daemonic mode is enabled but should be done if possible
        scheduler.shutdown()
        print('Exit The Job!')
```
##### 查看模块版本
```
module.__version__
 //例,查看flask模块的版本

>>> import flask
>>> flask.__version__
'0.10.1'

 //或者查看下模块下面的属性有没关于查看版本的
 dir(module)
```
#### python logging 日志

##### python logging 配置
```
log = logging.getLogger('apscheduler.executors.default')
    logging.basicConfig(level=logging.INFO,
                        format='[%(asctime)s] [%(levelname)s] [%(name)s:%(lineno)d] %(message)s',
                        datefmt='%Y-%m-%d %H:%M:%S',
                        filename='F:\OneDrive\python\loggerDemo.log',
                        filemode='a')  # w 删除之前的 重新写入 a 追加写入日志

    log.info('start to do it')
```

##### python logging 级别
```
logging.debug('debug message')
logging.info('info message')
logging.warn('warn message')
logging.error('error message')
logging.critical('critical message')
```

##### python 获取文件名路径
```
import sys
sys.argv[0]
```

##### job 工作时间计算
```
    end_time = datetime.now()
    logger.info('The work end')

    needTime = "The work took %0.2f minutes" % (
        (end_time - start_time).seconds / 60)
    logger.info(needTime)

except (IndexError, Exception) as e:
    logger.exception(e)
    logger.info('The work crash')
finally:
    cursor.close()
    db2.close()
    db.close()
```

