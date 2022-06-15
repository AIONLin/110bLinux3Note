# 2022/06/01 week16 


[參考資料](https://tw511.com/a/01/32123.html)

*  ## user跟group 的用法  

到第一台(ansible端)(我的centos7-2)
```
ansible-doc user
```
假設新增使用者:  (之前新增過tom
```
ansible server1 -m command -a "id tom" 
```
所以改成mary : 先確認沒有mary
```
ansible server1 -m command -a "id mary" 
```
新增: uid最好大於一千
```
ansible server1 -m user -a "name=mary uid=1100 state=present" 
```

確認:
```
ansible server1 -m command -a "id mary"
```
刪除
```
ansible server1 -m user -a "name=mary state=absent"
```
確認:
```
ansible server1 -m command -a "id mary"
```
新增群組
```
ansible server1 -m group -a "name=manager 
state=present"
```
把mary加入MANAGER group
```
ansible server1 -m user -a "name=mary state=present group=manager"
```
確認:
```
ansible server1 -m command -a "id mary"
```



*  ## playbook的用法:   
之前的練習都是one job 現在要把all jobs 放在一起同時執行。


　　
實做：使用playbook 批次部署環境
 [參考資料](https://tw511.com/a/01/32123.html)+老師範例 
	
	files：存放需要同步到異地伺服器的原始碼檔案及組態檔；
	handlers：當服務的組態檔發生變化時需要進行的操作，比如：重新啟動服務，重新載入組態檔，handlers ['hændləz] 處理程式
	meta：角色定義，可留空；
	tasks：需要進行的執行的任務；
	templates：用於執行lamp安裝的模板檔案，一般為指令碼；
	vars：本次安裝定義的變數

	yum 安裝服務

	service 啟動

	copy 把網站拷貝過去

	在playbooks 中定義任務：
	name： task description #任務描述資訊
	module_name: module_args #需要使用的模組名字：

	我們可以在ansible伺服器上安裝LAMP環境，再將組態檔通過ansible拷貝到遠端主機上
  

<br>

*  ## 老師範例
```
我的環境 : servers包含第二台機器(我的Centos7-3)跟第三台機器(我的Centos7-4)
```


## 模擬在上面安裝 網頁伺服器 mariadb 等等

下載telegram的壓縮檔  
解壓縮後上傳到ansible那台機器(我的Centos7-2)  
用win scp 選sftp ( 使用user or root 登入都行 )  
丟到ansible那台機器(我的Centos7-2)   下的 /home/user  


* 講解  

yml很重視縮排跟空格  
相同縮排相同地位  
-是 list 在一起的一組
<br>

* 範例一 : 先對 server1 (我的Centos7-3) 安裝網頁伺服器

先到
playbook.yml
修改讓他內容剩下安裝跟啟動 

![](/110bLinux3Note/pics/pj1_httpd/ansible_server_pg1_httpd2-0.jpg)

到 server1 確認網頁伺服器有沒有裝  
```
rpm -qa  | grep http 
```
如果有就先移除   
```
rpm -e httpd
```
確認 port
```
netstat -tunlp | grep 80
```
回到 ansible 執行 playbook
```
ansible-playbook playbook.yml
```
```
curl http://centos7-3
```
![](/110bLinux3Note/pics/pj1_httpd/ansible_server_pg1_httpd1.jpg)  


<br>

回到 playbook 把 hosts 改 servers  
Centos7-3、Centos7-4 都會 run  
綠色是上次完成的  
changed 是這次新執行 (之前跑過沒截到圖)
![](/110bLinux3Note/pics/pj1_httpd/ansible_server_pg1_httpd2.jpg)




* 改變port :

將下面這段內容
```
name: configure httpd server
copy: src=./httpd.conf dest=/etc/httpd/conf/httpd.conf
```
補回去tasks裡面  

而接下來會用到的 httpd.conf 檔案在老師提供的
pj1 裡本來就存在  
如果沒有 : 
```
先在主控台下載httpd然後拷貝conf檔案到pj1就好
cp etc/httpd/conf/httpd.conf /home/user/ansible/project1
```

到 pj1 裡的 httpd.conf 檔案將 port80 改 8080  
![](/110bLinux3Note/pics/pj1_httpd_port/ansible_server_pg1_httpd_port1.jpg)
playbook hosts 改為 server1
![](/110bLinux3Note/pics/pj1_httpd_port/ansible_server_pg1_httpd_port2.jpg)


值得注意的是 單純
"安裝>配置檔拷貝過去>啟動"  
```
ansible-playbook playbool.yml
```
只有這樣 server1 是不會有變化的 

這時候用 ansible-doc service 查 state 狀態有哪些

將 playbool.yml 的 start 改成 restart 
 
port 也改成 8081 再重新執行 
```
ansible-playbook playbool.yml
```
即可
![](/110bLinux3Note/pics/pj1_httpd_port/ansible_server_pg1_httpd_port3.jpg)
### 注意  

有可能會一直修改配置檔案，如果伺服器已經有人在用，restart 會讓他斷線，因此不建議用


那如何解決配置檔案修改後的使用？ : 改成 reloaded 即可不影響使用者

最後配置檔案版本 :
- hosts: server1
  tasks:
    - name: install httpd server
      yum: name=httpd state=present

    - name: configure httpd server
      copy: src=./httpd.conf dest=/etc/httpd/conf/httpd.conf
      notify: reload httpd server

    - name: start httpd server
      service: name=httpd state=started enabled=yes

  handlers:
    - name: reload httpd server
      service: name=httpd state=reloaded

notify的用法:
```
加在要偵測的事情下
當他有做這件事情時，觸發另外一件事

notify 的內容跟 handlers 的 name 要完全一樣
```


然後回到 ansible 
```
ansible-playbook playbook.yml
curl http://192.168.56.105:剛剛改的port 
```
![](/110bLinux3Note/pics/pj1_httpd_port/ansible_server_pg1_httpd_port5.jpg)

到第二台機器 ( Centos7-3 )
```
 netstat -tunlp | grep 剛剛改的port號
```

![](/110bLinux3Note/pics/pj1_httpd_port/ansible_server_pg1_httpd_port6.jpg)


## part2   LINUX - FTP VSFTPD 安裝及設定教學

[參考](https://hoohoo.top/blog/linux-ftp-teaching/)

先在 ansible/ 下創資料夾
```
cd ansible/
ls
mkdir vsftpd-install
cd vsftpd-install
gedit playbook.yml vsftpd.conf
```

playbook 內容 : 只做安裝與啟動，其他先註解掉
```
- hosts: server1
  tasks:
    - name: install vsftpd server
      yum: name=vsftpd state=present

    #- name: configure vsftpd server
    #  copy: src=./vsftpd.conf dest=/etc/vsftpd/vsftpd.conf
    #  notify: reload vsftpd server

    - name: start vsftpd server
      service: name=vsftpd state=started enabled=yes

  #handlers:
  #  - name: reload vsftpd server
  #    service: name=vsftpd state=reloaded
```
![](/110bLinux3Note/pics/pj1_vsftp/ansible_server_pg1_vsftpd1.jpg)
```
ansible-playbook playbook.yml
```
測試 : 
```
yum install -y ftp
ftp centos7-3 (( 第二台
user進入
ls
```
看的到東西代表成功  
![](/110bLinux3Note/pics/pj1_vsftp/ansible_server_pg1_vsftpd3.jpg)


* ## 配置檔案 :
vsftp 配置檔案有點問題用老師的
```
# Example config file /etc/vsftpd/vsftpd.conf
#
# The default compiled in settings are fairly paranoid. This sample file
# loosens things up a bit, to make the ftp daemon more usable.
# Please see vsftpd.conf.5 for all compiled in defaults.
#
# READ THIS: This example file is NOT an exhaustive list of vsftpd options.
# Please read the vsftpd.conf.5 manual page to get a full idea of vsftpd's
# capabilities.
#
# Allow anonymous FTP? (Beware - allowed by default if you comment this out).
anonymous_enable=YES
#
# Uncomment this to allow local users to log in.
# When SELinux is enforcing check for SE bool ftp_home_dir
local_enable=YES
#
# Uncomment this to enable any form of FTP write command.
write_enable=YES
#
# Default umask for local users is 077. You may wish to change this to 022,
# if your users expect that (022 is used by most other ftpd's)
local_umask=022
#
# Uncomment this to allow the anonymous FTP user to upload files. This only
# has an effect if the above global write enable is activated. Also, you will
# obviously need to create a directory writable by the FTP user.
# When SELinux is enforcing check for SE bool allow_ftpd_anon_write, allow_ftpd_full_access
#anon_upload_enable=YES
#
# Uncomment this if you want the anonymous FTP user to be able to create
# new directories.
#anon_mkdir_write_enable=YES
#
# Activate directory messages - messages given to remote users when they
# go into a certain directory.
dirmessage_enable=YES
#
# Activate logging of uploads/downloads.
xferlog_enable=YES
#
# Make sure PORT transfer connections originate from port 20 (ftp-data).
connect_from_port_20=YES
#
# If you want, you can arrange for uploaded anonymous files to be owned by
# a different user. Note! Using "root" for uploaded files is not
# recommended!
#chown_uploads=YES
#chown_username=whoever
#
# You may override where the log file goes if you like. The default is shown
# below.
#xferlog_file=/var/log/xferlog
#
# If you want, you can have your log file in standard ftpd xferlog format.
# Note that the default log file location is /var/log/xferlog in this case.
xferlog_std_format=YES
#
# You may change the default value for timing out an idle session.
#idle_session_timeout=600
#
# You may change the default value for timing out a data connection.
#data_connection_timeout=120
#
# It is recommended that you define on your system a unique user which the
# ftp server can use as a totally isolated and unprivileged user.
#nopriv_user=ftpsecure
#
# Enable this and the server will recognise asynchronous ABOR requests. Not
# recommended for security (the code is non-trivial). Not enabling it,
# however, may confuse older FTP clients.
#async_abor_enable=YES
#
# By default the server will pretend to allow ASCII mode but in fact ignore
# the request. Turn on the below options to have the server actually do ASCII
# mangling on files when in ASCII mode. The vsftpd.conf(5) man page explains
# the behaviour when these options are disabled.
# Beware that on some FTP servers, ASCII support allows a denial of service
# attack (DoS) via the command "SIZE /big/file" in ASCII mode. vsftpd
# predicted this attack and has always been safe, reporting the size of the
# raw file.
# ASCII mangling is a horrible feature of the protocol.
#ascii_upload_enable=YES
#ascii_download_enable=YES
#
# You may fully customise the login banner string:
#ftpd_banner=Welcome to blah FTP service.
#
# You may specify a file of disallowed anonymous e-mail addresses. Apparently
# useful for combatting certain DoS attacks.
#deny_email_enable=YES
# (default follows)
#banned_email_file=/etc/vsftpd/banned_emails
#
# You may specify an explicit list of local users to chroot() to their home
# directory. If chroot_local_user is YES, then this list becomes a list of
# users to NOT chroot().
# (Warning! chroot'ing can be very dangerous. If using chroot, make sure that
# the user does not have write access to the top level directory within the
# chroot)
#chroot_local_user=YES
#chroot_list_enable=YES
# (default follows)
#chroot_list_file=/etc/vsftpd/chroot_list
#

# You may activate the "-R" option to the builtin ls. This is disabled by
# default to avoid remote users being able to cause excessive I/O on large
# sites. However, some broken FTP clients such as "ncftp" and "mirror" assume
# the presence of the "-R" option, so there is a strong case for enabling it.
#ls_recurse_enable=YES
#
# When "listen" directive is enabled, vsftpd runs in standalone mode and
# listens on IPv4 sockets. This directive cannot be used in conjunction
# with the listen_ipv6 directive.
listen=NO
#
# This directive enables listening on IPv6 sockets. By default, listening
# on the IPv6 "any" address (::) will accept connections from both IPv6
# and IPv4 clients. It is not necessary to listen on *both* IPv4 and IPv6
# sockets. If you want that (perhaps because you want to listen on specific
# addresses) then you must run two copies of vsftpd with two configuration
# files.
# Make sure, that one of the listen options is commented !!
listen_ipv6=YES

pam_service_name=vsftpd
userlist_enable=YES
tcp_wrappers=YES
```

記得將playbook 註解拿掉 改成: 
- hosts: server1
  tasks:
    - name: install vsftpd server
      yum: name=vsftpd state=present

    - name: configure vsftpd server
      copy: src=./vsftpd.conf dest=/etc/vsftpd/vsftpd.conf
      notify: restarted vsftpd server

    - name: start vsftpd server
      service: name=vsftpd state=started enabled=yes

  handlers:
    - name: restarted vsftpd server
      service: name=vsftpd state=restarted


![](/110bLinux3Note/pics/pj1_vsftp/ansible_server_pg1_vsftpd4.jpg)  

anonymous_enable 先 yes 測試再改回 no 測試匿名登入

```
anonymous_enable 為 YES 時:
ftp centos7-3
anonymous
abc@gmail.com
可以進去
```
![](/110bLinux3Note/pics/pj1_vsftp/ansible_server_pg1_vsftpd_anonymous_yes2.jpg) 

```
改成no
ansible-playbook playbook.yml
anonymous
abc@gmail.com
會進不去
```
![](/110bLinux3Note/pics/pj1_vsftp/ansible_server_pg1_vsftpd_anonymous_no1.jpg)

![](/110bLinux3Note/pics/pj1_vsftp/ansible_server_pg1_vsftpd_anonymous_no3.jpg)

<br>
配置檔其實很多註解，將註解拿掉 : 

```
cat vsftpd.conf | grep -v ^# | grep -v ^$  
```
```
-v就是不要 ^#代表以井字號為開頭的 ^$是去掉空白行
 ```
得到配置檔案精簡版本 : 
```
anonymous_enable=No
local_enable=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
xferlog_std_format=YES
listen=NO
listen_ipv6=YES

pam_service_name=vsftpd
userlist_enable=YES
tcp_wrappers=YES
```
![](/110bLinux3Note/pics/pj1_vsftp/ansible_server_pg1_vsftpd_anonymous_short.jpg)
精簡版也是改anonymous_enable=NO或YES就好

* 新增使用者

既然把 anonymous_enable=NO 代表要進去要有在系統裡  
接下來要新建帳號給新使用者用  
之前練習的新增帳號沒設定密碼，接下來會進行密碼設定　　

查看文件
```
ansible-doc user
```
設定密碼不能直接設定　　

示範　:　
```
ansible server1 -m user -a "name=tom1 password=tom1"

ssh tom1@centos7-3
```

![](/110bLinux3Note/pics/pj1_vsftp/ansible_server_pg1_vsftpd_adduserTOM1_1.jpg)




密碼設定不行明文  要經過 hash : 
用兩個大括號包起來 並用 sha512 加密 :
```
ansible server1 -m user -a "name=tom1 password={{'tom1'|password_hash('sha512')}}"
```
![](/110bLinux3Note/pics/pj1_vsftp/ansible_server_pg1_vsftpd_adduserTOM1_2.jpg)
這樣才能ssh tom1@centos7-3 第二台
 


然後去改 playbook   
改成mandy :
```
- hosts: server1
  tasks:
    - name: install vsftpd server
      yum: name=vsftpd state=present

    - name: add a new user "mandy"
      user:
        name: mandy
        password: "{{'mandy'|password_hash('sha512')}}"

    - name: configure vsftpd server
      copy: src=./vsftpd.conf dest=/etc/vsftpd/vsftpd.conf
      notify: restarted vsftpd server

    - name: start vsftpd server
      service: name=vsftpd state=started enabled=yes

  handlers:
    - name: restarted vsftpd server
      service: name=vsftpd state=restarted
```
![](/110bLinux3Note/pics/pj1_vsftp/ansible_server_pg1_vsftpd_adduserMandyYML.jpg)
```
ansible-playbook playbook.yml
```
![](/110bLinux3Note/pics/pj1_vsftp/ansible_server_pg1_vsftpd_adduserMandy_playbook.jpg)
```
ftp centos7-3
mandy
ls
```
![](/110bLinux3Note/pics/pj1_vsftp/ansible_server_pg1_vsftpd_adduserMandyOK.jpg)


離開用bye


* ## 變數
project2:
定義在playbook裡面的變數
![](/110bLinux3Note/pics/pj2/ansible_server_pj2_%E8%AE%8A%E6%95%B81.jpg)
```
ansible-playbook playbook.yml
```


![](/110bLinux3Note/pics/pj2/ansible_server_pj2_%E8%AE%8A%E6%95%B812.jpg)
這邊因為已經安裝過所以顯示OK，但可以看出 app1 跟 app2 都有被 httpd 跟 vsftp 取代

END










