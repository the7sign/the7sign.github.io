---
layout: post
title:  "Shell Script Tip (1)"
date:   2016-10-14 00:00:00
categories: server
tags: centos shellscript
comments: true
---

쉘 스크립트로 배치를 일정 간격으로 돌리다 보면 하나의 프로세스가 돌아가는데 시간이 오래걸려 중복처리가 되는 경우가 발생하게 됩니다.

아래의 스크립트도 그 이유대문에 작성된 스크립트인데 참고해볼만 할듯하여 공유 드립니다.

``` bash
#!/bin/bash
# ShellScript Example

file="/root/batchflag.txt"  # 프로세스 진행여부를 판단할 파일
logfile="/root/log/push_batch/$(date '+%Y-%m-%d').log"
starttime=$(date '+%F %r')
echo "PUSH BATCH start time: $starttime" >> $logfile
if [ -f "$file" ]
then

    filedata=`cat $file`

    if [ $filedata = "0" ]      # 파일의 내용이 0이면 진행함
    then
        echo "1" > $file        # 진행하기전에 1로 내용을 변경함.
        # PHP PUSH EXECUTE
        /data/apps/php5.4/bin/php /data/www/push/remote_push_notification_send.php >> $logfile
        # PHP PUSH END
        echo "0" > $file        # 진행이 완료되면 0으로 변경함
        echo "job complete!" >> $logfile
    else
        echo "process now running!" >> $logfile
    fi
else
    echo "$file not found" >> $logfile
fi
endtime=$(date '+%F %r')

echo "PUSH BATCH end time: $endtime" >> $logfile
```
