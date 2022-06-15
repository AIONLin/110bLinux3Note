2022/06/08 week17

ansible playbook 繼續
```
上禮拜介紹 變數
切到pj3
把變數分離到別的檔案
```
```
ansible playbook -e app1=

vars

ansible-playbook playbook.yml
```

![]()

繼續到pj4
```
cd ..
cd project4
ls
cd group_/
ls
mv server1 servers
ls
cat servers
cd ..
ls
tree
```
執行
```
ansible-playbook playbook.yml
```

補充:
```
範例二
playbook 有改變
tree
server1跟2 代表兩個不同群組
```


我問題: 
```
yum lockfile is held by another process 錯誤 
```
```
server1 OK 改成servers有錯誤
```



老師補充 pj5-1
```
要自己弄 先複製原本pj5
cp -r  project5 project5-1

內容改動
```

補充內容:
```
playbook.yml  新增host = servers

cd group_XX 
然後echo "app : curl" > servers
```


pj6: 變數也可以在 命令列執行
```
ansible-playbook -e host=server1 playbook.yml
ansible-playbook -e host=server2 playbook.yml
ansible-playbook -e host=servers playbook.yml
```

補充pj6: 
```
如果想改app1 但不想改檔案內容 (盡量不要
ansible-playbook -e host=server1 -e app1=vim playbook.yml
命令列權限>檔案
```
pj7:
```
多了httpd.conf跑在8082
netstat -tunlp | grep 8082
如果被占用再改
```

```
假設不知道process有沒有啟動
```
```
用XXX 然後將結果保存在變數
之後把變數叫出來
用模組
想看到的屬性
```

```
ansible-playbook playbook.yml
```
補充7-1: 自己改
```
多了memory total
cat /proc/meminfo | grep MneTotal | awk '{print $2}'
```

取得cpu 核心數
```
cat /proc/cpuinfo
```



```
cat /proc/cpuinfo | grep processor
cat /proc/cpuinfo | grep processor | wc -l
```

pj8: 
```
在配置檔案裏面 根據不同機器做配置  
先拷貝到j2  
變成模板
```
j2 === jinja2
```
ansible server1 -m setup
```
如果想得到ip
有個變數
```
ansible server1 -m setup | grep ipv4
ansible server1 -m setup | grep fqdn
ansible servers -m setup | grep fqdn
```

在conf.j2
```
不一樣
根據機器有所不同
拷貝改成template
```
```
playbook.yml 改成servers
```
到第二台跟第三台看一下:
```
cd /etc/httpd/conf
cat /etc/httpd/conf/httpd.conf | grep root
```
補充:
```
改了j2 
ansible-playbook playbook.yml
```

pj9:
```
memory catch

redis memcached

用原版memcached
```

補充
```
ssh如果不是跑在22
gedit /etc/ansible/hosts
在 ip後面加 ansible_ssh_port=XX
```