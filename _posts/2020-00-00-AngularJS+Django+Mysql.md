---
title: AngularJS+Django+Mysql
author: Even
date: 2020-09-13 17:04:00 +0800
categories: [CS, Web]
tags: [web]
toc: false
---

设置数据库请参考<a href="https://segmentfault.com/a/1190000018964702">https://segmentfault.com/a/1190000018964702</a>

如果<code>mysql -uroot -p</code>输入密码后无法登录，重启一下mariadb试试：<code>systemctl restart  mariadb</code>

如果无法对数据库授权，出现<code>The MySQL server is running with the --skip-grant-tables option so it cannot execute this statement……</code>，需要先<code>flush privileges</code>

django settings：
<div>
<div>DATABASES = {</div>
<div>    # 'default': {</div>
<div>    #     'ENGINE': 'django.db.backends.sqlite3',</div>
<div>    #     'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),</div>
<div>    # }</div>
<div>    'default': {</div>
<div>        'ENGINE': 'django.db.backends.mysql',   # 数据库引擎</div>
<div>        'NAME': 'pen',  # 数据库名，先前创建的</div>
<div>        'USER': 'pen',     # 用户名，可以自己创建用户</div>
<div>        'PASSWORD': '+rrrey',  # 密码</div>
<div>        'HOST': '45.77.215.38',  # mysql服务所在的主机ip</div>
<div>        'PORT': '3306',         # mysql服务端口</div>
<div>    }</div>
<div>}</div>
</div>
还需要<code>pip install mysqlclient</code>

django连接mysql：
<a href="https://www.liujiangblog.com/course/django/165">https://www.liujiangblog.com/course/django/165</a>

如果无法连接，出现
<code>MySQLdb._exceptions.OperationalError: (2002, "Can't connect to MySQL server on '[ip addr]' (10061)")</code>
则检查mysql的配置文件<code>/etc/my.cnf.d/server.cnf</code>
里面注释掉
<code>skip-networking #屏蔽掉一切TCP/IP连接
bind-address = 127.0.0.1
</code>
这两行，重启mariadb即可。