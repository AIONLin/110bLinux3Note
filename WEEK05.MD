## 2022/03/16 LINUX3 week 5 

腳本產生
```
gedit prepare_web.sh
```
```
#!/usr/bin/bash

for i in {1..5}:                      
do
  mkdir -p myweb$i
  cd myweb$i
  echo $i > hi.htm
  cd ..

done
```
-p 用來判斷後面檔案 如果存在 跳過    
 
接著改變權限 :

```
chmod +x prepare_web.sh
```
```
./prepare_web.sh

ls
```

```
cd myweb3 
```

改起始腳本 ((( docker_httpd_setup5.sh
```
#!/usr/bin/bash

for i in {1..5}:
do
    portno=`expr 9000 + $i`
    docker run -d -p $portno:80 -v /home/user/myweb$i:/usr/local/apache2/htdocs httpd
done
```
```
ls
cd ..
ls
./docker_httpd_setup5.sh
```

``` 
docker ps -a               
```

如果有沒刪除乾淨的檔案 >> 
```
docker rm -f `docker ps -a -q`   
```
然後 重新
```
./docker_httpd_setup5.sh
```
```
docker ps
```
到google 看ip:9001/hi.htm

![]()

![]()

```
docker rm 容器   >>>
容器得是exit   
沒在執行的 除非加上 -f
docker commit 容器ID  新鏡像名稱:他的tag
```

運作過程 
```
image ---docker rum -->容器 ---- docker commit > new image
```
多補充
```
如果要產生新鏡像 用 docker commit 第二種 dockerfile(建議使用)類似寫腳本 可以不用手動去做
```

gedit Dockerfile  ((大小寫很重要 有規定
```
FROM centos:centos7   //通常是基礎

RUN -yum -y install httpd    // 自動安裝網頁伺服器

EXPOSE 80                       // 80port

ADD index.html /var/www/html      // 將本地端 index.html 檔案 丟到 docker 裡面 var/www/html


echo "hello index O_O" > index.html

ls

docker build -t myhttpd:1.0 .       // docker build 建立映像檔 t=tag    最後面的. 代表本地目錄


docker images 

docker run -d -p 8888:80 myhttpd:1.0 /usr/sbin/apachectl -DFOREGROUND
```
```
docker ps
```
ip:8888

![]()

## haproxy
>* 把每次的請求跟不同人要 一直按重新整理就會出先不同畫面

1. 
```
gedit haproxy.cfg
defaults
  mode http
  timeout client 10s
  timeout connect 5s
  timeout server 10s
  timeout http-request 10s

frontend myfrontend
  bind 0.0.0.0:8080
  default_backend myservers

backend myservers
  balance roundrobin
  server server1 192.168.48.171:9001
  server server2 192.168.48.171:9002
  server server3 192.168.48.171:9003
  server server4 192.168.48.171:9004
  server server5 192.168.48.171:9005
```
2. 
```
docker run -p 8080:8080  -d --name haproxy-master -v /home/user/haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg --privileged=true haproxy
```

3.
```
 接著打開5個server`./docker_httpd_setup.sh `    // 如果前面沒開的話
    

    [root@localhost user]# ./docker_httpd_setup.sh 
    4f89e2719097b67038c2bc499031982e48760c5e1eaa503c4128b3f05bacd390
    e8061ca913a448b9231ae3862579aa91ad2af2e237b4c93235449d1ebfc9b667
    8ef2dc4aa21ed53b4f0c192e16c0dc4b843303d6c49e8971a9894097f38a7a27
    ed705f5558e8b03c131802a140b4178056bae0f1af804d3b25c6967366bd0bed
    cf24adf4e34050b39966b7447b2e823c1dcb0cca6b8740a59ce2ad9a84dcf6bb
    
```

4. 
```
接著爬爬看，就可以使用了`curl 192.168.48.171:8080`
    

    [root@localhost user]# curl 192.168.48.171:8080
    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
    <html>
    <head>
    <title>Index of /</title>
    </head>
    <body>
    <h1>Index of /</h1>
    <ul><li><a href="hi.htm"> hi.htm</a></li>
    </ul>
    </body></html>
```

5. 
```
連連看網站，就可以看到load balance
```
![]()

```
vim test.py

print("hello python O_O ")

docker pull faucet/python3

docker run -it -v /home/user:/

mydata faucet/python3 bash

cd /mydata

python3 test.py
```

不用下載python到linux主系統 也可在 docker跑

![]()

* awk 過濾與篩選
```
ifconfig enp0s8 | grep netmask | awk '{print $2}'
ifconfig enp0s8 | grep netmask | awk '{print $0}'
```
```
ifconfig enp0s8 | awk '/netmask/ {print $2}'     
// 雙斜線進行篩選(匹配) 之後找出指定colum 
```
準備一個file 
```
vim file  
```
內容為:
```
Name Domain             // R 1
Deepak Banking          // R2
Neha Telecom
Vijay Finance
Guru Migration
```

接著
```
awk '{print $1}' file        等同於 cat file | awk '{print $1}'

awk 'NR==1 {print $1}' file  // 取出Name    // NR 代表  number of row
```

可以改成這樣取ip
```
ifconfig enp0s8 | awk 'NR==2 {print $2}'
```

取特定某一行
```
ifconfig enp0s8 | head -n 2
```

簡單作法
```
ifconfig enp0s8 | head -n 4 | tail -n 1    等同於  ifconfig enp0s8 | awk 'NR==4{print $0}'
```

不等於用法
```
awk 'NR!=1{print $1}' file         
```

只印有 Deepak 的
```
$ awk '$0 ~ /Deepak/{print}' file   等同於 awk '/Deepak/{print $0}' file
```

awk也可寫成腳本
```
有個格式如下:
	BEGIN{區
	塊
	{塊
	{區
	塊
	}二
	END{區
	塊
	}三
```
```
BEGIN是初始化
END 結束
各執行一次
區塊二就看檔案裡的row
BEGIN END 可以省略
最常用到區塊二
```

把file內容換成

```
11 dywang 81 12 A
152 linda 90 58 C
33 peter 72 95 C
4 rita 65 34 E
58 cora 5 85 D
```
```
awk 'BEGIN{sum=0} {sum+=$3;} END {printf("總合為:%d\n", sum)}' file
初始化變數sum=0 把col 3 數字總和 END時候 print出來
太長或太複雜 可以 編成.awk檔
```
```
vim process.awk
```
```
BEGIN{
  sum=0;
  i=0;
}

{
  sum+=$3;
  i++;
}

END{
  printf("總和=%d\n", sum);
  printf("平均=%.2f\n", sum/i);
}
```
```
awk -f process.awk file
``` 

另一範例 找出成績最高者
```
BEGIN{
  sum=0;
  max_name="";
}

{
  tmp_sum+=$3+$4;
  #printf("tmp_sum=%d sum=$d\n", tmp_sum, sum);

  if(tmp_sum > sum) {
  sum=tmp_sum;
  max_name=$2;
  }
}

END{
  printf("成績最高者為 : %s 分數為 : %d\n", max_name, sum);
}
```
```
awk -f process.awk file
```
```
awk 'BEGIN{sum=0} {sum+=$3;} END { printf("總和為:%d\n", sum)} '  file

```
如果是用逗號(非空白或tab)隔開 他會認為是一組
```
awk '{print $1}' file  會取出一整個

用分割符號 -F
awk -F, '{print $2}' file

cat /etc/passwd  在一堆東西取出使用者
awk -F: '{print $1}' /etc/passwd
```
END

