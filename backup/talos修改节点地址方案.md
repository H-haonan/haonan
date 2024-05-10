一.先添加新节点地址ip
talosctl edit mc 修改机器配置，在服务器接口地址那块加上客户现场地址的ip和网关（每台都改）
  addresses:
   - 172.16.100.151/24  #之前集群地址
   - 2408:8631:c02:ffa0::151/64 #之前集群地址
  - 172.16.102.151/24 #需要添加客户现场地址
   - 2408:8631:c02:ffa2::151/64 #需要添加客户现场地址
   bond:
   deviceSelectors:
   - busPath: "0000:07:00.0"
   - busPath: "0000:08:00.0"
   miimon: 10
   mode: active-backup
   interface: bond-metallb
   mtu: 1500
   routes:
   - gateway: 172.16.102.1 #需要修改的客户现场网关
   network:  0.0.0.0/0
   - gateway: 2408:8631:c02:ffa2::1  #需要修改的客户现场网关
   network: ::/0 
在cluster/etcd下添加俩配置
  advertisedSubnets: # listenSubnets defaults to advertisedSubnets if not set explicitly
   - 172.16.102.0/24 #和客户现场保持一致
   listenSubnets:
   - 172.16.102.0/24 #和客户现场保持一致
修改cluster/kubelet下nodeIP:
   nodeIP# The validSubnets field configures the networks to pick kubelet node IP from.
   validSubnets:
   - 172.16.102.0/24 #和客户现场保持一致
修改完后保存退出，看通过客户现场的地址是否能连接集群并看pod状态是否都正常。
二. 修改集群里面所有的旧地址，可以通过全局搜索替换修改   :%s/172.16.100.151/172.16.102.151/g
这块注意需要保留上面interface那块的之前的旧集群地址
三.按照节点三二一的顺序执行重启操作。
四.重启后观察talosctl service的状态，查看pod状态无问题后执行talosctl edit mc 将interface那块之前的旧地址删除掉。