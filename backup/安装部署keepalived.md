1. 安装部署keepalived（Master和Slave两机器同样操作）
1）安装keepalived
# yum -y install keepalived

2）Master节点的keepalived.conf配置
这里特别需要注意：
一定要设置keepalived为非抢占模式，如果设置成抢占模式会在不断的切换主备时容易造成NFS数据丢失。

# cp /etc/keepalived/keepalived.conf /etc/keepalived/keepalived.conf_bak
# >/etc/keepalived/keepalived.conf
# vim /etc/keepalived/keepalived.conf
! Configuration File for keepalived
global_defs {
    router_id master   #id可以随便设
}
vrrp_script chk_nfs {
    script "/etc/keepalived/nfs_check.sh"    #监控脚本
    interval 2
    weight -20   #keepalived部署了两台，所以设为20，如果三台就设为30
}
vrrp_instance VI_1 {
    state BACKUP    #两台主机都设为backup非抢占模式
    interface eth0  #网卡名写自己的，不要照抄
    virtual_router_id 51
    priority 100    #master设为100，backup设为80，反正要比100小
    advert_int 1
    nopreempt       #设置为非抢占模式必须要该参数
    authentication {
        auth_type PASS
        auth_pass 1111
    }   
    track_script {
        chk_nfs
    }
    virtual_ipaddress {
        172.16.60.244      #虚拟ip
    }
}

3）Slave节点的keepalived.conf配置
只需将priority参数项修改为80，其他配置的和master节点一样，脚本也一样。

# cp /etc/keepalived/keepalived.conf /etc/keepalived/keepalived.conf_bak
# >/etc/keepalived/keepalived.conf
# vim /etc/keepalived/keepalived.conf
! Configuration File for keepalived
global_defs {
    router_id master   
}
vrrp_script chk_nfs {
    script "/etc/keepalived/nfs_check.sh"    
    interval 2
    weight -20   
}
vrrp_instance VI_1 {
    state BACKUP    
    interface eth0  
    virtual_router_id 51
    priority 80   
    advert_int 1
    nopreempt       
    authentication {
        auth_type PASS
        auth_pass 1111
    }   
    track_script {
        chk_nfs
    }
    virtual_ipaddress {
        172.16.60.244      
    }
}


4）编辑nfs_check.sh监控脚本
# vim /etc/keepalived/nfs_check.sh
#!/bin/bash
A=`ps -C nfsd --no-header | wc -l`
if [ $A -eq 0 ];then
        systemctl restart nfs-server.service
        sleep 2
        if [ `ps -C nfsd --no-header| wc -l` -eq 0 ];then
            pkill keepalived
        fi
fi

设置脚本执行权限
# chmod 755 /etc/keepalived/nfs_check.sh

5）启动keepalived服务
# systemctl restart keepalived.service && systemctl enable keepalived.service

查看服务进程是否启动
# ps -ef|grep keepalived

6）检查vip是否存在
在两台节点机器上执行"ip addr"命令查看vip，其中会在一台机器上产生vip地址。
# ip addr|grep 172.16.60.244
inet 172.16.60.244/32 scope global eth0

0.3 安装部署Rsync+Inofity（Master和Slave两机器都要操作）
1）安装rsync和inotify
# yum -y install rsync inotify-tools

2）Master节点机器配置rsyncd.conf
# cp /etc/rsyncd.conf /etc/rsyncd.conf_bak
# >/etc/rsyncd.conf
# vim /etc/rsyncd.conf
uid = root
gid = root
use chroot = 0
port = 873
hosts allow = 172.16.60.0/24  #允许ip访问设置，可以指定ip或ip段
max connections = 0
timeout = 300
pid file = /var/run/rsyncd.pid
lock file = /var/run/rsyncd.lock
log file = /var/log/rsyncd.log
log format = %t %a %m %f %b
transfer logging = yes
syslog facility = local3

[master_web]
path = /data/k8s_storage
comment = master_web
ignore errors
read only = no   #是否允许客户端上传文件
list = no
auth users = rsync  #指定由空格或逗号分隔的用户名列表，只有这些用户才允许连接该模块
secrets file = /etc/rsyncd.passwd  #保存密码和用户名文件，需要自己生成

编辑密码和用户文件（格式为"用户名:密码"）
# vim /etc/rsyncd.passwd
rsync:123456

编辑同步密码（注意这个文件和上面的密码和用户文件路径不一样）
该文件内容只需要填写从服务器的密码，例如这里从服务器配的用户名密码都是rsync:123456，则主服务器则写123456一个就可以了
# vim /opt/rsyncd.passwd
123456

设置文件执行权限
# chmod 600 /etc/rsyncd.passwd
# chmod 600 /opt/rsyncd.passwd

启动服务
# systemctl enable rsyncd && systemctl restart rsyncd

检查rsync服务进程是否启动
# ps -ef|grep rsync

3）Slave节点机器配置rsyncd.conf
就把master主机/etc/rsyncd.conf配置文件里的[master_web]改成[slave_web]
其他都一样，密码文件也设为一样

# cp /etc/rsyncd.conf /etc/rsyncd.conf_bak
# >/etc/rsyncd.conf
# vim /etc/rsyncd.conf
uid = root
gid = root
use chroot = 0
port = 873
hosts allow = 172.16.60.0/24
max connections = 0
timeout = 300
pid file = /var/run/rsyncd.pid
lock file = /var/run/rsyncd.lock
log file = /var/log/rsyncd.log
log format = %t %a %m %f %b
transfer logging = yes
syslog facility = local3

[slave_web]
path = /data/k8s_storage
comment = master_web
ignore errors
read only = no
list = no
auth users = rsync
secrets file = /etc/rsyncd.passwd

编辑密码和用户文件（格式为"用户名:密码"）
# vim /etc/rsyncd.passwd
rsync:123456

编辑同步密码
# vim /opt/rsyncd.passwd
123456

设置文件执行权限
# chmod 600 /etc/rsyncd.passwd
# chmod 600 /opt/rsyncd.passwd

启动服务
# systemctl enable rsyncd && systemctl restart rsyncd

检查rsync服务进程是否启动
# ps -ef|grep rsync

