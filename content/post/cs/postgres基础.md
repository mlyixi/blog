---
title: "postgres基础"
date: 2014-09-06T14:08:52+08:00
tags: ["postgresql"]
categories: ["CS"]
toc: true
---

```zsh
sudo su - postgres
psql -c "alter user postgres with password 'StrongAdminP@ssw0rd'"
sudo vi /etc/postgresql/12/main/postgresql.conf
listen_addresses = '*'
sudo vi /etc/postgresql/12/main/pg_hba.conf
host    all             all             192.168.10.0/24            md5
sudo systemctl restart postgresql


sudo apt install pgadmin4 pgadmin4-apache2
```

```sql
create table dept(
	id serial primary key,
	name varchar(20)
)

create table employee1(
	id serial primary key,
	name varchar(20),
	age int,
	salary real,
	dept_id int references dept (id)
)

insert into dept (name) values('人事部');
insert into dept (name) values('财务部');
insert into dept (name) values('公关部');
insert into dept (name) values('总经理办公室');

insert into employee1 values(DEFAULT,'小乔',18,10000,1);
insert into employee1 values(DEFAULT,'大乔',19,10000,1);
insert into employee1 values(DEFAULT,'曹操',20,12000,2);
insert into employee1 values(DEFAULT,'周瑜',21,13000,3);
insert into employee1 values(DEFAULT,'刘备',22,14000,4);

select * from dept;
```
# 选择记录
```sql
SELECT * FROM user_tbl;
```
# 更新数据
```sql
UPDATE user_tbl set name = '李四' WHERE name = '张三';
```
# 删除记录
```sql
DELETE FROM user_tbl WHERE name = '李四' ;
```
# 添加栏位
```sql
ALTER TABLE user_tbl ADD email VARCHAR(40);
```
# 更新结构
```sql
ALTER TABLE user_tbl ALTER COLUMN signup_date SET NOT NULL;
```
# 更名栏位
```sql
ALTER TABLE user_tbl RENAME COLUMN signup_date TO signup;
```
# 删除栏位
```sql
ALTER TABLE user_tbl DROP COLUMN email;
```
# 表格更名
```sql
ALTER TABLE user_tbl RENAME TO backup_tbl;
```
# 删除表格
```sql
DROP TABLE IF EXISTS backup_tbl;
```