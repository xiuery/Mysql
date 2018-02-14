
## 常用查操作

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

### 5.错误处理

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
