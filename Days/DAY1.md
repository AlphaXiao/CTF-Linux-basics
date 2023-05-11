# 说明
- [x]  学习工具：在VMware里装了CentOS7、Kali和win2003三台虚拟机，使用这三台机子做实验。
- [x]  Linux基础：开始学习前已经了解了Linux的基本命令以及部分软件的使用例如vim、nano。
- [x]  虚拟机配置：Kali有配置好源、下载了Googlepinyin、下载了VM-tools。CentOS永久关闭了防火墙`systemctl disable firewall.service`。关闭了虚拟网卡服务 `systemctl disable libvirtd.service`，这段命令输完之后需要重启虚拟机才会关闭虚拟网卡服务，可以用`systemctl reboot`重启。centos指定了本地光盘作为软件的安装源，操作步骤为`cd /etc/yum.repos.d/`找到yum源文件存放位置。`rm -rf *`清空yum源文件。`systemctl start autofs` 开启光盘路径优化服务。`systemctl enablet autofs`使开机时自动开启。这样操作后会将光盘自动挂载到本地路径，存放在 `/misc/cd`。这样做的好处是可以离线安装软件包，下载软件不受网络影响。
- [x]  网络方面基础：目前是自学通过了CCNA考试。

## 学习为虚拟机配置IP地址 
### 配置DHCP自动获取
> kali虚拟机和centos虚拟机通过配置好的ip地址实现相互ping通。实验要求两台主机同时都使用nat模式。
1. 查看CentOS虚拟机是否处于NAT模式

    ![image](https://github.com/AlphaXiao/CTF-Linux-basics/blob/main/Days/pictures/1.png)

    ![image](https://github.com/AlphaXiao/CTF-Linux-basics/blob/main/Days/pictures/2.png)

2. 配置并查看ip地址
    - 通过`ip addr` 查看ip地址
    
    - Kali的ip地址是自动分配的：VMware菜单栏的编辑 → 虚拟网络编辑器 → 更改设置。最下方可以看到Kali是使用本地DHCP服务将IP自动分配给虚拟机的。
    
    - CentOS需要手动配置IP。
      - 具体怎么手动配置可以查看帮助文档 `ip addr help`。从上面Kali的IP地址中看到网段，这里自己配置CentOS的IP需要和Kali的IP在同一网段内。 `ip addr add 192.168.226.200/24 dev ens33`。 
      
        ![image](https://github.com/AlphaXiao/CTF-Linux-basics/blob/main/Days/pictures/3.png)
      
      - 配置完可以和Kali互ping一下 `ping -c4 x.x.x.x`（ping4次）。再ping一下外网 `ping -c4 114.114.114.114`(腾讯的dns服务器)。CentOS目前ping外网是ping不通的，因为没有配置网关。
      - Kali的网关是配置好的，`ip route`查看默认网关。给CentOS配置和Kali一样的网关。具体如何配置查看帮助route文档 `ip route help`。例：`ip route add default via x.x.x.x`。配置完再ping一下外网就可以ping通了。
    - Kali有自己配置好了DNS服务器地址所以可以上网。在Kali中`cat /etc/resolv.conf`可查看Kali默认DNS服务器地址。`nslookup 114.114.114.114`查看解析。给CentOS配置DNS服务器，比如把腾讯的DNS服务器配给它 `echo "nameserver 114.114.114.114">> /etc/resolv.conf`，配置完之后CentOS也可以上网了。

### 通过配置文件设定IP地址
1.  以上步骤配置完成后，当CentOS重启，IP地址还是会消失。所以我们需要通过配置文件设定IP地址。

2. 在CentOS中找到配置文件`cd /etc/sysconfig/network-scripts/`,里面的 `ifcfg=ens33`就是网卡配置文件。重新编写这个文件实现DHCP自动获取。由于权限问题，普通用户可以通过把原`ifcfg=ens33`改个名，然后自己创建一个`ifcfg=ens33`。
    - `mv ifcfg-ens33 ifcfg-ens33.bak` 把源文件改名。
    - `vim ifcft-ens33`
   写入：（配置IP地址是刚需，需要把下面四行命令背下来）
      ```
      DEVICE=ens33    
      TYPE=Ethernet
      ONBOOT=yes
      BOOTPROTO=dhcp
      ```
      - 设备是网卡，网卡名称是ens33
      - 类型是以太网卡
      - 允许对网络进程进行管理，如果这里是yes，就按照文件中所规定的获取ip地址的方式获取ip
      - 获取方式有三种：dhcp、static、none。后两者都是静态获取。
  
    - 上面只是改了配置文件，要DHCP生效需要重启网络服务，让系统读取配置文件`systemctl restart network`。配置完成后，每回CentOS虚拟机重启都会自动获取IP。

### CentOS配置静态IP
1. 和上面配置DHCP一样，需要改写配置文件，只是改写的内容不同。

2. 在CentOS中找到配置文件`cd /etc/sysconfig/network-scripts/`,改写 `ifcfg=ens33`文件。由于权限问题，普通用户可以通过把原`ifcfg=ens33`改个名，然后自己创建一个`ifcfg=ens33`。
    - `mv ifcfg-ens33 ifcfg-ens33.bak` 把源文件改名。
    - `vim ifcft-ens33`
   写入：（配置IP地址是刚需，需要把下面四行命令背下来）
      ```
      DEVICE=ens33    
      TYPE=Ethernet
      ONBOOT=yes
      BOOTPROTO=static
      IPADDR=x.x.x.x  //根据网段自己配置
      NETMASK=255.255.255.0 //或者 PREFIX=24
      GATEWAY=x.x.x.x
      DNS1=114.114.114.114 //DNS要写1 ，2编号
      DNS2=8.8.8.8 //可以有补备DNS服务器，8.8.8.8是谷歌
      ```

### Kali配置静态IP
> - 一般都用不上，因为Kali会DHCP自动获取，但会总比不会强。Kali是可以一个网卡两个IP的，因为自己本身会通过DHCP获取一个，你自己再配一个静态的，就会有两个，且两个都能用。
> - Kali的网络配置文件中不能配置DNS服务器地址，一般Kali会自己配置默认的DNS服务器，如果自己要配置，需要去 `/etc/resolv.conf`自己写。

1. Kali的网络配置文件在 `/etc/network/interfaces`。追加这个文件的内容: `sudo vim /etc/network/interfaces`:
    ```
    auto eth0
    iface eth0 inet static
    address x.x.x.x
    netmask 255.255.255.0
    gateway x.x.x.x
    ```
2. 重启网络服务 `sudo systemctl restart networking`

## 路由转发拓扑实验
> 需要实现192网段ping通172网段，以及中间涉及到了两个虚拟交换机vmnet2、vmnet3。客户机用的是win2003虚拟机。

![image](https://github.com/AlphaXiao/CTF-Linux-basics/blob/main/Days/pictures/4.png)

1. 查看win2003是否有VMnet2 和 VMnet3

    ![image](https://github.com/AlphaXiao/CTF-Linux-basics/blob/main/Days/pictures/5.png)
    
  - 如果没有VMnet2 和 VMnet3，需要手动建立。编辑→虚拟网络编辑器→更改设置→添加网络→添加VMnet2（把DHCP服务取消勾选→同样的方式建立VMnet3→确定
  
2. win2003（客户机）网络适配器要选VMnet2。针对客户机win2003确认连接的交换机为vmnet2同时配置ip。
    - 如果右下角没有网络图标，打开win2003，右键网上邻居→右键属性→右键本地连接→属性→勾选连接后在通知区域显示图标。
    - 配置IP地址：在同一个地方右键网上邻居→右键属性→右键本地连接→属性→双击TCP/IP→把IP地址、子网掩码、默认网关根据题目配置好。配置完可以win+R组合键，输入cmd，调出cmd输入ipconfig查看ip地址是否配置正确。

3. 建立克隆虚拟机centos7连接vmnet3同时配置ip地址
    - 克隆必须是基于关机快照，选择链接克隆。克隆机网络适配器选择VMnet3。根据上面的方法，配置静态IP。
      - 在CentOS中找到配置文件`cd /etc/sysconfig/network-scripts/`,改写 `ifcfg=ens33`文件。由于权限问题，普通用户可以通过把原`ifcfg=ens33`改个名，然后自己创建一个`ifcfg=ens33`。
      - `mv ifcfg-ens33 ifcfg-ens33.bak` 把源文件改名。
      - `vim ifcft-ens33`
      写入：
        ```
        DEVICE=ens33    
        TYPE=Ethernet
        ONBOOT=yes
        BOOTPROTO=static
        IPADDR=172.16.2.1 
        NETMASK=255.255.255.0 //或者 PREFIX=24
        GATEWAY=172.16.2.254
        DNS1=114.114.114.114 
        DNS2=8.8.8.8 
         ```
4. 为中间的网关服务器（原CentOS7虚拟机）配置ip地址和新网卡。
    - 添加多一个网卡：编辑虚拟机设置→点击添加→选网络适配器→确定。现在就有两个网络适配器了。
    - 一个网络适配器选VMnet2，另一个网络适配器选VMnet3。
    - 开机，先配置ens33的网络配置文件，再把ens33的配置文件复制粘贴一份。
      - 找到配置文件`cd /etc/sysconfig/network-scripts/`,改写 `ifcfg=ens33`文件。由于权限问题，普通用户可以通过把原`ifcfg=ens33`改个名，然后自己创建一个`ifcfg=ens33`。
      - `mv ifcfg-ens33 ifcfg-ens33.bak` 把源文件改名。
      - `vim ifcft-ens33`
      写入：（因为只是做实验，不用上网，所以可以不配置DNS、网关）
        ```
        DEVICE=ens33    
        TYPE=Ethernet
        ONBOOT=yes
        BOOTPROTO=static
        IPADDR=192.168.2.254 
        NETMASK=255.255.255.0 
         ```
    - 复制ens33拷贝成另一个网卡的网络配置文件。`cp ifcfg-ens33 ifcfg-ens34`。进入ens34文件修改设备名称和IP地址。
    - 重启网络服务：`systemctl restart network`。
    - 用 `ip addr` 查看是否配置正确。
    - 为了避免出现ens33接口连接到VMnet3上，ens34连接到Mnet2上这种错误。可以对比MAC地址。在`ip addr`里查看ens33的MAC地址，以太网接口的MAC地址是唯一的。打开中间CentOS7虚拟机的虚机设置，点击网络适配器（VMnet2）→高级→查看最下方MAC地址，对比看是否和前面查看的ens33的MAC地址相同。

5. 开启中间的网关服务器（原CentOS7虚拟机）的路由转发服务
    - 路由转发配置需要去内核配置文件更改。内核配置文件路径: `/etc/sysctl.conf`。
    - `vim /etc/sysctl.conf`写入：`net.ipv4.ip_forward = 1` 表示基于ipv4是否开启路由转发
    - 让内核控制文件立即生效: `sysctl -p`
 
## 日志服务 & 搭建日志服务器
1. 日志存放路径是 `cd /var/log/`，其中 `cd /var/log/secure`是登陆日志。
2. 日志服务是`rsyslog`，可查看进程`systemctl status rsyslog`，从进程信息中可以看出此进程是从开机便自动开启，会对系统中重要的进程的运行状态进行日记记录。进程都有配置文件，一个进程可能有多个配置文件。配置文件都在`/etc/`目录下。日志服务的配置文件是`/etc/rsyslog.conf`。可以分析进程配置文件，从而了解这些文件是如何记录的。
    - 例如，查看配置文件Rules部分中的`authpri.*`表示登录进程触发的日志记录。可以看到登陆日志是存放在`/var/log/secure`。
    - 可以做个小实验去异地实时备份登录日志信息，添加 `authpriv.*     /var/log/spare.txt`把登录日志信息同时存放进`/var/log/spare.txt`文档中。改完配置文件需要重启进程才会生效`systemctl restart rsyslog`。用`su -用户名`切换到其他用户，再`exit`返回root用户。`cat /var/log/spare.txt`，会看到刚刚切换用户的具体登陆信息。
    
        ![image](https://github.com/AlphaXiao/CTF-Linux-basics/blob/main/Days/pictures/6.png)
        
3. 日记级别分析。
    - 上面实验中的`authpri.*` 格式是 `进程名.日志级别`。
    - 日记有哪些级别可以通过查看man帮助得知 `man rsyslog.conf`。（注意：不是只有命令有man帮助手册，配置文件也有。配置文件的帮助手册就是教你如何配置的）
    
        ![image](https://github.com/AlphaXiao/CTF-Linux-basics/blob/main/Days/pictures/7.png)
        
    - 可以手动触发一下日志，除了刚刚用`su -用户`切换用户出发登陆日志外，还可以用 `logger -p 服务进程名.日志级别 "自己定义的触发事件名称"`，例如`logger -p authpriv.err "secure error"`。去 `tail -5 /var/log/secure`查看刚刚触发的事件。
    - 注意：emerg是日志级别中最紧急的，一旦触发，在屏幕上会立即显示。        

## ssh登录服务器后清理痕迹
1. `systemctl stop syslog` 或者 `service rsyslog stop`（后者是CentOS6的关闭方法）把日志服务给停了。把日志备份文件删除。
2. 高级一点的方式 `echo "" > secure` 直接把登录日志清空。但实际上还有很多文件会记录登录日志，比如wtmp文件，可通过`last -f wtmp`查看。还有apache、nginx web服务的日志（访问日志 & 错误日志会记录ip）、history历史记录都会有所记录，都需要清理。历史记录的查看方式是`history`，清理方式是`history -c`。

## 建立日志服务器实现针对以上问题的日志异地备份（备份登录日志）
> 继续用上面路由转发拓扑实验的虚拟机做日志服务器实验。

![image](https://github.com/AlphaXiao/CTF-Linux-basics/blob/main/Days/pictures/8.png)

1. 配置日志客户机
    - 修改日志服务的配置文件 `vim /etc/rsyslog.conf` ，配置完一定要**重启服务**才能生效 `systemctl restart rsyslog`。
    
        ![image](https://github.com/AlphaXiao/CTF-Linux-basics/blob/main/Days/pictures/9.png)
        
2. 配置日志服务器
    -  修改日志服务的配置文件 `vim /etc/rsyslog.conf`，在最后添加规则 `:fromhost-ip, isequal, "对方IP" /var/log/存放文件夹名/存放文件.log`，例如:`:fromhost-ip, isequal, "172.16.2.254 /var/log/mylog/172.16.2.254.log`表示根据IP 172.16.2.254接收，接收到的日志保存到 `/var/log/mylog/172.16.2.254.log`中。
        -   ```
            格式
            :属性, 匹配符号, "值" 保存位置
            
            属性：
            fromhost-ip 按照ip地址来接收
            fromhost 按照发送过来的主机名接收（dns）
            msg 按照日志内容接收
            hostname 按照日志内容中的主机名接收
            
            匹配符号：
            isequal 等于 “172.16.2.254”
            startswith 以什么开头 “172.16.2.”
            contains 包含 “172” 
            ```
    -  开启TCP协议以及514端口。取消配置文件第19，20行的注释就可以。Vim编辑器末行模式下用`:set nu`可以显示行号。
    
        ![image](https://github.com/AlphaXiao/CTF-Linux-basics/blob/main/Days/pictures/10.png)
        
    -  配置完一定要**重启服务**才能生效 `systemctl restart rsyslog`。
    -  查看514端口是否开启，查看端口号是用 `ss -antpl`，其中，-a 选项表示显示所有连接，-n 选项表示以数字形式展示地址和端口号，-t 选项表示只显示 TCP 连接，-p 选项表示显示连接所属的进程信息，-l 选项表示只显示监听状态的连接。

3. 验证日志是否备份
    - 比如在win2003上装一个Xshell，远程登陆CentOS客户机。再去日志服务器（克隆CentOS）查看 `cd /var/log`，里面会多出`mylog`文件夹及`172.16.2.254.log`日志备份文件。






















    





