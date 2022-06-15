## 2022/05/25 Linux3 week15

前置作業 : ansible ( 設定管理 ) 開環境

[繼續參考](ttps://tw511.com/a/01/32123.html) 

上禮拜學了 Ping 、 command 、shell

複習 : 
```
ansible servers -m ping 
ansible servers -m command -a "ifconfig"
ansible servers -m command -a "" | grep enp0s3
( 會失敗因為command不支援 複雜的符號指令
```
換成shell就有支援 
```
ansible servers -m shell -a "ifconfig | grep enp0s3"
ansible servers -m shell -a "ifconfig | grep -A 3 enp0s3"
( -A 代表after 看到enp0s3之後3行 )
```

預設模組是 command : 不寫出模組也行
```
ansible servers -a "ifocnfig" 
( 預設模組是 command )
```
執行後的顏色代表不同意思 

```
ansible-doc -l >>顯示有的模組
```

```
ansible-doc 模組名稱  >> 顯示模組用法
```

例如 :
```
ansible-doc shell  ((詳細
ansible-doc -s shell 簡單的顯示用法
```   

要控制的主機們在指令中前後順序都可 : 
```
ansible server1 -m command -a "uptime"
(uptime:開機時間(持續)
ansible -m command -a "uptime" 'server1' 效果同上
ansible -m command -a "uptime" server1 同上
```

看懂uptime格式 :
```
開機開多久 多少使用者 平均loading 一分鐘五分鐘十五分鐘 負擔XXX 
```


看記憶體狀態 : 
```
ansible -m command -a "free -m" server1
多少memory 用了多少 剩多少 
```

新增使用者 : 管理 
```
ansible server1 -m command -a "useradd tom" 
ansible server1 -m command -a "id tom"
ansible server1 -m command -a "id peter" 
看到紅色代表不存在
ansible server1 -m command -a "useradd peter"
ansible server1 -m command -a "id peter"
```

當不知道那些模組可使用時，command 是個好選擇，只是其他模組有更細緻功能

進階 : 
```
command雖然可以廣用但無法處理一些複雜的符號與指令:
ansible server1 -m command -a "echo tom > tom.txt" 
ansible server1 -m command -a "cat tom.txt"
ansible server1 -m command -a "pwd"
ansible server1 -m command -a "touch a"
```

所以執行有符號的指令時可以將模組改成 shell 
```
ansible server1 -m shell -a "echo tom > tom.txt"

ansible server1 -m shell -a "cat tom.txt"

ansible server1 -m command -a "ifconfig | grep -A 3 enp0s3"

ansible server1 -m shell -a "ifconfig | grep -A 3 enp0s3"
```
* ## 腳本 :
```
ls
mkdir test-ansible
cd test-ansible
gedit a.sh
```
```
#!/usr/bin/bash
date
hostname
```
```
echo "hellloooo"
然後
chmod +x a.sh
./a.sh 

上面確認在本地端可行後
ansible server1 -m script -a "./a.sh"

ansible server1 -m script -a "/root/test-ansible/a.sh" 
絕對路徑

ansible server2 -m script -a "/root/test-ansible/a.sh"
```

補充 : 
```
預設工作路徑是root :
ansible server1 -m command -a "pwd"
```

```
如果要切到tmp資料夾下執行其他動作 :
ansible server1 -m command -a "cd /tmp"  
就算cd 到 tmp 了之後執行 touch
ansible server1 -m command -a "touch abcd.txt"  
還是會放在root下
ansible server1 -m command -a "ls /tmp" 
他不會在tmp這裡出現
ansible server1 -m command -a "ls /root" 
要在root這裡才有
```

因為他們是兩個不同的 job，沒有關連性

正確做法 : 
```
ansible-doc -s command 先查文件 裡面有個chdir
ansible server1 -m command -a "chdir=/tmp ls" 
先改路徑 在執行後面指令
```

* ## 講解 creates、removes :
```
ansible server1 -m command -a "ls" 
假設裡面有a 這個檔案
ansible server1 -m command -a "creates=a ls" 
如果a 存在 不執行ls
ansible server1 -m command -a "creates=b ls" 
再來假設現在沒有b 執行ls
```
removes跟creates相反 : 如果存在執行後面指令
```
ansible server1 -m command -a "removes=a ls" 
a 存在 所以執行後面指令
ansible server1 -m command -a "removes=b ls" 
b 不存在 所以後面指令不執行
```

簡單來說
```
ansible server1 -m command -a "creates=b touch b" 
如果b 不存在就執行後面的touch b
ansible server1 -m command -a "removes=b rm b" 
如果b 存在 就執行
再執行一次
ansible server1 -m command -a "removes=b rm b"
的話因為b不存在了 會被skipped
```

可以放一起 : 
```
ansible server1 -m command -a "chdir=/tmp creates=b touch b"
先切資料夾 然判斷b存在否 然後決定是否執行touch (這邊creates=b是代表不存在就執行後面)

ansible server1 -m command -a "chdir=/tmp ls"
```
![]()



* ##　part2

先確認環境能 ssh 免密登入  
在 ansible那台機器做  
切換到　root　模式  
```
ssh-copy-id root@centos7-3  ( 第二台
ssh root@centos7-3 ( 第二台
```

開始 :  
* copy 模組
```
ansible-doc copy  
```
學會看文件 : 
```
文件裡的 OPTIONS 如果寫著 = 代表是必要的 mandatory   而 -號 可有可以沒有
  
文件的下面也會有範例可以看，雖然通常是給腳本用的，但仍可參考  
```


用copy當範例 : 
```
先在test-ansible資料夾下新增:
echo ""hello" > hello.txt  
cat hello.txt 

ansible server1 -m copy -a "src=./hello.txt dest=/tmp/hello.txt owner=user group=user mode=0644"  

ansible server1 -m command -a "ls -l /tmp/hello.txt"
```

思考時間 :  
Q : copy 如果原始檔案不存在還可以拷貝嗎 ?  
A : 可  
```
copy iniXXX(初始的英文) file:
ansible server1 -m copy -a "content='hello wwww' dest=/tmp/hi.txt"

ansible server1 command -a "cat /tmp/hi.txt"

結論:我們可以自己提供content放到dest裡面
```

補充 :
```
ansible server1 -m copy -a "content='hello wwww234' dest=/tmp/hi.txt backup=yes"  

加上backup=yes 會有另外一個檔案 原本的就不會被覆蓋
```


* ## file模組 :
```
用來新增刪減檔案或改變屬性，也包含了目錄的操作
ansible-doc file 
```
檔案或目錄都包含三種時間：  
```
access time(存取)  
modify time(內容變更)  
changed time(內容或屬性被改)
```

開兩個終端機好對比 : 
```
第二終端:
su
ansible server1 -m command -a "useradd tom"

選定一個檔案當範例:
ansible server1 -m command -a "ls -l /tmp/a.txt"

將owner改成tom:

ansible server1 -m file -a "path=/tmp/a.txt owner=tom group=tom mode=0755"

ansible server1 -m command -a "ls -l /tmp/a.txt"
```


創建目錄 :  
``` 
ansible server1 -m file -a "path=/tmp/testdir state=directory"
```
新增資料夾 : 
```
ansible server1 -m command -a "ls -l /tmp"
```
再次刪除 :
```
ansible server1 -m file -a "path=/tmp/testdir state=absent"

確認 :
ansible server1 -m command -a "ls -l /tmp"
```



	ansible server1 -m command -a "ls -l -d /tmp"


	ansible server1 -m file -a "path=/tmp/testdir state=absent" 刪除資料夾

	


新增檔案 :
```
ansible server1 -m file -a "path=/tmp/abcd.txt state=touch"

ansible server1 -m command -a "ls /tmp" 
```
![]()


* ## stat模組
用來查看檔案的屬性 : 
```
ansible-doc stat
```
例如 :   
在本地端 stat a.sh(檔案名稱) 可以看到檔案size 占了幾個區塊、Acess時間、modify時間、change時間、擁有者、擁有者群組、Inode等等  
必要的屬性是path  
```
ansible server1 -m stat -a "path=tom.txt"
```

yum模組 :  
```
用來安裝軟體，centos用在ubuntu也可
ansible-doc yum 
用來Install upgrade downgrades removes 跟list package
```
基本用法 : 
```
ansible server1 -m yum -a "name=httpd state=latest"

移除:absent  
ansible server1 -m yum -a "name=httpd state=absent"

確認是否移除 : 
ansible server1 -m shell -a "rpm -qa | grep httpd"

執行> 黃色
執行過 > 綠色
錯誤 > 紅色
```

變數比較適合用在劇本(腳本)

cron 跳過 主要用來新增定時任務


* ansible 用在 ubuntu 也行 : (本人沒有ubuntu因為硬碟空間不夠裝了)
```
ubuntu是用apt安裝
開啟ubuntu   
	確認ssh root@uunumtu可以 
    ( ssh-copy-id root@ubuntu那台
	會被擋 要先回 ubuntu那台然後:
    gedit /etc/ssh/sshd_conflg  
	找到permitRootLogin proXXXXX那行  
	下面那行加上permitRootLogin yes (因為有些系統不允許root遠端登入)
	之後確認:ssh root@ubuntu那台
```

回到主控台  :
```
cd到test-ansible將ubuntu加進去當server3
vim /etc/ansible/hosts
加上 :  
[server3]
ubuntu的IP
ansible server3 -m ping
```

然後去到ubuntu查軟體
```
(dpkg -l 很像 rpm -qa >>可以查詢系統有的軟體)
dpkg -l | grep gedit
```

回到主控台刪掉 gedit :
```
ansible server3 -m yum -a "name=gedit state=absent"
```

回到ubuntu看 :
```
dpkg -l | grep gedit
```

再回到主控新增 :
```
ansible server3 -m yum -a "name=gedit state=present"
```
回到ubuntu看 :
```
dpkg -l | grep gedit
```
結論 :
``` 
ansible 的yum模組針對不同平台都可安裝
```

* ## 啟動關閉軟體 service 
```
ansible-doc service
```

state 是最常用
```
配合started / stopped /restarted /reloaded
```
剛剛都是安裝未啟動
```
curl http://centos7-3  (第二台) 會失敗
```
啟動 :
``` 
ansible server1 -m service -a "name=httpd state=started"
```

看啟動情形 :
```
ansible server1 -m command -a "systemctl status httpd"

這時候  
curl http://centos7-3   (第二台) 就OK
```
![]()

補充 :
```
ansible_facts:

ansible執行前會有ansible_facts >　到被控端去收集資訊再去執行我們下的指令
```
```
setup模組:

跟ansible_facts類似動作，收集很多資訊回來
ansible server1 -m setup
ansible server1 -m setup | grep cpu
ansible server1 -m setup | grep mem

根據收集的資訊,讓機器有不同配置
```
END