##  2022/03/09 week4  linux3 Docker

aufs
```
docker的檔案管理系統 一層一層出去 只要系統更動 就會多出一層 每層紀錄跟下一層之間的差異
```

```
一個image(是一個file)跑出一個docker 新增刪除修改產生一個layer
變更的東西要儲存(新的鏡像建立 docker沒儲存的話 新layer會消失
```
```
在新建層才能讀寫
下面的就不能改變
修改也會產生新一層 使用者看下去就是改變過的 舊的被隱藏住
```
操作:
```
su
docker images
docker run -it ubuntu bash   ( i是interXXXX t-終端機 
```

```
ls

cd /home

ls

echo "hi" > hi.txt

cat hi.txt

exit  (沒儲存就離開)

docker run -it ubuntu bash

cd /home ls (會喪失所有東西)

echo "hi" > hi.txt 

cat hi.txt
要跳離docker才能儲存
執行身分@執行id

ctrl+p ctrl+q

docker ps (可知剛剛docker還在
```

```
docker commit -h
docker commit id的名字(打幾個字就好) ubuntu:v1
docker images
docker ps
docker attach id
exit
docker ps 
docker ps -a
docker rm id
docker ps -a
docker images
docker run -it ubuntu:v1 bash
cd /home
ls
cat hi.txt
```
```
docker run -it ubuntu bash
ping 8.8.8.8
docker pull busybox
docker images
docker run -it busybox
ping 8.8.8.8
ctrl c 
docker ps
docker attach
```
```
跑另外一個  終端 (相同dokcer
docker exec -it 他id sh
ifconfig
```
![]()

如果想rm docker
不能用 docker rm id  
要這樣:
```
docker stop 他id
docker ps 
docker ps -a
docker rm 他id
```
暴力美學(簡短步驟)
```
docker run -it busybox sh
```

開另一視窗
```
su
docker ps
docker rm - f 他id
docker ps -a
```


補充:
```
 一次刪掉全部 running docker + exit docker
docker run -it busybox sh
```
```
開三台docker

docker ps (確認幾台
docker ps -a -q (直接show 正在跑的docker id)
docker rm -f $(docker ps -a -q)
docker ps -as
```
![]()

docker好用的點之1 : 
```
跑一個網頁伺服器:
google docker hub
搜尋 httpd
往下拉可以看到使用說明
docker pull httpd
```

補充備份: 
```
把做好的鏡像 備份到docker hub
先註冊docker hub 帳號  
 	Docker hub (2022/03/09):
	aionlin
	a13181496529765@gmail.com
	dockerforlinux3
	
docker login
輸入帳密
login succeeded

docker 全名有三部分  儲存倉庫位置/帳號 /鏡像位置 image files name

docker images
docker tag ubuntu:v1 帳號/unbuntu:v1
docker images
docker push 帳號/ubuntu:v1
```

抓下來 
```
docker rmi -f 他id
docker images
docker pull 帳號/ubuntu:v1
docker images
```

```
netstat -tunlp | grep 80
```
```
docker run -d -p 8888:80 httpd 
(-d 讓他跑在背景 -p 代表port XX:ZZ XX放本地Port ZZ是docker在跑的port
```

```
docker run -d -p 8889:80 httpd
docker run -d -p 8890:80 httpd
docker ps
```
```
如果數量很大 寫腳本how
google 搜尋 bash for
            bash 運算
```

新終端機
```
su 
ls
which bash
gedit docker_httpd_setup5.sh
```
```
#!/usr/bin/bash

for i in{1..5};
do
    portno=`expr 9000 + $i`
    docker run -d -p $portno:80 httpd
done
```
```
ls
chmod +x docker_httpd_setup5.sh
./docker_httpd_setup5.sh
docker ps
```
```
gedit_docker_httpd_teardown5.sh
```
```
#!/usr/bin/bash

docker rm -f `docker ps -a -q`
#docker rm -f $(docker ps -a -q) 兩個一樣
chmod +x docker_httpd_teardown5.sh
./docker_httpd_teardown5.sh
```

改變網頁內容
```
最笨的:
docker ps
docker run -d -p 8888:80 httpd
docker ps
docker exec -it 他id bash
usr/local/apache2
ls
cd /htdocs
echo "hi" > hi.htm
ls
192.168.56.XXX:8888/hi.htm

mapping 把本地mapping到資料夾

exit
ls
mkdir web
cd web
echo "hi2" > hi2.htm
ls
cd ..
ls
docker run -d -p 8888:80  -v /home/user/web:/usr/local/apache2/htdocs httpd
docker exec -it XXX bash
cd htdocs
ls
```

開另一終端機 
```
su 
cd home
ls
echo "hello" > hello.htm
```


















