k8s集群安装部署（先决条件）
一. 添加 Helm Chart 仓库
请将命令中的<CHART_REPO>，替换为latest，stable或alpha。更多信息，请查看[选择 Rancher 版本](https://docs.rancher.cn/docs/rancher2.5/installation/resources/choosing-version/_index)来选择最适合您的仓库。

latest: 建议在尝试新功能时使用。
stable: 建议在生产环境中使用。（推荐）
alpha: 未来版本的实验性预览。
helm repo add rancher-<CHART_REPO> https://releases.rancher.com/server-charts/<CHART_REPO>
国内用户，可以使用放在国内的 Rancher Chart 加速安装：
helm repo add rancher-<CHART_REPO> https://rancher-mirror.rancher.cn/server-charts/<CHART_REPO>

二. 创建命名空间
kubectl create namespace cattle-system

三. 使用 Rancher 生成的自签名证书
helm install rancher rancher-<CHART_REPO>/rancher \
 --namespace cattle-system \
 --set hostname=rancher.my.org

等待 Rancher 运行：

kubectl -n cattle-system rollout status deploy/rancher
Waiting for deployment "rancher" rollout to finish: 0 of 3 updated replicas are available...
deployment "rancher" successfully rolled out

四. 通过nodeport访问
执行kubectl edit svc rancher -n cattle-system
把：spec下的type由ClusterIp修改为 NodePort，保存退出，
执行 kubectl get svc -n cattle-system 获取rancher 对外暴露端口
然后随便访问一个 节点IP+":"" + rancher对外暴露的端口/dashboard/ 即可访问rancher的web界面

五. 获取认证token
kubectl -n cattle-system exec $(kubectl -n cattle-system get pods -l app=rancher | grep '1/1' | head -1 | awk '{ print $1 }') -- reset-password

详情可参照rancher官网：https://docs.rancher.cn/docs/rancher2/installation/install-rancher-on-k8s/_index/#%E5%85%88%E5%86%B3%E6%9D%A1%E4%BB%B6