# Mysql授予账户权限问题


mysql通过root账号登录，新建账户并授予权限会报错：

```sql
mysql> GRANT SELECT ON *.* TO 'select_user'@'%';
ERROR 1045 (28000): Access denied for user 'root'@'%' (using password: YES)
```

解决方案：

```sql
mysql> update mysql.user set Grant_priv='Y' where user = 'root' and host = '%';
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

然后退出登录，重启mysql服务，之后就能正常授权了。

## Reference

- [MySQL错误：Access denied for user &#39;root&#39;@&#39;%&#39; to database &#39;xxx&#39; - 我想养只狗 - 博客园](https://www.cnblogs.com/Dog1363786601/p/17226101.html)


---

> 作者: [starifly](https://github.com/starifly)  
> URL: http://localhost:1313/posts/mysql-account-permission-grant-problem/  

