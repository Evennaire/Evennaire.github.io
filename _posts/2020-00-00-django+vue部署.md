---
title: django+vue部署
author: Even
date: 2020-03-21 03:51:00 +0800
categories: [CS, Web]
tags: [web]
toc: false
---

参考1 基础配置 [https://www.cnblogs.com/Sunzz/p/10946346.html](https://www.cnblogs.com/Sunzz/p/10946346.html)
参考2 前后端分离 [https://blog.csdn.net/Coxhuang/article/details/103413212](https://blog.csdn.net/Coxhuang/article/details/103413212)
前提：本地vue+django已经调通，可以跨域访问

# 1. 修改django代码
+ 修改settings.py
	- 修改: DEBUG = False
	- 修改：ALLOWED_HOSTS = ['*']
	- 添加：STATIC_ROOT = os.path.join(BASE_DIR, 'static_root/')
+ pip freeze > requirements.txt
	慎用！执行后原来自己写的依赖库被覆盖了，原来的模块似乎丢了
	可以先复制，之后手动加进去
	比如django和django-cors-headers这两个最重要的模块。。
+ python manage.py collectstatic
	静态文件会被收集到static_root/文件夹下


# 2. 修改前端vue代码
axios的baseURL设置成/api/
dev的时候在config/index.js里/api代理成127.0.0.1:8000;
生产环境的时候就由nginx反代成后端接口，具体见最末的nginx.conf配置



# 3. linux环境
#### >> 后端
1. 安装py3 [https://www.jianshu.com/p/7c2b62c37223](https://www.jianshu.com/p/7c2b62c37223)

2. 安装virtualenv & uwsgi

   ```
   pip3 install virtualenv -i https://pypi.douban.com/simple
   pip3 install uwsgi -i https://pypi.douban.com/simple
   ```

    安装后添加virtualenv的环境变量

   ```
   vim /etc/profile
   #将下面内容添加到文件的最下面
   PATH=$PATH:/usr/local/python3/bin
   #是添加的进行生效命令
   source /etc/profile
   #最后查看是否添加成功
   echo $PATH
   ```
3. 安装redis
	装了半天出现
	
	```
	*** [err]: pending querybuf: check size of pending_querybuf after set a big value in tests/unit/pendingquerybuf.tcl
	the used_memory of replica is much larger than master. Master:43866048 Replica:6483753
	```
	
	可能内存不够。不装了，反正暂时没用到数据库
	
4. 虚拟环境

   ```
   # 创建虚拟环境
   virtualenv my_project
   # 进入虚拟环境
   source my_project/bin/activate
   ```
5. 测试后端
	把后端代码放到my_project下，例：my_project/backend，runserver一下看能否正常运行。这里会有centos sqlite3 版本过低的问题，更新版本参考 [https://blog.csdn.net/weixin_43336281/article/details/100055435](https://blog.csdn.net/weixin_43336281/article/details/100055435)（这篇文章里有个小问题，就是创建软链接ln的时候要改成绝对地址的写法）

6. uwsgi
	在my_project/backend下新建uwsgi.ini
	```
	[uwsgi]
	# 设置uwsgi 启动用户，不设置也可，会有警告，也可以设置为当前登录的用户
	uid = root
	gid = root
	#使用nginx连接时使用，Django程序所在服务器地址
	socket=/home/apps/mywebsite/evenyin/backend/uwsgi.sock
	#直接做web服务器使用，Django程序所在服务器地址；如果不想外网直接访问到后端，可以注释掉；建议开启，方便新手确认uwsgi安装好了没
	http=0.0.0.0:8085
	#项目目录
	chdir=/home/apps/mywebsite/evenyin/backend
	#项目中wsgi.py文件的目录，相对于项目目录
	wsgi-file=backend/wsgi.py
	# module也根据wsgi.py文件的目录来写
	module=backend.wsgi:application
	# 进程数
	processes=1
	# 线程数
	threads=2
	# uwsgi服务器的角色
	master=True
	# 存放进程编号的文件路径
	pidfile=uwsgi.pid
	# 日志文件路径
	daemonize=uwsgi.log
	# 指定依赖的虚拟环境
	virtualenv=/home/apps/my_project
	# clear environment on exit #退出时清除环境
	vacuum = true
	```
	这里路径和端口什么的都是乱填的，需要自己把几个路径都修改并且check一下是否正确，建议把uwsgi.ini, uwsgi.log, uwsig.pid这些都放在同一文件夹如my_project/uwsgi/下。然后启动uwsgi：
	```
	# 启动
	uwsgi --ini uwsgi.ini
	# 停止
	uwsgi --stop uwsgi.pid
	# 重启
	uwsgi --reload uwsgi.pid
	```
	这个时候可以登[server ip]:8085看uwsgi看后端是否可以运行。
	如果不能运行，一定记得查看uwsgi.log文件：
	1. 先把项目bug解决
	2. 如果发现开了多个uwsgi（log原文忘了）就`sudo pkill -f uwsgi -9`
	3. 如果一切正常，但没有访问信息，网页提示无法访问，基本是防火墙问题。查看firewalld, selinux是否开启端口；如果是**阿里云服务器**要去阿里云面板设置安全组，阿里云自己有防火墙
	4. 如果一切正常，有访问记录，看返回码，如返回500 server error，是代码runtime error了
	

#### >> 前端
1. 生成dist文件夹
	如果要在服务器上build就先
	+ 安装npm
		```
		wget https://nodejs.org/download/release/v8.6.0/node-v8.6.0-linux-x64.tar.gz
		tar -zxvf node-v8.6.0-linux-x64.tar.gz
		ln -s /home/mytemp/node-v8.6.0-linux-x64/bin/node /usr/bin/node
		ln -s /home/mytemp/node-v8.6.0-linux-x64/bin/npm  /usr/bin/npm
		```
		可输入node -v和npm检查是否安装成功
	+ vue项目根目录下`npm run build`生成dist文件夹
	
	更好的办法是在本机build好直接ftp上传即可。

2. 安装nginx
	参考[https://www.jianshu.com/p/79a313d6cf99](https://www.jianshu.com/p/79a313d6cf99)

3. nginx.conf
	```
	#user  nobody;
	worker_processes  1;
	events {
	    worker_connections  1024;
	}
	
	http {
	    include       mime.types;
	    default_type  application/octet-stream;
	    sendfile        on;
	    keepalive_timeout  65;
	    
	    # backend
	    server {
	        listen 8082;
	        server_name [server ip];   
	        location / {
	            include uwsgi_params;
	            uwsgi_pass unix:/home/apps/mywebsite/evenyin/backend/uwsgi.sock;
	        }
	    }
	    
	    # frontend
	    server {
	        listen       8888;
	        server_name  [server ip];
	        location / {
	            root   /home/apps/mywebsite/dist;
	            index  index.html index.htm;
		    try_files $uri $uri/ @router;
	        }
			location @router {
			    rewrite ^.*$ /index.html last;
			}
	        location /api {
	            proxy_pass http://[server ip]:8082/;
	        }
	        
	        #error_page  404              /404.html;
	        # redirect server error pages to the static page /50x.html
	        error_page   500 502 503 504  /50x.html;
	        location = /50x.html {
	            root   html;
	        }
	    }
	}
	```
	nginx可以先部署后端（# backend那部分），访问[server ip]:8082，先确认后端部署+nginx反代是否成功；再调前端。