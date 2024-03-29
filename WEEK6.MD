## 2022/03/23 week6 LINUX3


老師的練習題 :
```
192.168.60.3  老師架了
用nmap掃過 nmpa -A 192.168.60.3 
root下有flag.txt
先進系統 在進去root

yum install nmap

yum install epel-release
```



上課 rsync + inotify

[參考](https://blog.gtwang.org/linux/rsync-local-remote-file-synchronization-commands/)

備份 、 rsync 應用
![]()
```
inotify   event creat/del/modify 偵測到 幫你備份
```

rsync簡介 
```
類似scp cp 
man rsync 看介紹與用法
```
![]()
```
rsync透過ssh 傳送 所以有加密傳送

yum install rsync

rpm -qa | grep rsync //看有沒有此軟體

rsync 參數 來源 目的

-v 詳細過程 verbose 自動化時就不需要這些 等有問題才有訊息

-r 遞迴 recursive 包含子資料結

-a 保留擁有者 權限等等

-z 節省頻寬

-h 少用
```

```
su 

echo "hi" > a.txt

rsync -avh a.txt /tmp
```
開另個視窗 然後 
```
cd /tmp
ls
cat a.txt
```

切回去第一個視窗
```
備份目錄

mkdir testdir  //rm -rf *

touch {1..10}.txt

ls

cd ..

rsync -avh testdit/ /tmp
```

去另個看
```
cd testdir/

echo "1" >1.txt

cd ..

rsync -avh testdir/ /tmp
```
可以看見差異式變更

原理是: 
```
mdsum5 (雜湊hash) 修改的話 再一次md5sum就會不一樣

cd testdir/

rsync 比較兩邊md5sum 把不同做備份

touch 11.txt 12.txt

cd ...

rsync -avh testdir/ /tmp
```

刪除:
```
cd testdir/

rm 4.txt 5.txt

rsync -avh testdir/ /tmp 
不會動 預設是不改動
要加參數告訴她

rsync -avh --delete testdir/ /tmp
```

* ## 備份
``` 
cd testdir

mkdir a/b/c -p  /// a>b>c  ///-p 如果沒a就幫建 沒b就幫 沒c就幫

tree
```
![]()

```

開新vb

cd .. 

cd tmp

mkdir backup
```

回到第一台 :
```
rsync -avzh testdir/ user@192.168.56.XXX:/tmp/backup/
```
要做ssh認證 所以要先做免密登入
```
ssh-keygen
ssh-copy-id user@192.168.56.XXX -f
ssh user@192.168.56.XXX

注意 ! user有時候可以依照需求改成root
```

```
 rsync -avzh testdir/ user@192.168.56.XXX:/tmp/backup/
```

如果遠端埠號有變，就改如下：
```
rsync -avzh -e "ssh" -p 22 testdir/ user@192.168.56.XXX:/tmp/backup/
```
```
cd testdir/

touch {a..c}.txt

tree

rsync -avzh --exclude 'a.txt' --exclude 'b.txt' testdit/ user@192.168.56.XXX:/tmp/backup/
```
也可以設定最大最小的備份檔案大小值
```
剛剛都是remote shell 現在要跑在伺服器　　
也可創vuser 跟系統使用者分開
```

產生配置黨放在etc下叫rsync.conf /////放在伺服器端
```
cd /etc

ls rsync.conf

gedit rsync.conf

uid=root   #使用者身分別

gid=root   #群組
身分別

pid file = /var/run/rsyncd.pid

log file = /var/log/rsyncd.log

secrets file = /etc/rsyncd.passwd     # 虛擬使用者 密碼放這裡
```
###############mod-相關參數與配置 整合
```
[mod1]    
   path = /backup1
   read only = no  #這樣才能write
   auth users = vuser   #針對此mod 只有 vuser可以用
```

虛擬機第二台
```
gedit /etc/rsync.passwd

vuser:user(密碼)

chmod 600 /etc/rsyncd.passwd
```
遠端與本地時間要同步
```
ntpdate watch.stdtime.gov.tw
```
兩台都做
date確認
7-6 (伺服器)
```
systemctl start rsyncd
cat /var/run/rsyncd.pid
cat /var/log/rsyncd.log

mkdir /backup1
```

7-5 (客戶端)
```
gedit /etc/rsync_vuser.passwd

 放密碼
user
chomd 600 /etc/rsync_vuser_passwd
ls
rsync -avz --progress --password-file=/etc/rsync_vuser.passwd testdir/ vuser@192.168.56.XXX::mod1
```












