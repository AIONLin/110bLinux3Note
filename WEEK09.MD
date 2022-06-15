## 2022/04/13 week 9 繼續 jump server

prometheus+ grafana  (ansible)  

啟動jump server

https://www.webdesigntooler.com/google-smtp-send-mail

https://myaccount.google.com/security

```
二次認證打開 
點產生一組密碼的
選取應用程式
選擇其他
jumpserver_test 產生一組密碼 
放到一個檔案裡 save
```

```
SMTP host：smtp.gmail.com
SMTP port：465
SMTP security mode：SSL/TLS

SMTP authentication：yes

SMTP account： smallko@email.nqu.edu.tw

SMTP password：[google應用程式密碼]

ssetting - > email 

subject prefix > MyJumpServer
```


回到console 
```
新增 mary
mary
gmail
user 
submit
ctrl + c
docker-compose down
docker rm -f 'docker ps -a -q'
```

[參考](https://tw511.com/a/01/11537.html)
```
grep sed awk
```

準備test.txt內容
```
aaa
bbbbb
AAAbbb
BBBASDABBBA
```
```
grep -A2 b test (-A2後面多兩行
```
grep -B1 b test (-B1前面多一行
```
grep -C1 b test (-C1前後各一行
```
```
grep -c1 b test (-c count
```
```
grep -c a test
```
```
grep -e AAA -e bbb test (-e 或 
```
```
grep -i b test ( -i 不管大小寫
```
```
grep -in b test (-n 顯示行號
-ni 或 -in 或 -n -i 或 -i -n 都一樣
```
```
grep -o ASDA test (-o 找到東西 只show他其它不顯示
```
```
grep -q aa test ( 不想看結果 只想看有無
```
```
echo $? (顯示上個指令成功與否
0是OK 其他是沒
```
```
grep -v aaa test (-v 不包含
```
```
grep -w aaa test (-w 匹配一個單字
```
```
gedit grep1.txt
AAA
```
```
grep -f grep1.txt test (-f 讀檔案
```
```
systemctl start httpd
curl 127.0.0.0
確認192.168.56.104 ( 確認存取

cat /var/log/httpd/access_log
哪台機器 中間兩個是- - 代表直接存取
200 狀態碼 
2是成功  
3跳轉 
4error 
5伺服器問題
```

```
cat access_log | awk '{print $1}'

cat access_log | awk '{print $1}' |sort |uniq
cat access_log | awk '{print $1}' |sort |uniq |wc -l2

cat access_log |  grep -o  "[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}"  | sort | uniq | grep -v "192.168.153.150"
```
```
cat 1.txt (放個1進去
grep [0-9] 1.txt
[0-9]{1,3} 代表 前面位數0到9都可以 後面代表 匹配一位數兩位數或三位數 都府和
echo 12 > 1.txt
grep "[0-9]\{1,3\}" 1.txt
先用"" 包起來 然後因為 {}是特殊符號 要跳脫 符號前面加上\
echo "345" > 1.txt
如果看到一長串不知道是三小 可以先把跳脫符號拿掉 方便看
看起來就是--> 數字.數字.數字.數字
cat access_log | grep -o "[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}" | sort | uniq | grep -v "192.168.153.150" | wc -l
```

簡單寫法
```
egrep  "[0-9]{1,3}" 1.txt (不用跳脫符號
```
```
cat access_log | egrep -o "[0-9]{1,3}.[0-9]{1,3}.[0-9]{1,3}.[0-9]{1,3}"
```

另外一範例 etc
```
grep -r -n "root" /etc  (處理資料夾就要加上 -r 代表遞迴
grep -r -n "root" /etc more
```

```
cat demo

sed "s/a/A/" demo

sed "s/a/A/g" demo

sed "s#a#A#g" demo

sed "s%a%A%g" demo 只是呈現被修改後的內容

cat demo  (原本的檔案不會被改 
如果想把修改過的改進去 加上一個-i

sed -i "s/a/A/g" demo

cat demo

sed -i s/SELINUX=enforcing/SELINUX=disable/g /etc/selinux/config
```

cat demo
```
sed "s/A/a/" demo
sed "3s/A/A/" demo

sed " /BBCC/ s/A/a/" demo
```


cat demo
```
AAA

bbbb

AABBCCDD
```
```
sed -i "s/^$/d" demo
sed -i "/^$/d" demo
```

