# lamp框架（Linux Apache Mysql php）
## MySQL 数据库
> MySQL最初是一个开源软件，而开源软件通常会上传到GNU社区。在GNU社区，需要遵循GPL协议（**不能以盈利为目的**）。由于Sun公司看好MySQL，所以资助了MySQL，让其继续开源。Sun公司靠服务器产品为生，但后来产业逐渐落寞。由于Sun的名气很大，所以后来被Oracle收购。收购之后，Oracle要求MySQL不再遵循GPL协议，但MySQL不愿意。为了解决这个问题，MySQL的创始人就用女儿的名字Maria创建了另一个数据库MariaDB（除了logo和MySQL不一样，其他都一样）。
1. 安装mysql数据库: `yum install mariadb-server`。mariadb 和 mysql 用法都是一样的，是第七版关盘中默认使用的数据库。可以理解为mysql的双胞胎兄弟。
2. 启动服务 `systemctl start mariadb` ，查看端口号验证是否安装成功 `ss -antpl | grep "3306"`。mysql的端口号为3306.
3. 进入数据库 `mysql -u root`

![image](https://github.com/AlphaXiao/CTF-Linux-basics/blob/main/Days/pictures/23.png)

4. - [x] [MySQL基础](https://github.com/AlphaXiao/CTF-Linux-basics/blob/main/MySQ%E5%9F%BA%E7%A1%80.pdf)  （可下载）
### 一些基础用法
1. 数据库存储数据的结构类型:库（文件夹）中可以存放 很多表（存数据的）
  - 进入mysql用的命令是 `mysql -u root`，但进入之后提示符变味了`MariaDB [(none)]> `。mysql是客户端软件，后者是服务端，这是计cs结构中很常见的client-server模式。
3. 为mysql数据库的root用户设定密码

![image](https://github.com/AlphaXiao/CTF-Linux-basics/blob/main/Days/pictures/24.png)

4. 学习再数据库中控制数据库的语句 sql语句（无视大小写）

  - ![image](https://github.com/AlphaXiao/CTF-Linux-basics/blob/main/Days/pictures/25.png)

  - 查看命令的使用方式，比如查看`show`如何使用:`help show;`
  - 查看表内容
  
    ![image](https://github.com/AlphaXiao/CTF-Linux-basics/blob/main/Days/pictures/26.png)
    
  - 让表格内容变得human-readable:
    - 方法一:`select * from user\G;` 这个方法有缺陷，一旦表格非常大会占用很大的内存空间去加载。
    - 方法二:查看字段内容
      
      ![image](https://github.com/AlphaXiao/CTF-Linux-basics/blob/main/Days/pictures/27.png)
      
    - 方法三:根据条件查看表，例如`select host,user,password from user where user='root';`
  - 建立数据库，建立表格写入数据
    -  ![image](https://github.com/AlphaXiao/CTF-Linux-basics/blob/main/Days/pictures/28.png)
    -  ![image](https://github.com/AlphaXiao/CTF-Linux-basics/blob/main/Days/pictures/29.png)

  - 其他增删改查这些基础的知识可以去看上传的MySQL基础文档。 

## PHP安装
![image](https://github.com/AlphaXiao/CTF-Linux-basics/blob/main/Days/pictures/30.png)

1. `yum install php php-mbstring php-mysql` 
  - 修改配置文件`vim /etc/php.ini`
    
    ![image](https://github.com/AlphaXiao/CTF-Linux-basics/blob/main/Days/pictures/31.png)
    
2. 有可能在安装完php后还是没有访问权限，有可能是selinux的问题。可以用`setenforce 0`临时关闭去刷新页面。 `setenforce 1`是开启。
  > selinux 会为linux操作系统中所有的文件设定标签。
    
    
    

