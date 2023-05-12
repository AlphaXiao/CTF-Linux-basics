# Xshell的使用
1. ssh 远程连接 [一般做实验是真机用Xshell连虚拟机]
    - `ssh 用户名@IP地址`,例如:`ssh root@192.168.2.1`
2. 传文件
    - sftp是一种安全文件传输协议，`sftp 用户名@IP地址`,例如:`sftp root@192.168.2.1`。
    - `put`是上传，`get`是下载。例如:`put c:\users\xiaoxiao\desktop\software\...`（注意Windows路径是反斜杠）。最好的方式是输入`put`直接回车（enter），会弹窗出来让你选择要传输的文件。
      - lcd 命令切换windows(本地)路径 [l表示local（本地）]
      - lls 查看windwos（本地）本地文件[l表示local（本地）]
      - ls 查看（对方）linux的数据
      - cd 切换（对方）linux路径
      - put 弹窗出来让你选择要传输的文件 
    - CentOS7其实是自带VMtools的，这意味着你可以从直接从真机拖拽或者复制文件进虚拟机。但有时候tools可能会失效，你又着急上传东西进虚拟机，这个时候你可以选择用Xshell这样的远程工具进行文件传输。
    
# 计划任务
> - 计划任务是一种非常有用的手段，可以让用户在预定时间自动执行某个命令或脚本。如果某个服务器被攻陷成为了肉机，肯定不可能立刻让审计人员发现端倪，这个时候大多黑客是先搞个计划任务，定时触发病毒或者一些命令脚本。作为安全运维人员需要查看系统中是否有定期计划任务脚本。
> - 计划任务有两种一次性计划任务`at`和周期性计划任务`cron`。在运用前，需要先检查这两个软件的进程是否开启 `systemctl status atd` 和 `systemctl status crond`。命令中的`d`表示的是`daemon` 可以理解为守护进程，一般软件的进程都是软件名字加个d。

## 一次性计划任务 at
1. `date` 命令可查看当前时间。`date +%F`命令仅仅显示年月日。使用`man date`可以查看具体使用方式。例如`小时：分钟` `date + %H:%M`。
2. 如果要当前主机在1分钟后把test写进 `/tmp/draft.txt`中，命令为`at now +1 min` `echo "test" > /tmp/draft.txt`，写完后 Ctrl + D 结束编写。
3. 指定时间执行任务，例如:`at 8:00 2022-09-23`，跟着需要执行的命令，可跟多条，写完后 Ctrl + D 结束编写。
4. `at -l`查看未运行任务。`at -c 任务编号`查看未运行脚本内容，看最后就行，前面都是环境变量信息。`atrm 编号` 删除未运行的计划。
5. 如果当前时间11点设定了11:10分执行关机命令 11:05的时候把机器关上了。 11:20的时候开机。开机会立即执行超时的计划任务。
6. 如果是定时触发一个脚本，定完时之后，需要执行命令直接写脚本的绝对路径就可以。例如: 有脚本`/home/user/my_script.sh`。设定10分钟之后执行`at now + 10 minutes` `/bin/bash /home/user/my_script.sh`。需要注意的是，在执行脚本前，可能需要修改脚本的权限，以允许其执行。可以使用`chmod +x /home/user/my_script.sh` 。

## 周期性计划任务 cron
1. 例1:在6月份每周五的11点16分执行命令`date >> /tmp/test.txt`
    - `date >> /tmp/test.txt` 把当前时间写进`/tmp/test.txt`文档中。
    - `crontab -e` 会直接进入vim编辑页面。具体这个软件如何使用可以查看它的配置文件。配置文件都放在`/etc`下，`/etc/crontab`是它的配置文件。
    - `vim /etc/crontab` 根据题目
        - ![image](https://github.com/AlphaXiao/CTF-Linux-basics/blob/main/Days/pictures/11.png)
    - 注意：普通用户是不能更改`vim /etc/crontab`，来实施计划任务的，因为没有权限。普通用户需用`crontab -e`自己编辑。一开始自己学习可以在root下，在配置文件中看着注释来设定。
2. 例2：普通用户使用crontab计划任务对 `/var/log/secure` 进行**打包并备份**。要求将登录日志打包备份到`/tmp`目录中，一分钟备份一次看看效果。
    - `crontab -l` 可以查看周期性计划任务。如果是在普通用户下设定的周期性计划任务，用root账户查看（root可以查看所有账户的计划任务），则命令为 `crontab -u 用户名 -l`。
    - 编写任务 `* 11 19 6 * cd /var/log; tar -zcf /tmp/secure.tar.gz secure`把secure文件打包。先执行分号前的命令再执行分号后。
    - 上面这样写会存在一个问题，每分钟备份一次，后一分钟的备份会覆盖前一分钟的备份，始终都只有一个文件。我们需要让备份文件的文件名产生变化，备份成不同的文件。命令变为 ```* 11 19 6 * cd /var/log; tar -zcf /tmp/secure`date +\%H:\%M`.tar.gz secure``` 。```secure`date +\%H:\%M```是名字结合时间（月:日）的写法。还要注意百分号是特殊字符，用在命令中需要使用转义字符。

# 一个钓鱼网站构建的整个过程
> 只要有一个linux系统，开启DHCP分发DNS，使其他主机网页重定向去到钓鱼网站，后台记录你的信息。因此主机所有数据都会经过linux系统，hacker可以随意监听或截取数据包。说白了就是直接控制DNS。

实验拓扑（Win2003、CentOS）：

![image](https://github.com/AlphaXiao/CTF-Linux-basics/blob/main/Days/pictures/12.png)

1. 配置两个网卡的IP地址
2. 通过yum源安装dhcp服务：`yum install dhcp`
    - 配置DHCP，让其可以给Windows分发ip。①需要有地址池 ②要给客户分配网关和dns地址。`vim /etc/dhcp/dhcpd.conf`
    
    ![image](https://github.com/AlphaXiao/CTF-Linux-basics/blob/main/Days/pictures/13.png)
    
    精简配置内容：
    
    ![image](https://github.com/AlphaXiao/CTF-Linux-basics/blob/main/Days/pictures/14.png)
    
    启动服务并验证是否开启：`systemctl start dhcpd` `ss -anupl | grep 67` Windows的dhcp端口是67/68，linux的dhcp是67。
    
3. 把Windows2003配置为自动获取IP。点击右下角的小电脑→属性→tcp/ip协议→勾选两个自动获取选项。验证：调出cmd输入 `ipconfig /all`。
4. 设定DNS服务
    - 安装dns：`yum install bind`
    - 配置dns服务使Windows客户机能够正常解析网址。DNS有两个配置文件，先配置主配置文件 `vim /etc/named.conf`
    
        ![image](https://github.com/AlphaXiao/CTF-Linux-basics/blob/main/Days/pictures/15.png)
        
        ![image](https://github.com/AlphaXiao/CTF-Linux-basics/blob/main/Days/pictures/16.png)
        
    - 验证dns服务： `nslookup www.qq.com`

5. Centos已经可以正常访问dns解析了，但是因为windows没有做nat网络地址转换,所以Windows还不行
















































