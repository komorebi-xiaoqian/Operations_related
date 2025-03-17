## 1.查看IP地址

```
ifconfig
ifconfig ens33 | awk 'NR==2{print $2}'
```

## 2.关闭防火墙

```
systemctl stop firewalld
sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config 
# 查看防火墙状态
firewall-cmd --state
getenforce
```

## 3.下载zabbix5.0版本

```
rpm -Uvh https://repo.zabbix.com/zabbix/5.0/rhel/7/x86_64/zabbix-release-latest-5.0.el7.noarch.rpm
yum clean all
yum makecache
```

## 4.安装Zabbix数据库和客户端

```
yum install zabbix-server-mysql zabbix-agent
```

## 5. zabbix换阿里云源

```
sed -i 's#http://repo.zabbix.com#https://mirrors.aliyun.com/zabbix#' /etc/yum.repos.d/zabbix.repo
```

## 6.安装SCL源

```
yum install centos-release-scl
```

## 7.修改 /etc/yum.repos.d 目录下的 CentOS-SCLo-scl-rh.repo 文件

```
# 删除CentOS-SCLo-scl.repo 文件 
		rm -rf CentOS-SCLo-scl.repo
	# 把文件内容全删除，用下面这个替换
```

```
# CentOS-SCLo-rh.repo
#
# Please see http://wiki.centos.org/SpecialInterestGroup/SCLo for more
# information
 
[centos-sclo-rh]
name=CentOS-7 - SCLo rh
baseurl=http://vault.centos.org/centos/7/sclo/$basearch/rh/
gpgcheck=0
enabled=1
 
[centos-sclo-rh-testing]
name=CentOS-7 - SCLo rh Testing
baseurl=http://buildlogs.centos.org/centos/7/sclo/$basearch/rh/
gpgcheck=0
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-SCLo
 
[centos-sclo-rh-source]
name=CentOS-7 - SCLo rh Sources
baseurl=http://vault.centos.org/centos/7/sclo/Source/rh/
gpgcheck=0
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-SCLo
 
[centos-sclo-rh-debuginfo]
name=CentOS-7 - SCLo rh Debuginfo
baseurl=http://debuginfo.centos.org/centos/7/sclo/$basearch/
gpgcheck=0
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-SCLo
```

```
# 清除缓存
	yum clean all
# 建立缓存
	yum makecache
# 查看可用的软件包
	yum repolist
```



## 8.安装Zabbix前端一些组件

```
yum install -y zabbix-web-mysql-scl zabbix-apache-conf-scl
```

## 9.安装mariadb数据库

```
yum install mariadb-server -y
```

## 10.mariadb数据库启动并设置开机自启

```
systemctl enable --now mariadb
```

```
# 查看数据库状态
systemctl status mariadb
```

```
# 查看端口
netstat -tunlp
```

## 11.mysql初始化

```
mysql_secure_installation
```

```
NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none): 
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] y
New password: 
Re-enter new password: 
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] n
 ... skipping.

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

## 12.进入数据库为zabbix配置数据库

```
mysql -uroot -p
```

```
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 9
Server version: 5.5.68-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
3 rows in set (0.00 sec)

# 创建数据库zabbix设置utf8编码格式，以utf8格式启动
MariaDB [(none)]> create database zabbix character set utf8 collate utf8_bin;
Query OK, 1 row affected (0.01 sec)

# 创建zabbix用户，并设置密码为123456
MariaDB [(none)]> create user zabbix@localhost identified by '123456';
Query OK, 0 rows affected (0.00 sec)

# 把zabbix数据库下表的所有权限授予给zabbix用户
MariaDB [(none)]> grant all privileges on zabbix.* to zabbix@localhost;
Query OK, 0 rows affected (0.00 sec)

# 刷新权限
MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)

# 退出
MariaDB [(none)]> exit;
Bye

```

## 13.导入初始架构和数据

```
# 查看文件
ls /usr/share/doc/zabbix-server-mysql-5.0.46/create.sql.gz 
```

```
zcat /usr/share/doc/zabbix-server-mysql-5.0.46/create.sql.gz | mysql -uzabbix -p zabbix
```

```
# 查看是否导入成功
[root@localhost yum.repos.d]# mysql -uzabbix -p123456
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 11
Server version: 5.5.68-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| zabbix             |
+--------------------+
2 rows in set (0.00 sec)

