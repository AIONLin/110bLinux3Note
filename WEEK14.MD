2022/05/18 Linux3 prometheus week 14

開三台機器
```
su
systemctl start prometheus.service
systemctl start pushgateway
systemctl start docker
systemctl status docker
```

到第二台:
```
systemctl start httpd
接下來用docker跑 (版本
如果docker grafane 不行>
systemctl status grafana＞ps -ef | grafana＞netstat -tunlp | grep 3000＞pkill -9 grafana

跑起來  
普羅看狀態httpd_status_centos7_3
```

到第一台:
```
確認docker 啟動
docker pull grafana/grafana:7.5.15

docker run -d --name=grafana -p 3000:3000 grafana/grafana:7.5.15

(docker rm -f 要刪掉的id

3000 打開用admin登入  
admin
admin>123456　
```
設定過程:
```
setting data sources  > add data source
prometheus
http://192.168.56.104:9090

save and test 綠色OK 
右邊+號 create
add and empty panel
```
下滑
```
普羅米修斯
httpd_status_centos7_3 (第二台
調整時間
stop httpd 測試
記得restart
```

* ## 開始結合Inotify
https://notify-bot.line.me/zh_TW/

```
LINE 創群組  
把notify 加入群組  
發行權杖 選擇剛剛創的群組  
複製權杖  
切到網頁grafana  
```
```
左邊面板
鈴鐺 選 channels
type > line
給他名稱(NAME
貼上權杖
save
```


可以test line 會不會有訊息
![]()
```
回到grafane 面板
選擇edit ( telegram有截圖
設定
panel 下滑
```

設定 ![]()

save



開始講 ansible 自動化運作管理
參考 https://tw511.com/a/01/32123.html


開始實作 : 
```
第一台:
ssh root@centos7-3 (無密碼登入第二台 先確認可以

要做ssh認證 所以要先做免密登入
ssh-keygen
ssh-copy-id root@192.168.56.第二台
ssh root@192.168.56.第二台

之後exit 回到第一台
yum install epel-realease
yun install ansible
```

其他兩台
```
systemctl start sshd
```

* 做簡單配置
```
gedit /etc/ansible/hosts
```

![]()如下 : 
```
[server1]
192.168.56.105

[server2]
192.168.56.106

[servers]
192.168.56.105
192.168.56.106
```
然後儲存(server可以是web或DB等等)
```
測試:
ansible server1 -m ping
ansible server2 -m ping
ansible servers -m ping
```

直接下命令的模式 :
```
 ansible 指定對象 做甚麼事
ansible 很多模組,不同模組不同功能例如:安裝軟體、新增使用者、新增群組、建立目錄拷貝
```

* ## ping 模組
剛剛的指令是在測試是否連通 server1、server2  
  
如果把server1的介面卡關掉
```
ifconfig enp0s3 down 
(關掉第二台enp0s3介面卡)

ansible servers -m ping 
只會看到第三台回應
```
記得ifconfig enp0s3 up 打開回去

* ansible-doc
``` 
第一台:
ansible-doc -l 會顯示ansible模組與簡短說明
顯示ansible 有多少module與說明

ansible-doc -l | wc -l 查看有多少模組

ansible-doc -l | grep ping 
有可以查看各種作業系統與網路設備的ping
```
```
ansible-doc 模組  ( 可以看完整說明
例如 : 
ansible-doc ping  
```

* ## command
```
ansible server1 -m command -a "ifconfig"
( -a引數Argument)
ansible server1 -m command -a "date" 確認時間
ansible servers -m command -a "date"
```

到第三台調整時間讓不同步 :
```
date -s 20220101
```
如果時間不同步 ( 手動設定讓他時間不同(不同機器 :
```
ansible servers -m command -a "ntpdate time.stdtime.gov.tw"
```


查看其他台機器下的tmp資料夾 :
```
ansible servers -m command -a "ls /tmp"
cd /tmp && ls   (( &&代表前面執行成功才執行後面
cd /tmp ; ls  ((分號不管成功與否
ansible servers -m command -a "cd /tmp && ls"
會失敗 因為command模組無法支援&&
```

將command模組換成shell :
```
ansible servers -m shell -a "cd /tmp ; ls" 
就可以了
```

原因 : 
```
command不支援 shell支援 各有分別
```

END


[參考資料](https://tw511.com/a/01/32123.html)

ansible servers -m shell -a "cd /tmp ; ls"

ansible servers -m command -a "cd /tmp ; ls"



