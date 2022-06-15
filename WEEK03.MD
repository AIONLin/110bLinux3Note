## 2022/03/02 LInux3 week3 
[參考資料](https://segmentfault.com/a/1190000018391069)

vm or vb: 負重多 heave weight 
docker  
隔離化技術 用區隔方式 比較輕量

what is 隔離的技術 
[參考這邊](https://segmentfault.com/a/1190000018391069)


網路空間的隔離  
隔離的空間就是網路
在不同空間 名字一樣OK 



```
ip netns add (( net name space
ip netne exec 在某命名空間幹嘛
範例是只在命名空間net1 看 ip addr
```
老師示範
```
ip netns add net1
ip netns ls
ip netns add net2
ls
ip netns del net2
ip netns ls
ip netns exec net1 if config
ip netns exec net1 ip addr
ip netns exec net1 if config -a
ip netns exec net1 ifconfig lo up
ip netns
```
![]()

介紹 ip link add type veth
```
ip metms add met0
ip link add type veth
ifconfig -a 才能看到veth0 veth1
ip link set dev veth0 netns net0
ip link set dev veth1 netns net1
```
```
ip netns exec net0 ifconfig -a
ip netns exec net1 ifconfig -a
```

```
把 net0、1 改成eth0
ip netns exec net0 ip link set dev veth0 name eth0
ip netns exec net1 ip link set dev veth1 name eth0
ip metms exec met0 if config -a
ip metms exec met1 if config -a
```

```
ip netns exec net0 ip addr add 10.0.0.1/24 brd + dev eth0
ip netne exec net0 ifconfig eth0 up 
ip netns exec net1 i[ addr add 10.0.1.0/24 brd + dev eth0
ip netne exec net1 ifconfig eth0 up

ip netns exec net1 ifconfig

ip netns exec net0 ping 10.0.0.2
```

開另個終端
```
su
ip netns exec net1 echo "hi" > hi.htm
ip netns exec net1 python -m SimpleHTTPServer 80
```

回到原視窗
```
ip netns exec net0 curl http://10.0.1.2/hi.htm
```

去到新開那個 
```
clear 
ls 
看到hi.htm
```

回到舊的 
```
ls
也可以看到--> 沒有檔案隔離
```


老師提供簡短指令
```
ip netns add net0


ip netns add net1


ip netns ls
 

ip link add type veth


ip link  set veth0 netns net0


ip link set veth1 netns net1


ip netns exec net0 ip link set dev veth0 name eth0


ip netns exec net1 ip link set dev veth1 name eth0


ip netns exec net0 ip addr add 10.0.1.1/24 brd + dev eth0   //// * 10.0.0.1


ip netns exec net1 ip addr add 10.0.1.2/24 brd + dev eth1 /////// *10.0.0.2 */eth0


ip netns exec net0 ifconfig eth0 up


ip netns exec net0 ifconfig eth1 up  *net1 * eth0


ip netns exec net0 ping 10.0.1.2


ip netns exec net1 echo "hi" > hi.htm


ip netns exec net1 python -m SimpleHTTPServer 80


ip netns exec net0 curl http://10.0.1.2/hi.htm
```

```
習題  O-----------O-----------O
       I         I I         I->10.0.2.1/24  
  10.0.1.1/24  10.0.1.2/24     
                 10.0.2.2/04   

 (第一台) O--要PING----------->10.0.2.1
  
```
* part2   
```
ip netns exec net0 bash
ifconfig
```
```
su - user 切回 本來畫面
su user
firefox 10.0.0.2/hi.htm
```
另個情境 三格不同網路空間 創一個bridge 存活在root space
像是原本的linux host only的網路


ip netns del net0-------> 說 在busy
ip netns exec net0 if config eth0 down ((關網路試試看 失敗
解決 晚點


ip netns add net4 、5、6
i[ netns ls
ip link add type veth
ip link add type veth
ip link add type veth
01 23 45 總共三對
135 (net4、5、6)
024 (bridge)
8'25
ip linl set dev veth netnsXXX
老師reboot他的


10'09
su
ip netns add net0
ip netns add net1
ip netns add net2
ip link add type veth
ip link add type veth
ip link add type veth
```
```
ip link set veth1 netns net0
3 1
5 2
ip netns exec net0 ip link set dev veth1 name eth0
ip netns exec net0 ip addr add 10.0.1.1/24 dev eth0
ip netns exec net0 ifconfig eth0 up  ((這啥小
ipnetns exec net0 ifconfig
ip netns exec net0 ip link set dev eth0 up

ip netns exec net1 ip link set dev veth3 name eth0
ip netns exec net1 ip addr add 10.0.1.2/24 dev eth0
ip netns exec net1 ip link set dev eth0 up

ip netns exec net2 ip link set dev veth5 name eth0
ip netns exec net2 ip addr add 10.0.1.3/24 dev eth0
ip netns exec net2 ip link set dev eth0 up
```

```
yum install bridge-utils
brctl -h
brctl addbr br0   創造 然後名稱-> br0
```

```
綁定介面卡
brctl addif br0 veth0 
brctl addif br0 veth2
brctl addif br0 veth4
brctl show br0
ifconfig br0 up
ifconfig
ifconfig veth0 up     # ip link set dev veth4 up
ifconfig veth2 up
ifconfig veth4 up
```

```
ip netns exec net0 ping 10.0.1.2
bridge 是第二層 通常沒有ip
但可以給
```
```
ip addr add 10.0.1.254/24 brd + dev br0
ifconfig br0
ip netns exec net0 ping 10.0.1.254
```
```

如果要刪除br

```
brctl delbr br0
ifconfig br0 down
brctl delbr br0
```


* 進入docker

image container 
```
有點像軟體 program process
存在hardd 執行時 進入XX 變成process
--> image就是一個檔案 跑起來變成container
```
倉庫 repository
```
存放image的地方 例如 docker hub
隔離的技術 
各種 namespace 與隔離內容
systemD process id = 1
```

docker 如何變成good
```
 ---> 隔離技術、cgroups 兩大悍將
```

* docker 的安裝
[參考](https://docs.docker.com/engine/install/centos/)

```
sudo yum install -y yum-utils

sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

sudo yum install docker-ce docker-ce-cli containerd.io

docker -v
```

連鏡像都沒有 先查詢有哪些
```
docker images
```
```
docker run hello-world 說unable to XXX
```

repo/name/image tag  才=完整鏡像名稱  
tag-版本 如果不寫tag 就是預設最新
```
docker run -it ubuntu echo "hi"
```