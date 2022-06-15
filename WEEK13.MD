## 2022/05/11 Linux3 prometheus week 13 

普羅米修斯跟grafana

自主監控系統狀態、通知管理者

[AIOPs](https://www.ibm.com/tw-zh/cloud/learn/aiops) [AIDevOps](https://www.ithome.com.tw/news/143584)


pushgateway : 把他們都start (普羅米、node、push
```
systemctl start prometheus.service
          status
systemctl start node_exporter.service
          status
systemctl start pushgateway
          status
```

確認
```
echo "some_metric 87.8787" | curl --data-binary @- http://192.168.56.104:9091/metrics/job/some_job
```
![]()


[參考資料](https://blog.51cto.com/root/3033785)

切換到第二台 (腳本放這 :
```
mkdir -p /app/scripts/pushgateway

cd /app/scripts/pushgateway
```

```
#!/bin/bash
# For waiting connections
label="count_netstat_wait_connetions"
count_netstat_wait_connetions=`netstat -an | grep -i wait | wc -l`
echo "$label:$count_netstat_wait_connetions"
echo "$label $count_netstat_wait_connetions" | curl --data-binary @- http://192.168.56.108:9091/metrics/job/pushgateway/instance/$instance_name
```
```
hostname -f 
-f 取出 判斷是否localhost
在此可以不寫
給他標籤說明 > 有沒有等待人家正要進來的連線
```
```
netstat -an | grep -i wait
```
```
netstat -an | grep -i wait wc -l   
((wc統計 l:有幾行
```

得出正在等待的筆數 丟到前面變數

操作 :
```
cat tcp_waiting_connections.sh

chmod +x 

tcp_waiting_connections.sh
```

回到普羅 : count_netstat_wait_connetions
![]()

回到第二台 : 
```
ls (剛剛創的pushgateway 裡面
cp 
打開另一個
label改一下
檢查網頁伺服器是否正常
yum install httpd
systemctl start httpd
netstat -tunlp | grep 80
```

* 第二個腳本內容

```
curl http://192.168.56.104  太多東西
curl http://192.168.56.104 > /dev/null  丟到 null
curl -q http://192.168.56.104 > /dev/null 2>&1
echo $?
	systemctl stop httpd
	curl http://192.168.56.104 > /dev/null
	echo $?
```

-改成_

```
完整:

#!/bin/bash

# For waiting connections
label="httpd_status_centos7_3"
curl -q  http://192.168.56.105 >/dev/null 2>&1
status=`echo $?`
if [ $status -eq 0 ];then
   result=1
else
   result=0
fi
#echo $label $result
echo "$label $result" | curl --data-binary @- http://192.168.56.104:9091/metrics/job/$label

```
```
systemctl start httpd
./check_httpd_centos第二台.sh
./check_httpd_centos7-3.sh
```

* 定期任務
```
pwd

crontab -e

* * * * * /app/scripts/pushgateway/check_httpd_centos7-3.sh 
* * * * * sleep 15;/app/scripts/pushgateway/check_httpd_centos7-3.sh
* * * * * sleep 30;/app/scripts/pushgateway/check_httpd_centos7-3.sh
* * * * * sleep 45;/app/scripts/pushgateway/check_httpd_centos7-3.sh

contab -l
```
![]()



回到grafana :
``` 
3000port
new dashboard > edit panel
設定完成後 save 
check_httpd_status_centos7-3
```

```
cat /etc/hosts
cat </etc/hosts
wc /etc/hosts
wc </etc/hosts
```

新指令 whatis :
```
whatis cat
whatis read
man 2 read 
man 1p read
read -p "what is ur name:" name
echo $name

cat或wc可以讀檔案
read只能讀標準輸入
```

多行文件:
```
cat <<EOF               //這個代表起始標籤
helo
wold
EOF                    //代表結束           
cat <<EOF
```

加大難度 
```
cat <<EOF > /tmp/a.txt
hey
yoooo
1224
EOF

cat /tmp/a.txt
```

[學會以上內容後你就可以看得懂這篇文章](https://blog.51cto.com/root/3033785)

範例 : 

這邊是一個字串，將它丟掉標準輸入 cat讀標準輸入　　
```
cat <<< "hi"
cat <<< "hi" > /tmp/a.txt
```

再複雜點　:
```
ls -l /proc/self/fd    
proc : 
self : 
fd   : 
```
ctrl+alt+f2
```

產生檔案:
```
touch {a..c}{1..10}.txt  擴展
```
echo "1" >1.txt
echo "2" >1.txt
echo "3" >1.txt
1.txt剩下3
echo "1" >>1.txt
echo "2" >>1.txt
echo "3" >>1.txt
```

介紹exec:
```
exec > 2.txt
echo "1"
echo "2"
echo "3"
exec > /dev/pts/1 
把她還原到終端模式
就可以cat 2.txt
```
Ｑ：究竟2.txt裡面會不會有東西呢?

A : 會

```
(echo "ho" ; echo $$)

看執行狀況:
ps -ef | grep prometheus
ps -ef | grep prometheus | grep -v grep 
ps -ef | grep [p]rometheus                (((((等等這三小
ps -ef | grep prometheu[s]     

ls | egrep "[ab]"2.txt
```
END

