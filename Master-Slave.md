
## 简单主从配置

### 1.master
```
$ vi /etc/my.cnf
    server-id = 1
    log-bin = /var/run/mysqld/mysql-bin

$ mysql -uroot -ppassword  
    mysql>show master status;       #slave中需要结果集中的position
```

### 2.slave
```
$ vi /etc/my.cnf
    server-id = 2

$ mysql -uroot -ppassword
	mysql>stop slave;
	mysql>CHANGE MASTER TO
    MASTER_HOST='127.0.0.1',
    MASTER_USER='root',
    MASTER_PASSWORD='root',
    MASTER_LOG_FILE='binlog.000001',
    MASTER_LOG_POS=154;
	msyql>start slave;
	mysql>show slave status;
```

### 3.Error

- 查看具体的error
```
mysql>show slave status
```

- id冲突
```
删掉从库的中相应的id
```

- 从库同步delete语句：not find record
```
#直接执行跳过错误
mysql>SET GLOBAL SQL_SLAVE_SKIP_COUNTER = 1;   #值可以设置, 跳过指定数量的事务
mysql>stop slave;
mysql>start slave;
```

- memory存储引擎表的同步
```
#修改存储引擎类型
```

- 修改mysql的配置文件，通过slave_skip_errors参数来跳所有错误或指定类型的错误
```
$ vi /etc/my.cnf
    slave-skip-errors=1062,1053,1146    #跳过指定error类型的错误
    slave-skip-errors=all               #跳过所有错误
```