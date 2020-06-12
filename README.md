# CentOS 7 系统管理员指南

[TOC]

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

4. 防火墙放行ssh服务及重新加载防火墙

   ```
   firewall-cmd --permanent --add-service=samba
   ```

5. 重启sshd服务

   ```
   # systemctl restart sshd
   ```

# 二、配置Selinux和Firewall

## 	Selinux

1. 关闭Selinux服务  

   ```
   # vi /etc/selinux/config
   SELINUX=enforcing更改为SELINUX=disabled
   # reboot 重启生效
   ```

2. 查看Selinux状态  

   ```
   # sestatus
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

6. 防火墙放行ssh

   ```
   # firewall-cmd --permanent --add-service=ssh
   ```

   

# 三、将Samba设置为域成员

## 加入域

1. 将CentOS的DNS指向域控制器

   ```
   # vi /etc/resolv.conf
   ```

2. 使CentOS从域控制器同步时间，例如，域控制器IP地址为：192.168.253.130

   ```
   # ntpdate 192.168.253.130
   ```

3. 安装以下软件包

   ```
   # yum install realmd oddjob-mkhomedir oddjob samba-winbind-clients samba-winbind samba-common-tools -y
   ```

4. 要共享域成员上的目录或打印机，安装samba软件包

   ```
   # yum install samba -y
   ```

5. 加入AD，需另外安装samba-winbind-krb5-locator软件包

   ```
   # yum install samba-winbind-krb5-locator -y
   ```

6. 重命名现有的/etc/samba/smb.conf Samba配置文件

   ```
   # mv /etc/samba/smb.conf /etc/samba/smb.conf.old
   ```

7. 加入域， 例如，要加入一个名为*ad.example.com*的域

   ```
   # realm join --membership-software=samba --client-software=winbind ad.example.com
   ```

8. 验证winbindd是否正在运行

   ```
   # systemctl status winbind
   ```

9. 启动smbd服务

   ```
   # systemctl start smb
   ```

   ## 验证Samba是否已作为域成员正确加入

1. 查询AD域的管理员帐号

   ```
   # getent passwd ad.example.com\administrator
   ```

2. 