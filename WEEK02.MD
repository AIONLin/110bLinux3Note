2022/02/23 Linux3 week2

複習上禮拜的
http://[自己的ipv6]  
dynv6 註冊 申請domain name  
確定跟電腦(vb ifconfig)的ipv6位址一樣  
輸入網域進去   
才能繼續申請https

-----------------------------
200X開頭才是全球通用ipv6  
 別人才能連到自己的機器上  
如果沒有 就用 gedit ifcfg-enp0s3  
然後在最後加入 NETWORKING_IPV6=yes
IPV6_AUTOCONF=yes  
這兩行


https://intone.cc/2021/08/lets-encrypt-%E6%86%91%E8%AD%89-centos-7-apache-%E7%92%B0%E5%A2%83%E5%AE%89%E8%A3%9D-certbot%E5%B7%A5%E5%85%B7/

開始
```
yum install epel-release mod_ssl certbot
產生憑證
```
```
certbot certonly--webroot -w /var/www/html -d 自己網站名稱 --email a1318@gmail.com --agree-tos
```
看到恭喜就OK
```
cd /etc/letsencrypt/live/自己的網域/
ls
systemctl restart https
```
在https://自己的
可以進去但有警告標誌
要去掉變合法

```
gedit /etc/httpd/confd
SSLCertifica File  那邊
+1行 SSLCerXXx /etc/XX.pem
KeyFile /etc/XX /privkey.pem
第三個 .pem
```
然後再登入 就OK(不會有錯誤訊息
憑證有效期3個月


改換https://kkplay3c.net/ssl-for-free/
SSL for free 註冊帳號會比較好
