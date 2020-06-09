# CentOS 7 系统管理员指南

# 	一、配置IP地址

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

   

# 二、配置Selinux和Firewall

​	Selinux

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

firewall

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

   

