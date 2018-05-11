    

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
  //如下为missed异常
    if event.exception:
        print('The job EVENT_JOB_MISSED :(')
        print('The job start again````For missed')
  //如下为正常值执行
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
  //如下表示当被键盘打断的时候,这再一次执行代码
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
// coding=utf-8
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
    '''
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
    '''
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
例,查看flask模块的版本

>>> import flask
>>> flask.__version__
'0.10.1'

或者查看下模块下面的属性有没关于查看版本的
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

##### 可变参数
```

```

