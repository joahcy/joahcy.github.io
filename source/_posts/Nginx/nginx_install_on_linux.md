---

title: Linux下的Nginx安装
date: 2017-06-01
categories: Nginx
tags: Nginx

---

## 安装依赖 ##

- PCRE  
	PCRE(Perl Compatible Regular Expressions)是一个Perl库，包括 perl 兼容的正则表达式库。
	nginx的http模块使用pcre来解析正则表达式，所以需要在linux上安装pcre库。
	`yum install -y pcre pcre-devel`
	***注：pcre-devel是使用pcre开发的一个二次开发库。nginx也需要此库。***

- zlib  
	zlib库提供了很多种压缩和解压缩的方式,nginx使用zlib对http包的内容进行gzip,所以需要在linux上安装zlib库。  
	`yum install -y zlib zlib-devel`
- openssl  
	OpenSSL 是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及SSL协议，并提供丰富的应用程序供测试或其它目的使用。
	nginx不仅支持http协议，还支持https（即在ssl协议上传输http），所以需要在linux安装openssl库。  
	`yum install -y openssl openssl-devel`

## 编译安装 ##
将nginx-1.8.0.tar.gz拷贝至linux服务器。
解压：  
`tar -zxvf nginx-1.8.0.tar.gz
cd nginx-1.8.0`
- configure配置
	./configure --help查询详细参数（参考本教程附录部分：nginx编译参数）
	参数设置如下：
	`./configure \
	--prefix=/usr/local/nginx \
	--pid-path=/var/run/nginx/nginx.pid \
	--lock-path=/var/lock/nginx.lock \
	--error-log-path=/var/log/nginx/error.log \
	--http-log-path=/var/log/nginx/access.log \
	--with-http_gzip_static_module \
	--http-client-body-temp-path=/var/temp/nginx/client \
	--http-proxy-temp-path=/var/temp/nginx/proxy \
	--http-fastcgi-temp-path=/var/temp/nginx/fastcgi \
	--http-uwsgi-temp-path=/var/temp/nginx/uwsgi \
	--http-scgi-temp-path=/var/temp/nginx/scgi
	`
	***注意：上边将临时文件目录指定为/var/temp/nginx，需要在/var下创建temp及nginx目录***
- 编译安装
	`make`
	`make  install`

## 启动nginx ##
`cd /usr/local/nginx/sbin/
./nginx`
- 查询nginx进程：
   ` ps -ef |grep nginx`
	***注意：执行./nginx启动nginx，这里可以-c指定加载的nginx配置文件，如下：
	./nginx -c /usr/local/nginx/conf/nginx.conf
	如果不指定-c，nginx在启动时默认加载conf/nginx.conf文件，此文件的地址也可以在编译安装nginx时指定./configure的参数（--conf-path= 指向配置文件（nginx.conf））***

## 停止nginx ##
- 方式1，快速停止：
	`cd /usr/local/nginx/sbin
	./nginx -s stop`
	此方式相当于先查出nginx进程id再使用kill命令强制杀掉进程。
- 方式2，完整停止(建议使用)：
	`cd /usr/local/nginx/sbin
	./nginx -s quit`
	此方式停止步骤是待nginx进程处理任务完毕进行停止。

## 重启nginx ##
- 方式1，先停止再启动（建议使用）：
	对nginx进行重启相当于先停止nginx再启动nginx，即先执行停止命令再执行启动命令。
	如下：
	`./nginx -s quit
	./nginx`
- 方式2，重新加载配置文件：
	当nginx的配置文件nginx.conf修改后，要想让配置生效需要重启nginx，使用-s reload不用先停止nginx再启动nginx即可将配置信息在nginx中生效，如下：
	`./nginx -s reload`

## 安装测试 ##
nginx安装成功，启动nginx，即可访问虚拟机上的nginx。
