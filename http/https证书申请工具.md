## acme.sh 工具使用

+ 1、安装acme.sh：
````bash
curl https://get.acme.sh | sh 
````
或者
````bash
curl https://get.acme.sh | sh -s email=my@example.com
````

+ 2、添加软链接：
````bash
ln -s  /root/.acme.sh/acme.sh /usr/local/bin/acme.sh
````

+ 3、切换CA机构： 
````bash
acme.sh --set-default-ca --server letsencrypt
````
+ 4、申请证书： 
````bash
acme.sh  --issue -d 你的域名 -k ec-256 --webroot  /var/www/html
````

+ 5、安装证书：
````bash
acme.sh --install-cert -d 你的域名 --ecc --key-file	/path/to/keyfile/server.key  --fullchain-file /path/to/crtfile/server.crt 
````

+ 6、自动为你创建 cronjob, 每天 0:00 点自动检测所有的证书, 如果快过期了, 需要更新, 则会自动更新证书. 需要cron组件

## [acme.sh 参考](https://github.com/acmesh-official/acme.sh/wiki/%E8%AF%B4%E6%98%8E)