# CentOS 7 系统管理员指南

目录

一、配置IP地址和SSH

二、开机启动级别

三、配置Selinux和Firewall

四：系统安装

五、将Samba设置为域成员



# 	一、配置IP地址和SSH

## 配置IP地址

1. 列出网络适配器

   ```
   # nmcli -p dev
   ```

   

2. 设置网络参数 

   ```
    # vi /etc/sysconfig/network-scripts/ifcfg-eth0   
   ```

3. 设置DNS

   ```
   # vi /etc/resolv.conf
   ```
   
## 配置SSH

1. 打开/etc/ssh/sshd_config文件

   ```
   # vi /etc/ssh/sshd_config
   ```

2. 使root具备ssh登录的权限，去掉#注释

   ```
   #PermitRootLogin yes
   ```

   改为

   ```
   PermitRootLogin yes
   ```

3. 启用22号端口，去掉前面的注释

   ```
   #Port 22
   ```

   改为

   ```
   Port 22
   ```

4. 防火墙放行ssh服务并重新加载防火墙（永久生效）

   ```
   # firewall-cmd --add-service=ssh --permanent
   # firewall-cmd --reload
   ```

   若已更改端口号，采用如下命令放行（永久生效）

   ```
   # firewall-cmd --add-port=10022/tcp --permanent
   # firewall-cmd --reload
   ```

   

5. 重启sshd服务

   ```
   # systemctl restart sshd
   ```




# 二、开机启动级别

开机默认进入命令行模式

~~~
systemctl set-default multi-user.target
~~~

开机默认进入图形用户界面

~~~
systemctl set-default graphical.target
~~~



# 三、配置Selinux和Firewall

## 	Selinux

1. 关闭Selinux服务  （不推荐）

   ```
   # vi /etc/selinux/config
   SELINUX=enforcing 更改为 SELINUX=disabled
   # reboot 重启生效
   ```

2. 查看SElinux状态  

   ```
   # sestatus
   ```

   或

   ```
   # getenforce
   ```

   

3. 例如，修改SSH端口为10022，则如下命令修改端口的上下文类型

   ```
   # semanage port -a -t ssh_port_t -p tcp 10022
   # semanage port -l | grep ssh
   ```

   ### 额外说明：

   |    名称    |         模式         |            作用            |
   | :--------: | :------------------: | :------------------------: |
   | enforcing  |       强制模式       |   拒绝非法访问并录入日志   |
   | permissive | 许可模式（警告模式） | 暂时允许非法访问并录入日志 |
   |  disabled  |       禁用模式       |  允许非法访问且不录入日志  |

   临时关闭selinux策略
   
   ```
   # setenforce 0
   ```
   
   临时开启selinux策略
   
   ```
   # setenforce 1
   ```

## firewall

1. 开启firewall服务  

   ```
   # systemctl start firewalld
   ```

   

2. 关闭firewall服务

   ```
   # systemctl stop firewalld
   ```

3. 禁止开机启动firewall服务  

   ```
   # systemctl disable firewalld
   ```

4. 允许开机启动firewall服务  

   ```
   # systemctl enable firewalld
   ```

5. 查看firewall状态  

   ```
   # systemctl status firewalld
   ```

   

# 四、系统安装

1. 将分区表设置为gpt格式
   1. 将光标移到【install CentOS 7】位置
   2. 按下【Tab】键，将光标移至行末
   3. 输入关键词 inst.gpt 回车
2. 磁盘分区
   1. 选择自动建立分区
   2. 选择手动建立分区


# 五、将Samba设置为域成员

## 加入域

1. 防火墙放行samba服务

   ```
   # firewall-cmd --permanent --add-service=samba
   ```

2. 将CentOS的DNS指向域控制器

   ```
   # vi /etc/resolv.conf
   ```

3. 使CentOS从域控制器同步时间，例如，域控制器IP地址为：192.168.253.130

   ```
   # ntpdate 192.168.253.130
   ```

4. 安装以下软件包

   ```
   # yum install realmd oddjob-mkhomedir oddjob samba-winbind-clients samba-winbind samba-common-tools -y
   ```

5. 要共享域成员上的目录或打印机，安装samba软件包

   ```
   # yum install samba -y
   ```

6. 加入AD，需另外安装samba-winbind-krb5-locator软件包

   ```
   # yum install samba-winbind-krb5-locator -y
   ```

7. 重命名现有的/etc/samba/smb.conf Samba配置文件

   ```
   # mv /etc/samba/smb.conf /etc/samba/smb.conf.old
   ```

8. 加入域， 例如，要加入一个名为*ad.example.com*的域

   ```
   # realm join --membership-software=samba --client-software=winbind ad.example.com
   ```

9. 验证winbindd是否正在运行

   ```
   # systemctl start winbind
   # systemctl status winbind
   ```

10. 启动smbd服务

   ```
   # systemctl start smb
   # systemctl status smb
   ```

   ## 验证Samba是否已作为域成员正确加入

11. 查询AD域的管理员帐号administrator

    ```
    # getent passwd ad.example.com\\administrator
    ```

12. 查询AD域中“ Domain Users”组的成员

    ```
    # getent group "ad.example.com\\Domain Users"
    ```

13. 如果以上命令可以正常执行，可以使用域用户和组设置文件和目录的权限。

    例如，设置/root/svr/example.txt文件的所有者ad.example.com\administrator和组ad.example.com\Domain Users用户

    ```
    # mkdir -p /srv/samba/example/
    # chown "ad.example.com\administrator":"ad.example.com\Domain Users" /root/svr/example.txt
    ```

14. 例如，要将/ srv / samba / example /目录的所有者设置为root用户，请对Domain Users组授予读写权限，并拒绝对所有其他用户的访问：

    ```
    # chown root:"ad.example.com\Domain Users" /srv/samba/example/
    # chmod 2770 /srv/samba/example/
    ```

15. 在Samba服务器上配置文件共享，在smb.conf中添加如下配置信息

    ```
    # vim /etc/samba/smb.conf
    [example]
    path = /srv/samba/example/
    read only = no
    ```

16. 在windows计算机进行访问，例如，samba服务器地址为：192.168.253.100

    ```
    \\192.168.253.100
    ```

17. 关于 强制位（s权限）和粘滞位（t权限）说明：

    -  S_ISUID、S_ISGID两个常量，叫做强制位权限，作用在于设置使文件在执行阶段具有文件所有者的权限，相当于临时拥有文件所有者的身份 ，一个文件运行时的身份，可以分别通过以下命令来设置s权限：

      ```
      chmod u+xs filename 
      chmod u-s filename 
      chmod g+xs filename 
      chmod g-s filename 
      ```

      在设置s权限时文件属主、属组必须先设置相应的x权限，否则s权限并不能正真生效（chmod命令不进	行必	要的完整性检查，即使不设置x权限就设置s权限，chmod也不会报错，当我们ls -l时看到rwS，	大写S说明s	权限未生效）

    

    -  S_ISVTX 使一个目录既能够让任何用户写入文档，又不让用户删除这个目录下他人的文档，t权限就是能起到这个作用。t权限一般只用在目录上，用在文档上起不到什么作用
      在一个目录上设了t权限位后，（如/home，权限为1777)任何的用户都能够在这个目录下创建文档，但只能删除自己创建的文档(root除外)，这就对任何用户能写的目录下的用户文档 启到了保护的作用，可以通过以下命令来设置t权限

      ```
      chmod +t filename
      chmod -t filename
      ```

18. 



