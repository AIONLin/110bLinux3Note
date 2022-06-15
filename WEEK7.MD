## 2022/03/30 LINUX3 week 7 

[之後有時間在做 因為要跑很久](https://www.ltsplus.com/linux/rhel-centos-7-compile-kernel)

可以換的 kernel  https://www.kernel.org/
建議選取longterm的 穩定

wget 後
```
tar xvjf linux-4.19.237.tar.xz
```
```
cp -v /boot/config-3.10.0-1127.19.1.el7.x86.64 .config
```
```
看到*字號代表編進去核心(常用在編
m是模組化 媽糾   (不常用的編成m
空白代表不編譯

然後會跑很久 可以先去做別的
```

* [參考](https://github.com/jumpserver/Dockerfile)
![]()
```
server1~5 每台帳密都不一樣  
維護管理不方便  
登入server 後，做了哪些事情有audit(稽核)功能　　
責任歸屬處理方便
所以推薦　jumpserver
```
```
user  ------- jumpserver  --------  server
```
```
只要記得jump server 的帳密
```
jump server 規定一些規則 限制使用者(權限)做哪些事 (避免誤刪

管理是很多台同時，需要熟悉工具

------------------------------------------------------
4A：
```
1.認證
2.授權 
3.稽核 auditing
4.紀錄 accounting
```
```
jump server 安裝也要mysql redis 所以花時間
這邊簡化版 用docker(正式時候不會用，因為效能差)
```


-------------------------------------------------------------------------------------------
開始操作 [參考](https://docs.docker.com/compose/extends/) :

	docker-compose (額外安裝的套件 (做XXXX
	sudo chmod +x /usr/local/
	拉到快速部屬那邊

	git clone --depth=1 https://github.com/wojiushixiaobai/Dockerfile.git

	cd Dockerfile

	cp config_example.conf .env

	docker-compose -f docker-compose-redis.yml -f docker-compose-mariadb.yml -f docker-compose.yml up



part2

	如果遇到error for XX container
	docker-compose down

	docker ps -a
	
    docker rm -f XXX (jms
	
    gedit /home/user/Dockerfile
	
    DOCKER_SUBNET=172.20.0.0/16 改成這樣




[參考](https://docs.docker.com/compose/install/):
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```
[再來參考](https://github.com/jumpserver/Dockerfile)
```
git clone --depth=1 https://github.com/wojiushixiaobai/Dockerfile.git

cd Dockerfile

cp config_example.conf .env

vim .env     /////把它改成DOCKER_SUBNET=172.20.0.0/16

docker-compose -f docker-compose-redis.yml -f docker-compose-mariadb.yml -f docker-compose.yml up
```

error : 
```
ERROR: for celery  Container "8c16158f4adb" is unhealthy.

ERROR: for lion  Container "8c16158f4adb" is unhealthy.

ERROR: for web  Container "8c16158f4adb" is unhealthy.

ERROR: for koko  Container "8c16158f4adb" is unhealthy.
ERROR: Encountered errors while bringing up the project.
```
試試看
```
docker ps -a  確認P

docker rm -f $(docker ps -a -q) 刪掉所有ps

docker-compose down

docker-compose -f docker-compose-redis.yml -f docker-compose-mariadb.yml -f docker-compose.yml up  再來一次
```
























