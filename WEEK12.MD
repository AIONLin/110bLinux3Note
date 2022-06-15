## 2022/05/04 Linux3 prometheus week12

node_exporter :
```
27 # gedit /opt/module/prometheus/prometheus.yml 
    scrape_configs:
    - job_name: "prometheus"
        static_configs:
        - targets: ["localhost:9090"]

    - job_name: "node_exporter1"
        static_configs:
        - targets: ["192.168.56.108:9100"]

    - job_name: "node_exporter2"
        static_configs:
        - targets: ["192.168.56.109:9100","192.168.56.110:9100"]
28 # systemctl daemon-reload
29 # systemctl restart prometheus.service
```
繼續上週prometheus內容
三台虛擬機開好

	systemctl start prometheus
	systemctl start node_exporter
	gedit prometheus.yml


把上週本地有效改成全局 (node 要先做好才能接著
```
第一台:
cd /opt/module/prX
vi prXX.yml
把localhost 改成 ip
cat /etc/hosts
改好啟動
systemctl start pr
還有啟動 node exporter
```

第二台:啟動node exporter

第三台啟動:node_exporter
```
netstat -tunlp 
```

開google 輸入ip:9090
到這裡 完成上禮拜最後才能接下去


* 到graph (網頁裡的)
來認識查詢指令(promql)


promql:
```
up   > 有啟用的
函式 instance(label標籤 用大括號刮起來

可以在查詢欄 輸入 up{ job="node_exporter1"}

或加個 ! 變成 up{ job!="node_exporter1"}
up{ instance="192.168.56.104"}

也可以正則表達式up{ instance=~"192.168.157.*"}

變成否定 up{ instance!~"192.168.157.*"}
```
普羅米修斯呈現簡單圖表


到第二台機器
```
關掉網路或關機
ifconfig enp0s3 down
回到網頁重新整理 觀看變化
![]()
記得開網路
ifconfig enp0s3 up
```
top 只呈現數字


回到網頁 node_load1 會取得目前最新附載值
例如:
```
node_load5 (取五分鐘
```
O_O


可以查詢一段時間的變化量或差異量
但無法用來做繪圖(只能當下)
```
up{filter}[5m]   5m>觀測時間點
up{}offestXXXXXX
up{}[5m]XXXXX
```

例如:
```
node_load1 {instance="192.168.56.104:9100"} [1m] 

node_load1{instance="192.168.56.104:9090"} offset 5m

node_load1{instance="192.168.56.104:9090"}[1m] offset 5m
```


[參考](https://yunlzheng.gitbook.io/prometheus-book/parti-prometheus-ji-chu/promql/prometheus-metrics-types)

![]()

Gauge可增可減

CPU模式
https://blog.csdn.net/qq_35753140/article/details/105121525


si > soft interrupt


繪圖(前五分鐘變化率可以繪圖
```
increase(node_cpu_seconds_total{mode="idle"}[5m])

sum(increase(node_cpu_seconds_total{mode="idle"}[5m])) by (instance)

(1- sum(increase(node_cpu_seconds_total{mode="idle"}[5m])) by (instance)/sum(increase(node_cpu_seconds_total[5m])) by (instance))*100
```


* MEMORY:

[參考](https://www.modb.pro/db/43636)
```
memory
 node_memory_MemTotal_bytes - 主機上的total

• node_memory_MemFree_bytes - 主機上的空閒內存

• node_memory_Buffers_bytes_bytes - 緩衝區中的内存

• node_memory_Cached_bytes_bytes - 頁面緩存中的内存

Total-free-buffers-cached = ing
```
結果如下 :
```
 (node_memory_MemTotal_bytes-(node_memory_MemFree_bytes+ node_memory_Cached_bytes + node_memory_Buffers_bytes)) / node_memory_MemTotal_bytes * 100
```

磁碟使用率:
```
(node_filesystem_size_bytes{mountpoint="/boot"} - node_filesystem_free_bytes{mountpoint="/boot"}) / node_filesystem_size_bytes{mountpoint="/"} * 100

跟在終端機用這個一樣 df -h | grep host
```

* 繪圖與警報  
[參考](http://max043.blogspot.com/2019/03/centos-7-grafana-602.html)

第一台
```
cd /etc/yumXXX
vi /etc/yum.repos.d/grafana.repo
```
```
[grafana]
name = grafana
baseurl = https://packages.grafana.com/oss/rpm
repo_gpgcheck = 1
enabled = 1
gpgcheck = 1
gpgkey = http://packages.grafana.com/gpg.key
sslverify = 1
sslcacert = /etc/pki/tls/certs/ca-bundle.crt
```

```
yum install grafana

systemctl daemon-reload

systemctl start grafana-server
status
```
可以進去網頁了 (網頁 : port 3000   忘記可以查一下)
```
netstat -tunlp | grep grafana
```
```
預設帳號密碼
admin
admin > 123456
```

回到網頁 
```
到官方網站 grafana.com
設定那邊data sources新增(add data source  普羅米修斯

點他 url那邊的localhost 打ip  ((http://192.168.56.104:9090

save & test

瀏覽器頁面 到官方網站 grafana.com
右上 product進去後

往下看到Other cool stuff > Dashboards
找到node_exporter 查出編號(Get this dashboard:) 1860
複製好回到剛剛葉面
creat >>>> + Import
輸入1860 然後按下load >>> error gateway timeout
改成回到第二分頁找到 node_exporter的json下載
然後再次回到第一個分頁 upload 剛剛der JSON
選擇來源 普羅米修斯
OK了
```

* 安裝pushgateway

裝到第一台機器
```
cd /home/user/Downloads

wget https://github.com/prometheus/pushgateway/releases/download/v1.4.2/pushgateway-1.4.2.linux-amd64.tar.gz

[root@centos7-2 Downloads]# tar xvfz pushgateway-1.4.2.linux-amd64.tar.gz
pushgateway-1.4.2.linux-amd64/
pushgateway-1.4.2.linux-amd64/LICENSE
pushgateway-1.4.2.linux-amd64/NOTICE
pushgateway-1.4.2.linux-amd64/pushgateway

[root@centos7-2 Downloads]# mv  pushgateway-1.4.2.linux-amd64 pushgateway

[root@centos7-2 Downloads]# mv pushgateway /opt/module/

[root@centos7-2 Downloads]# cd /opt/module/

[root@centos7-2 module]# ls
prometheus  pushgateway

[root@centos7-2 module]# cd pushgateway/
 
gedit /usr/lib/systemd/system/pushgateway.service
```

設定檔:
```
[Unit]
Description=pushgateway
After=network.target

[Service]
Type=simple
ExecStart=/opt/module/pushgateway/pushgateway 
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

設定完後 
```
daemon-reload 
```
```
systemctl start pushgateway
```

切換到普羅米修斯資料夾
```
gedit /opt/module/prometheus/prometheus.yml 
```
編輯設定檔
拉到最下面 
多個job name如下
```
- job_name: 'pushgateway'
    honor_labels: true
    static_configs:
      - targets: ['192.168.56.104:9091']   ( server ip + pushgateway port
```
```
systemctl restart pushgateway
```
回到普羅米修斯網站
```
(9090 選擇status > target
看到pushgateway代表OK
```
![]()


固定時間每15秒把資料抓回去

回到終端機:
```
echo "some_metric 87.8787" | curl --data-binary @- http://192.168.56.104:9091/metrics/job/some_job
```

回到普羅米修斯:
```
打some_metric 執行
值會跟著變
```
END