MariaDB [(none)]> use zabbix
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [zabbix]> show tables;
+----------------------------+
| Tables_in_zabbix           |
+----------------------------+
| acknowledges               |
| actions                    |
| alerts                     |
| application_discovery      |
| application_prototype      |
| application_template       |
| applications               |
| auditlog                   |
| auditlog_details           |
| autoreg_host               |
| conditions                 |
| config                     |
| config_autoreg_tls         |
| corr_condition             |
| corr_condition_group       |
| corr_condition_tag         |
| corr_condition_tagpair     |
| corr_condition_tagvalue    |
| corr_operation             |
| correlation                |
| dashboard                  |
| dashboard_user             |
| dashboard_usrgrp           |
| dbversion                  |
| dchecks                    |
| dhosts                     |
| drules                     |
| dservices                  |
| escalations                |
| event_recovery             |
| event_suppress             |
| event_tag                  |
| events                     |
| expressions                |
| functions                  |
| globalmacro                |
| globalvars                 |
| graph_discovery            |
| graph_theme                |
| graphs                     |
| graphs_items               |
| group_discovery            |
| group_prototype            |
| history                    |
| history_log                |
| history_str                |
| history_text               |
| history_uint               |
| host_discovery             |
| host_inventory             |
| host_tag                   |
| hostmacro                  |
| hosts                      |
| hosts_groups               |
| hosts_templates            |
| housekeeper                |
| hstgrp                     |
| httpstep                   |
| httpstep_field             |
| httpstepitem               |
| httptest                   |
| httptest_field             |
| httptestitem               |
| icon_map                   |
| icon_mapping               |
| ids                        |
| images                     |
| interface                  |
| interface_discovery        |
| interface_snmp             |
| item_application_prototype |
| item_condition             |
| item_discovery             |
| item_preproc               |
| item_rtdata                |
| items                      |
| items_applications         |
| lld_macro_path             |
| lld_override               |
| lld_override_condition     |
| lld_override_opdiscover    |
| lld_override_operation     |
| lld_override_ophistory     |
| lld_override_opinventory   |
| lld_override_opperiod      |
| lld_override_opseverity    |
| lld_override_opstatus      |
| lld_override_optag         |
| lld_override_optemplate    |
| lld_override_optrends      |
| maintenance_tag            |
| maintenances               |
| maintenances_groups        |
| maintenances_hosts         |
| maintenances_windows       |
| mappings                   |
| media                      |
| media_type                 |
| media_type_message         |
| media_type_param           |
| module                     |
| opcommand                  |
| opcommand_grp              |
| opcommand_hst              |
| opconditions               |
| operations                 |
| opgroup                    |
| opinventory                |
| opmessage                  |
| opmessage_grp              |
| opmessage_usr              |
| optemplate                 |
| problem                    |
| problem_tag                |
| profiles                   |
| proxy_autoreg_host         |
| proxy_dhistory             |
| proxy_history              |
| regexps                    |
| rights                     |
| screen_user                |
| screen_usrgrp              |
| screens                    |
| screens_items              |
| scripts                    |
| service_alarms             |
| services                   |
| services_links             |
| services_times             |
| sessions                   |
| slides                     |
| slideshow_user             |
| slideshow_usrgrp           |
| slideshows                 |
| sysmap_element_trigger     |
| sysmap_element_url         |
| sysmap_shape               |
| sysmap_url                 |
| sysmap_user                |
| sysmap_usrgrp              |
| sysmaps                    |
| sysmaps_elements           |
| sysmaps_link_triggers      |
| sysmaps_links              |
| tag_filter                 |
| task                       |
| task_acknowledge           |
| task_check_now             |
| task_close_problem         |
| task_data                  |
| task_remote_command        |
| task_remote_command_result |
| task_result                |
| timeperiods                |
| trends                     |
| trends_uint                |
| trigger_depends            |
| trigger_discovery          |
| trigger_tag                |
| triggers                   |
| users                      |
| users_groups               |
| usrgrp                     |
| valuemaps                  |
| widget                     |
| widget_field               |
+----------------------------+
166 rows in set (0.01 sec)

MariaDB [zabbix]> exit
Bye
```

## 14.修改zabbix_server配置文件

```
vim /etc/zabbix/zabbix_server.conf 
grep "^DBP" /etc/zabbix/zabbix_server.conf 
DBPassword=123456
```

## 15.配置php后端

```
vim /etc/opt/rh/rh-php72/php.fpm.d/zabbix.conf
```

```
# 查看设置是否成功
tail -1 zabbix.conf 
php_value[date.timezone] = Asia/Shanghai
```

## 16.重启服务

```
systemctl restart zabbix-server zabbix-agent httpd rh-php72-php-fpm

# 设置开机自启
systemctl enable zabbix-server zabbix-agent httpd rh-php72-php-fpm

```

## 17.解决显示中文乱码问题

```
# 下载个中文字体
yum install wqy-microhei-fonts
# 替换
 \cp /usr/share/fonts/wqy-microhei/wqy-microhei.ttc /usr/share/fonts/dejavu/DejaVuSans.ttf
```

## 18.安装zabbix_agent2

```
yum install zabbix-agent2 -y
```

## 19.配置zabbix_agent2

```
grep -Ev "^#|^$" /etc/zabbix/zabbix_agent2.conf 
PidFile=/var/run/zabbix/zabbix_agent2.pid
LogFile=/var/log/zabbix/zabbix_agent2.log
LogFileSize=0
Server=192.168.174.130
ServerActive=192.168.174.130
Hostname=nfs01
Include=/etc/zabbix/zabbix_agent2.d/*.conf
ControlSocket=/tmp/agent.soc
```

## 20.zabbix-agent2启动并设置开机自启

```
systemctl enable --now zabbix-agent2
```

