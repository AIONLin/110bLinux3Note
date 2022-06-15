## 2022/04/27 Linux3 prometheus week11 

三台虛擬機

準備工作: 安裝監控軟體前
```
su
把機器好好命名
hostnamectl set-hostname XXXX
```
```
把ip綁定 ( 管理方便 ( 上課方便

ifconfig ( 先確認原本ip

wired setting ( 開始設定

ipv4 > manual

ip 255.255.255.0 192.168.157.2   8.8.8.8

每台都設定 ( 共3台 (記得設定完關一下網路再開

memory 2G + 2cpu
```

開始:
```
su
gedit /etc/hosts
改成等等要用的ip
ping centos7-X (互ping)
```
![]()
```
測試
root@centos7-3  
```

紀錄一下 My IP : 
```
7-2 : 192.168.56.104
7-3 : 105
7-4 : 106
```


急縮比
```
core (2M
   |
   |
-------
|     | (switch/router
|     |
-------
 |    | (2M
 |    |
user  user2
```
電信公司到府時 可以問急縮比(白天晚上等等社區用電量是不一樣的
```
PT test (滲透測試

堆疊 stacking
![]()

HA 高可用 high availability

cluster (集群
spf (single point failure

如果網路有迴圈 很容易有 broadcats stonm
stp (spanning tree protocol
ft  (容錯 fault tolerance
```

![]()

-------------------------------------------


node exporters
時間序列、資料庫  
TSDB (可以知道增長量 跟時間有關的資料庫   
grafana ( 視覺化 因為TSDB送到http server不太直觀


第一台是普羅米修斯 五個套件

第二三台是 node exporters

開始:
```
https://prometheus.io/download/
複製連結 ( linux
到第一台機器
cd Downloads/
連結上去 開始下載
[參考](https://www.codeleading.com/article/10525895320/)
```

下載完後
```
ls
sudo groupadd  創造個普羅專用的群組
sudo groupadd prometheus
sudo useradd -g prometheus -m -d /var/lib/prometheus -s /sbin/nologin prometheus

附加上去 指定家目錄  告訴她這是系統帳號無法logini
```

解開普羅米修斯:
```
mkdir -p /opt/module
解壓縮到tar xf prometheus-2.35.0.linux-amd64.tar.gz -C /opt/module       tar xf prometheus-2.4.2.linux-amd64.tar.gz -C /opt/module
cd opt/module
ls -l 
chown - R pro把擁有者擁有群組都變成普羅米修斯
名字改短 mv prometheus-2.35.0.linux-amd64/ prometheus
chown -R prometheus.prometheus prometheus
```

配置
```
cd /usr/lib/systemd/system
ls

補充:
man 不同情境有不同用法
man 8 httpd ( 關於網路架設
```
網頁伺服器執行檔  
```
選項 
對應指令
wantedby ( 有執行 不用成功
required ( 有執行 要成功
pwd ( 要在/system
gedit /usr/lib/systemd/system
```
```
systemctl start prometheus.service
```

補充 配置檔案
```
[Unit]
Description=Prometheus
After=network-online.target

[Service]
Type=simple
Restart=on-failure
ExecStart=/opt/module/prometheus/prometheus --config.file=/opt/module/prometheus/prometheus.yml --storage.tsdb.path=/opt/module/prometheus/data --storage.tsdb.retention=30d --web.enable-lifecycle --log.level=debug

[Install]
WantedBy=multi-user.targe
```

有修改檔案都 
```
systemctl daemon-reload
```
port 9090
到google 查看

![]()


* 下載node_exporter
```
cd /home/user/Downloads
```
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz
```
出現錯誤
```
Resolving github.com (github.com)... failed: Name or service not known.
```
```
ip route show 
```
看到    
```
	default via 192.168.56.2 dev enp0s8 proto static metric 100
	default via 10.0.2.2 dev enp0s3 proto dhcp metric 103
	10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 103
	192.168.2.0/24 dev enp0s10 proto kernel scope link src 192.168.2.2 metric 102
	192.168.56.0/24 dev enp0s8 proto kernel scope link src 192.168.56.105 metric 100
	192.168.122.0/24 dev virbr0 proto kernel scope link src 192.168.122.1
```
然後
```
ip route del default via 192.168.56.2
```
```
讓default via 10.0.2.2 dev enp0s3 proto dhcp metric 103 這個到第一個 (對外的
就可以了
```
```
tar xvfz node_exporter-1.3.1.linux-amd64.tar.gz
mv node_exporter-1.3.1 linux-amd64  node_exporter
ls 
```
```
mv node_exporter /opt/module/
cd /usr/lib/systemd/system
```
```
gedit node_exporter.service
```

配置檔案:
```
[Unit]
Description=node_exporter
Documentation=https://github.com/prometheus/node_exporter
After=network.target
 
[Service]
Type=simple
ExecStart=/opt/module/node_exporter/node_exporter
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

然後
```
systemctl daemon-reload
systemctl start node_exporter
systemctl status node_exporter
看到active代表成功
```
![]()


![]()

END

