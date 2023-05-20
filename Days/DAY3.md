# lamp框架（Linux Apache Mysql php）
1. 安装mysql数据库: `yum install mariadb-server`。mariadb 和 mysql 用法都是一样的，是第七版关盘中默认使用的数据库。可以理解为mysql的双胞胎兄弟。
2. 启动服务 `systemctl start mariadb` ，查看端口号验证是否安装成功 `ss -antpl | grep "3306"`。mysql的端口号为3306.
3. 进入数据库 `mysql -u root`

![image](https://github.com/AlphaXiao/CTF-Linux-basics/blob/main/Days/pictures/23.png)


