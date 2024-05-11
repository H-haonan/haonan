先看是几个节点的集群，需要在每个节点执行以下操作。
git clone https://github.com/yuyicai/update-kube-cert.git
cd update-kube-cert
chmod 755 update-kubeadm-cert.sh
./update-kubeadm-cert.sh all --cri docker
执行完需要执行重启docker和kubelet的操作
systemctl restart docker
systemctl restart kubelet