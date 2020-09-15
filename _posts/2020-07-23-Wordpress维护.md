---
title: Wordpress
author: Even
date: 2020-07-23 14:23:00 +0800
categories: [CS, Web]
tags: [wordpress, web]
toc: false
---

```
# 杀死所有uwsgi进程
sudo pkill -f uwsgi -9

# 启动
uwsgi --ini uwsgi.ini
# 停止
uwsgi --stop uwsgi.pid
# 重启
uwsgi --reload uwsgi.pid
# 日志路径
/home/soaproj/uwsgi/uwsgi.log
```

```
# nginx
# 重启
/usr/local/nginx/sbin/nginx -s reload
# conf路径
/usr/local/nginx/conf/nginx.conf
# 查看日志
cat /usr/local/nginx/logs/access.log | tail -n 20
cat /usr/local/nginx/logs/error.log | grep GET\ /\
```

```
# mariadb
# 重启
service mariadb restart
```

两台服务器间文件实时同步：
<a href="https://www.jianshu.com/p/76ec8ac0c8b1?tdsourcetag=s_pctim_aiomsg" target="_blank" rel="noopener noreferrer">Linux服务器间文件实时同步</a>
