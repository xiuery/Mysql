# Mysql

mysql的常用语法

### 常用

- 修改密码
```
mysql>update user set password=password('root') where user='root' and host='localhost';
```

- 清空表
```
#只清空数据
mysql>delete from xx
#清空数据，并且重置auto_increment
mysql>truncate table xx
```


### 1.数据导出

- 所有库
```
$ /usr/bin/mysqldump -u root -proot\!123\$ --all-databases > databases.sql
```

- 除指定库中的指定表外的所有库所有表
```
$ /usr/bin/mysqldump -u root -proot\!123\$ --all-databases --ignore-table=xs.app_logs > databases.sql
```

- 指定库中所有表
```
$ /usr/bin/mysqldump -u root -proot\!123\$ xs > xs-datas.sql
```

- 单张表
```
$ /usr/bin/mysqldump -u root -proot\!123\$ xs app_logs > xs-app_logs-datas.sql
```

- 除指定表之外，可指定多张表
```
$ /usr/bin/mysqldump -u root -proot\!123\$ xs --ignore-table=xs.app_logs --ignore-table=xs.app_user > xs-ignore-app_logs-app_user-datas.sql
```

- 单张表+条件
```
$ /usr/bin/mysqldump -u root -proot\!123\$ xs app_logs --where='ID<49000000' > xs-app_logs-datas-1.sql
$ /usr/bin/mysqldump -u root -proot\!123\$ xs app_logs --where='ID>48888889' > xs-app_logs-datas-2.sql
```

- 导出结构不导出数据
```
$ /usr/bin/mysqldump -u root -proot\!123\$　--opt　-d　数据库名　>　xxx.sql　
```

- 导出数据不导出结构
```
$ /usr/bin/mysqldump -u root -proot\!123\$　-t　数据库名　>　xxx.sql
```

- 其他选项
```
--all--database/-A
--add-drop--database
--add-drop-tables
--add-locking           #用LOCK TABLES和UNLOCK TABLES语句引用每个表转储
--allow-keywords        #允许创建关键字列名
---comments[={0|1}]
--compact	            #该选项禁用注释并启用--skip-add-drop-tables、--no-set-names、--skip-disable-keys和--skip-add-locking选项
```

### 2.特殊查询

- 字符匹配截取
```
substring_index(字段名, 匹配字符, 从第几个开始匹配)    #返回匹配成功位置
substring(字段名, 开始位置, 长度)                    #返回截取后字段
example：
trim(substring(title,length(substring_index(title,'-',3))+2))
```

- 条件匹配
```
where title REGEXP '-'
```

- 合并多个查询结果
```
union all
```

- 复制表结构
```
create table `xs`.`app_log_1` like `xs`.`app_log`
```

- 复制表结构+数据
```
create table `xs`.`app_log_1` as select * from `xs`.`app_log`
```

- 查询插入记录
```
mysql>insert into `xs`.`app_log`
select
`id` as `id`,
`user_name` as `username`
`user_id` as `user_id`
`create_time` as `create_time`
`update_time` as `update_time`
`delete_time` as `delete_time`
from `xs`.`app_acivity`
where id > 1000000
```
- 查询同一字段值存在多个
```
select `id`,`name`,count(`name`) from `xs`.`app_user` group by `name` having count(`name`) > 1
```

### 3.数据库脚本
```
/usr/bin/mysql -uroot -proot\!123\$ << EOF
insert into `xs`.`app_log`
select
`id` as `id`,
`user_name` as `username`
`user_id` as `user_id`
`create_time` as `create_time`
`update_time` as `update_time`
`delete_time` as `delete_time`
from `xs`.`app_acivity`
where id > 1000000
EOF
```

### 4.数据库备份脚本
```
#!/bin/bash
DB_USER=root
DB_PWD=root!123$

cd /backup

#echo "------------- start backup web -------------"
#WEBBAK=webbak_$(date +%y%m%d).tar.xz
#不备份指定目录文件
#tar --exclude=/home/wwwroot/app/cache/* -zcvf $WEBBAK  /home/wwwroot/app
#echo "------------- end backup web -------------"

echo "------------- start backup database -------------"

echo "##### xs ignore app_logs"
XSIGNORELOGSBAK=xs-ignore-app_logs.sql
XSIGNORELOGSBAKTAR=xs-ignore-app_logs-$(date +%y%m%d).tar.xz
/usr/bin/mysqldump -u$DB_USER -p$DB_USER --ignore-table=xs.app_logs > $XSIGNORELOGSBAK
tar zcvf $XSIGNORELOGSBAKTAR $XSIGNORELOGSBAK
```

### 5.添加用户并授权
```
# 创建用户 - 添加权限
create user xiuery@'%' identified by 'xiuery';
grant select,update,delete on *.* to xiuery;

# or 创建用户并添加权限
grant select,update,delete on *.* to 'xiuery'@'%' identified by 'xiuery';

# 使修改立即生效
flush privileges;
```

### 6.授权远程连接
```
# 添加新用户
grant select,update,delete on *.* to 'xiuery'@'%' identified by 'xiuery';

# 修改my.cnf
vi /etc/my.cnf
skip-networking             # 注释掉
bind-address = 0.0.0.0      # 需要连接的ip地址

# 重启mysql

# 如果还是无法远程连接，就要修改服务器的防火墙了
# ×××××××× 注意：这是很不安全的 ×××××××××
# 打开防火墙配置文件
vi /etc/sysconfig/iptables
# 开放3306
-A INPUT -p tcp -m tcp --dport 3306 -j ACCEPT
```

### 7.错误处理

- ONLY_FULL_GROUP_BY报错
```
#在sql_mode中删除ONLY_FULL_GROUP_BY
$ vi /etc/my.cnf
sql_mode = 'STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION'

or

mysql>set global sql_mode = 'STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION';
```

- 支持federated引擎
```
#在配置文件中添加federated
$ echo 'federated' >> /etc/my.cnf
```

- 数据库表crashed修复
```
#检查指定表
$ /usr/bin/mysqlcheck -u root -proot\!123\$ -c xs app_logs
#修复指定表
$ /usr/bin/mysqlcheck -u root -proot\!123\$ -r xs app_logs
```

- source 导入乱码的问题
```
mysql -u root -p --default-character-set=utf8
```

- auto_increment的问题
在表设计自增时，如果想要在中间插入几条记录,先更新表
```
update `xs`.`app_log` set id = id+3 where id>20 order by id desc
```
此时就会造成一个问题，表会增加3个id数据，但是auto_increment 任然停留在原来的位置，这时再添加数据就会报主键错误
