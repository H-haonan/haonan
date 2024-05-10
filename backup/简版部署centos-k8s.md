简版
根据提供的patch文件生成talos的三个部署文件。
talosctl gen config mingyang https://172.16.102.102:6443/  --config-patch-control-plane @patch.yaml
注意：需要修改里面的地址信息以及网口信息。

根据console口显示的dhcp分配的地址下发配置文件。
talosctl apply-config --insecure -n 172.16.102.188 --file controlplane.yaml 
注意： IP地址为dhcp分配的地址

修改talosconfig配置文件，里面的endpoint和nodes需要改成节点地址
执行：talosctl config merge ./talosconfig 使talos配置合并到默认配置
位置：/root/.talos/config

同理，修改kubeconfig的配置
执行： talosctl kubeconfig 
注意：这块执行后如果之前有配置直接覆盖就行提示选俩o

查看talosctl service状态，etcd显示prepraing时候执行
talosctl bootstrap --nodes 172.16.102.188 地址为节点ip

Kubectl get pods -A 当pod状态都正常运行的时候
在当前文件夹下导入最新的cncp-helm配置文件
部署顺序，cncp-basic-services第一个部署 cncp-core-compose最后部署，其他业务按情况在之间部署即可

部署cncp-basic-service
首先需要修改位于/cncp-helm/cncp-basic-services/values文件，修改里面的metallb地址池ipAddressPool:   修改后wq保存退出

在cncp-helm文件夹下执行helm install cncp-basic-services ./cncp-basic-services/ --debug部署，
若想卸载执行helm uninstall cncp-basic-services

部署ddi业务：
同理也需要修改cncp-helm/ddi-components/values文件，里面有enable的选项，若是需要部署就enable: true 若没有该业务enable: false 即可
这里面需要修改业务地址，dns下面的clusterNetworkIp4ServiceAddress代表dns业务的ipv4地址，这块注意需要从规划的metallb地址池里面选择，下面地址同理。
clusterNetworkIp6ServiceAddress为dns的ipv6业务地址。
Dhcp4,dhcp6同理。


部署nginx业务
只需要部署nginx-deployment.yaml即可，kubectl create -f nginx-deployment.yaml
部署siit,nat,sipalg情况一样，他们都是macvlan形式的
部署顺序是先是configmap配置文件再是部署文件最后探测业务是否好使
对应的文件夹下配置文件为：cm文件，deploy文件，networkavailabilityprobe
部署的时候只需要修改deploy文件里面的有关于macvlan的地址情况即可
注意修改地址信息和位于master的网口名，下面还有环境变量的网关信息别忘了改
env:
        - name: IPV4_DEFAULT_GATEWAY
          value: 172.16.201.1
        - name: IPV6_DEFAULT_GATEWAY
          value: 2408:8631:c02:ffb1::1
这三个部署方式一样，同理部署即可。

最后部署前端dashboard，即cncp-core-components文件
部署方式同理，先修改value里面的内容这块上面的cluster:集群的证书文件需要整体替换为/root/.kube/config的配置，只需要将ip地址修改为域名即可https://cncp-ms-01:6443
注意缩进
还需要修改下面的业务地址
ddi:
  dnsClusterNetworkIp4ServiceAddress: 172.16.102.240
  dnsClusterNetworkIp6ServiceAddress: 2408:8631:c02:ffa2::240
  dhcp4ClusterNetworkIp4ServiceAddress: 172.16.102.241
  dhcp6ClusterNetworkIp6ServiceAddress: 2408:8631:c02:ffa2::241
分别对应dns的四，六以及dhcp的四，六地址（与部署ddi-components的时候选择的metallb地址池下的业务地址一致）

注意发货前修改下面的imagePullPolicy，都改成IfNotPresent。

基础网络部署
Kubectl create -f talos-dashboard.yaml 这个必须部署要不dashboard的pod会起不来，这个pod部署完需要龙哥或者我这边进行添加数据。

最后一步有一个kube-fledged的文件夹，发货前需要先部署一下kube-fledged.yaml 再部署kube-fledged-ic.yaml 主要作用是将镜像文件传到每一台服务器上，不部署去现场有可能会有pod转移到其他节点镜像拉取不到的情况。