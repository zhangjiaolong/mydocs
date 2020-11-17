# mariadb-mysql双机主备-主主模式

## 1. mysql-mariadb主备实现

### （1）安装mariadb后：

1. 修改root密码

```
ALTER USER 'root'@'localhost' IDENTIFIED BY '密码';

FLUSH PRIVILEGES;
```

2. 开启远程访问 将mariadb中的配置文件中将下面的内容前加"#"
```
#bind-address=127.0.0.1

保存退出
重启mariadb
sudo systemctl restart mariadb
```

3. 添加备份账户（主服务器以及从服务器都要添加）

```
grant replication slave on *.* to '备份账户名称'@'备份服务器ip(非本机)' identified by '备份账户密码';

FLUSH PRIVILEGES;
```

服务器A配置：
```
打开mariadb配置文件[mysqld]添加一下内容
server-id = 1
log-bin=mysql-bin 
binlog-do-db = auditdb
binlog-ignore-db = mysql
log-slave-updates
sync_binlog = 1
auto_increment_offset = 1
auto_increment_increment = 2
replicate-do-db = auditdb
replicate-ignore-db = mysql,information_schema

#查看主机状态
show master status;

#查看file、position的内容并分别填写到master_log_file、master_log_pos的值
master_host='59.151.15.36',master_user='备份账户名称',master_password='备份账户密码', master_log_file='上一条语句查询出来的内容',master_log_pos='上一条语句查询出来的内容';

#启动备份
start slave;

#查看启动状态 查询的内容中Slave_IO_Running: Yes  Slave_SQL_Running: Yes 则表示启动成功。
show slave status\G;
```

服务器B配置：
```
打开mariadb配置文件[mysqld]添加一下内容
server-id = 2
log-bin=mysql-bin 
replicate-do-db = auditdb
replicate-ignore-db = mysql,information_schema,performance_schema
binlog-do-db = auditdb
binlog-ignore-db = mysql
log-slave-updates
sync_binlog = 1
auto_increment_offset = 2
auto_increment_increment = 2

#查看主机状态
show master status;

#查看file、position的内容并分别填写到master_log_file、master_log_pos的值
master_host='59.151.15.36',master_user='备份账户名称',master_password='备份账户密码', master_log_file='上一条语句查询出来的内容',master_log_pos='上一条语句查询出来的内容';

#启动备份
start slave;

#查看启动状态 查询的内容中Slave_IO_Running: Yes  Slave_SQL_Running: Yes 则表示启动成功。
show slave status\G;
```
