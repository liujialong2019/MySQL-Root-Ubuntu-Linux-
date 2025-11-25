# MySQL-Root-Ubuntu-Linux-PassWord-Change
本文总结了 MySQL 在 Linux 系统上第一次设置密码、判断密码方式以及修改密码后的行为变化。


## 1️⃣ 第一次安装 MySQL 时的 root 密码设置  

1. 安装 MySQL（以 Ubuntu 为例）
    ```
    sudo apt update
    sudo apt install mysql-server -y
    sudo systemctl start mysql
    sudo systemctl enable mysql
    ```

3. 设置 root 密码（第一次配置）

    Ubuntu 上 MySQL 默认 root 使用 auth_socket 插件，直接使用 Linux 系统用户登录：

    ```
    sudo mysql
    ```

    如果想为 root 设置密码并使用密码登录，可以执行：

    ```
    ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '你的新密码';
    FLUSH PRIVILEGES;
    ```

    注意：此操作将 root 的认证方式从 auth_socket 改为 密码验证。

##  2️⃣ 如何判断 root 用户第一次使用哪种验证方式


在 MySQL 中执行：

```
sudo mysql
SELECT user, host, plugin FROM mysql.user WHERE user='root';
```

输出示例：

```
+------+-----------+-----------------------+
| user | host      | plugin                |
+------+-----------+-----------------------+
| root | localhost | auth_socket           |
+------+-----------+-----------------------+
```

auth_socket → 使用系统用户登录，无需密码

mysql_native_password 或 caching_sha2_password → 使用密码登录


## 3️⃣ 设置密码后的变化

如果 root 使用密码登录（mysql_native_password / caching_sha2_password）

必须使用密码：

```
mysql -u root -p
```

sudo mysql 也需要输入密码，Linux 系统用户权限不再直接允许登录 MySQL。

如果 root 使用 auth_socket（默认 Ubuntu 行为）

无需密码，直接使用系统用户登录：

```
sudo mysql
```

适合本地开发环境，生产环境建议使用密码登录以增强安全性。

## 4️⃣ 修改 root 密码示例

```
sudo mysql
```

修改为密码登录：

```
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123123';
FLUSH PRIVILEGES;
```

恢复 Ubuntu 默认 auth_socket 登录：

```
ALTER USER 'root'@'localhost' IDENTIFIED WITH auth_socket;
FLUSH PRIVILEGES;
```

#  5️⃣ 总结
```
状态                                               登录方式  
auth_socket（默认 Ubuntu）                         sudo mysql 无需密码  
mysql_native_password / caching_sha2_password	   mysql -u root -p 需要密码
```

第一次设置密码：需要执行 ALTER USER ... IDENTIFIED WITH ... BY ...

判断方式：查看 plugin 列

修改密码后的影响：改变认证插件后，原来的系统用户登录方式可能失效，需要密码登录。
