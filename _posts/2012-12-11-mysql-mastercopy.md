---
layout: post
title: MySQL主从复制
date: 2012-12-11 09:29
comments: true
categories: 
---
首先修改mysql的配置文件mysql.ini或my.cnf

服务器A(主)：192.168.0.26    
[mysqld]  
log-bin="E:/log/backup"  
server-id=26

服务器B(从)：192.168.0.27  
[mysqld]  
log-bin="E:/log/backup"  
server-id=27  

重启A和B

在A中授权：  
{% highlight sql %}
GRANT REPLICATION SLAVE ON *.* to 'tom'@'%' identified by '000000';
{% endhighlight %}   
其中tom为新建的用户名，%为所有的IP都可以，推荐使用固定的IP地址，000000为密码

在A中执行：
{% highlight sql %}
mysql>show master status;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| backup.000004    |      225 |              |                  |
+------------------+----------+--------------+------------------+
{% endhighlight %}   


在B中配置：
{% highlight sql %}
mysql>change master to 
	  master_host='192.168.0.26',
	  master_user='tom',
	  master_password='123456',
      master_log_file='backup.000004',
	  master_log_pos=225;
mysql>start slave;    // 启动从服务器复制功能

mysql> show slave status\G; // 检查复制状态
...
Slave_IO_Running: Yes       // 此状态必须YES
Slave_SQL_Running: Yes       // 此状态必须YES
...
{% endhighlight %} 

到此全部完成，在A中添加的数据库，表，数据都会同步到B中。