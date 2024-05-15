配置根据官方文档配置：
machine:
    disks:
        - device: /dev/sdb # The name of the disk to use.
          # A list of partitions to create on the disk.
          partitions:
            - mountpoint: /var/mnt/extra # Where to mount the partition.
需要注意的是：
sdb硬盘需要清空并格式化
方法：
https://www.system-rescue.org/Download/
去上面链接选择一个下载ios镜像做系统盘。
然后进入系统盘进行格式化分区操作
fdisk -l /dev/sdb   命令将会显示指定设备（在本例中为 /dev/sdb）的分区表信息
fdisk /dev/sdb   命令会启动交互式的分区编辑器，用于管理指定磁盘上的分区
o 回车
n 回车
p 回车
1 回车
回车
回车
y
p 回车 
看分区对不对没问题的话w保存
 mkfs.xfs /dev/sdb1
然后重启加上官方配置。
