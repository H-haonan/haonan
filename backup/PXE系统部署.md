PXE系统部署
✧ 使用http用来发布linux系统的安装树（也可以使用NFS、FTP server或HTTPS）
✧ DHCP server为客户端分配ip并提供TFTP服务器地址及PXE启动文件位置
✧ TFTP server为客户端提供引导文件。

三个服务可以安装在同一台服务器上，也可以安装在三台服务器上。

1、安装配置http
安装httd服务： yum install httpd -y ，安装完成之后可以使用 yum info httpd 或者使用 rpm -qa | grep httpd 查看安装信息
查看httpd服务是否运行： systemctl status httpd
启用httpd服务： systemctl start httpd 
关闭防火墙：systemctl stop firewalld.service 
查看http服务器配置vim /etc/httpd/conf/httpd.conf 
修改http服务器文件存放位置DocumentRoot "/usr/local/httpd/file"  服务器监听端口 Listen 8088
修改配置后重启http服务器并将所需要的yaml文件放入/usr/local/httpd/file路径下。
2、安装配置DHCP server
yum -y install dhcp 
编辑配置文件，这里是很重要的。
 vim /etc/dhcp/dhcpd.conf 
实例：

ddns-update-style none;
subnet 172.16.200.0 netmask 255.255.255.0  //配置网段
{    
  range 172.16.200.201 172.16.200.240;          //配置地址池
  option routers 172.16.200.1;                  //配置网关
  default-lease-time 600;
  max-lease-time 7200;
  filename"ipxe.efi";                         //指定pxe引导程序的文件名 
  next-server 172.16.200.252;                   //指定tftp服务器的地址
}

systemctl start dhcpd  #开启服务
systemctl status dhcpd  #查看服务状态（running）

3、安装配置TFTP server  //服务器通过TFTP（Trivial File Transfer Protocol，简单文件传输协议）提供引导镜像文件的下载.

安装TFTP，然后编辑配置文件，开启开启服务，默认的数据目录/var/lib/tftpboot
yum -y install tftp-server xinetd   //安装tftp（tftp依赖于xinetd，所以也要安装xinetd）

允许外界访问tftp
sudo firewall-cmd --add-service=tftp


vi /etc/xinetd.d/tftp 
Service    tftp
{
socket_type             = dgram
protocol                = udp
wait                    = yes
user                    = root
server                  = /usr/sbin/in.tftpd
server_args             = -s /var/lib/tftpboot
disable                 =no   // #将disable=yes 修改为no,no表示开启TFTP服务;
per_source              = 11
cps                     = 100 2
flags                   = IPv4
}
systemctl start xinetd
systemctl start tftp 
systemctl status tftp 查看服务状态是否在运行


4、安装syslinux：  //用来提供pxe的引导程序
yum -y install syslinux
rpm -ql syslinux | grep pxelinux   //查找pxe引导程序的位置

将pxelinux.0拷贝到tftpboot目录下
cp /usr/share/syslinux/pxelinux.0  /var/lib/tftpboot/ 
在/var/lib/tftpboot/ 中新建一个pxelinux.cfg目录
mkdir /var/lib/tftpboot/pxelinux.cfg

将talos系统的启动所需的文件复制到/var/lib/tftpboot/ 目录下（以下红色标注的2个文件）：
ls /var/lib/tftpboot/ 
ipxe.efi          （这是一个pxe引导文件）
pxelinux.0        （这文件是为legcay启动，它是legcay的启动镜像） 
pxelinux.cfg      （该文件夹下放的是启动菜单，手动创建）



