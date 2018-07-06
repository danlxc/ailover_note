# python sh 脚本定时

    #!/bin/sh
    date1=`date +%Y-%m-%d`
    pfile="/www/strategy"
    logfile="/www/logs/strategy"
    maillist="guigenshen@5irich.com,liuquan@5irich.com"

    cd $pfile
    for prun in `ls $pfile/job*.py`
        do
        /usr/local/bin/python3 $prun >/dev/null
    #    /usr/local/bin/python3 $prun 2>>$logfile/error.log
    done
        /usr/local/bin/python3 $pfile/cj_incomecompute.py >/dev/null
    #    /usr/local/bin/python3 $pfile/cj_incomecompute.py 2>>$logfile/error.log

    det=`cat $logfile/error.log |grep $date1`
    if [  $? = 0 ]
            then
            /usr/local/bin/python3 $pfile/mail_ssl.py "$maillist" "Strategy Python is Error" "$det"
        else
            echo OK
    fi

    det=`cat $logfile/app.log |grep $date1 |grep 信号触发`
    if [  $? = 0 ];then
            cat $logfile/app.log |grep $date1 |grep 信号触发 >$pfile/app-$date1.log
            /usr/local/bin/python3 $pflie/mail_ssl.py "$maillist" "信号触发" "详细日志请看附件" "app-$date1.log"
        else
            echo OK >/dev/null
    fi

    det=`cat $logfile/app.log |grep $date1 |grep strategyTransfer接口调仓失败`
    if [ $? = 0 ];then
            cat $logfile/app.log |grep $date1 |grep strategyTransfer接口调仓失败 >$pfile/app-$date1.log
            /usr/local/bin/python3 $pfile/mail_ssl.py "$maillist" "strategyTransfer接口调仓失败" "app-$date1.log"
        else
            echo OK >/dev/null
    fi

## 运行sh脚本

```
sh python.sh
```



