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

1. 关闭Selinux服务  

   ```
   # vi /etc/selinux/config
   SELINUX=enforcing更改为SELINUX=disabled
   # reboot 重启生效
   
   ```

2. 