# 说明
- [x]  学习工具：在VMware里装了CentOS7、Kali和win2003三台虚拟机，使用这三台机子做实验。
- [x]  Linux基础：开始学习前已经了解了Linux的基本命令以及部分软件的使用例如vim、nano。
- [x]  虚拟机配置：Kali有配置好源、下载了Googlepinyin、下载了VM-tools。CentOS永久关闭了防火墙`systemctl disable firewall.service`。关闭了虚拟网卡服务 `systemctl disable libvirtd.service`，这段命令输完之后需要重启虚拟机才会关闭虚拟网卡服务，可以用`systemctl reboot`重启。
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
      - 具体怎么手动配置可以查看帮助文档 `ip addr help`。从上面Kali的IP地址中看到网段，这里自己配置CentOS的IP需要和Kali的IP在同一网段内。子: `ip addr add 192.168.226.200/24 dev ens33`。 
      
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

### 路由转发拓扑实验
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

































    





