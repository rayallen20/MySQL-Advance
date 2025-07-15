# 附录2 MySQL8安装

## STEP1. 安装

```
sudo apt update
sudo apt upgrade -y
sudo apt install mysql-server -y
```

## STEP2. 检测

```
root@mysql-master:~# mysql --version
mysql  Ver 8.0.42-0ubuntu0.22.04.2 for Linux on x86_64 ((Ubuntu))
```

## STEP3. 启动并设置开机自启动

```
root@mysql-master:~# sudo systemctl enable mysql
Synchronizing state of mysql.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable mysql
```

```
root@mysql-master:~# sudo systemctl start mysql
```

## STEP4. 检查状态

```
root@mysql-master:~# sudo systemctl status mysql
● mysql.service - MySQL Community Server
     Loaded: loaded (/lib/systemd/system/mysql.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2025-07-14 02:08:01 UTC; 2min 36s ago
   Main PID: 49730 (mysqld)
     Status: "Server is operational"
      Tasks: 37 (limit: 19009)
     Memory: 364.3M
        CPU: 1.837s
     CGroup: /system.slice/mysql.service
             └─49730 /usr/sbin/mysqld

Jul 14 02:08:00 mysql-master systemd[1]: Starting MySQL Community Server...
Jul 14 02:08:01 mysql-master systemd[1]: Started MySQL Community Server.
```

## STEP5. 安全初始化

```
root@mysql-master:~# sudo mysql_secure_installation
```

### 5.1 是否启用密码强度验证插件

```
Securing the MySQL server deployment.

Connecting to MySQL using a blank password.

VALIDATE PASSWORD COMPONENT can be used to test passwords
and improve security. It checks the strength of password
and allows the users to set only those passwords which are
secure enough. Would you like to setup VALIDATE PASSWORD component?

Press y|Y for Yes, any other key for No: n
```

- 选否即可

### 5.2 是否移除匿名用户

```
By default, a MySQL installation has an anonymous user,
allowing anyone to log into MySQL without having to have
a user account created for them. This is intended only for
testing, and to make the installation go a bit smoother.
You should remove them before moving into a production
environment.

Remove anonymous users? (Press y|Y for Yes, any other key for No) : y
```

- 选是即可

### 5.3 是否禁止root远程登录

```
Normally, root should only be allowed to connect from
'localhost'. This ensures that someone cannot guess at
the root password from the network.

Disallow root login remotely? (Press y|Y for Yes, any other key for No) : n
```

- 选否即可

### 5.4 是否删除`test`数据库

```
By default, MySQL comes with a database named 'test' that
anyone can access. This is also intended only for testing,
and should be removed before moving into a production
environment.


Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y
```

- 选是即可

### 5.5 是否重新加载权限表

```
Reloading the privilege tables will ensure that all changes
made so far will take effect immediately.

Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y
```

- 选是即可

## STEP6. 设置允许远程登录

### 6.1 设置密码

- 登录MySQL

```
root@mysql-master:~# mysql -u root -p
```

- 修改密码

```SQL
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';
Query OK, 0 rows affected (0.01 sec)
```

- 刷新权限

```
mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.01 sec)
```

- 退出并重新登录

```
mysql> exit
Bye
```

```
root@mysql-master:~# mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
...
```

### 6.2 允许远程登录

- 修改配置文件

```
vim /etc/mysql/mysql.conf.d/mysqld.cnf
```

将其中的`bind-address = 127.0.0.1`修改为`bind-address = 0.0.0.0`

- 重新启动服务

```
root@mysql-master:~# sudo systemctl restart mysql
```

- 创建`root@'%'`用户

```
root@mysql-master:~# mysql -u root -p
...
mysql> CREATE USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
Query OK, 0 rows affected (0.01 sec)
```

- 赋权并刷新权限

```
mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
Query OK, 0 rows affected (0.00 sec)
```

```
mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)
```

- 确认

```
mysql> SELECT user, host FROM mysql.user WHERE user = 'root';
+------+-----------+
| user | host      |
+------+-----------+
| root | %         |
| root | localhost |
+------+-----------+
2 rows in set (0.01 sec)
```

注: 生产环境不要允许所有IP远程访问,仅允许指定的客户端IP即可

```
CREATE USER 'admin'@'你的客户端IP' IDENTIFIED BY '安全密码';
GRANT ALL PRIVILEGES ON *.* TO 'admin'@'你的客户端IP';
```