#1.首先需要一个http服务器，通过docker或者在centos上直接yum install 部署都可以
docker run -d -p 8081:80 -v /var/www:/usr/local/apache2/htdocs httpd
这个docker命令会创建并运行一个httpd容器,参数说明如下:
- `-d`: 后台运行容器
- `-p 8081:80`: 映射容器内80端口到宿主机8081端口
- `-v /var/www:/usr/local/apache2/htdocs`: 把宿主机/var/www目录挂载到容器的/usr/local/apache2/htdocs,作为网页文档根目录
- `httpd`: 使用httpd镜像启动容器
这样就在宿主机8081端口启动了一个httpd服务器,网页文档来自宿主机的/var/www目录。
访问方式:
在宿主机浏览器访问 http://宿主机IP:8081 
就可以访问容器中的httpd服务。
并且我们可以通过挂载目录,来修改宿主机/var/www下的文件,即可实时更新httpd服务器中的网页文件。
需要注意:
1. 宿主机的/var/www目录需要存在,且有一定的访问权限。
2. 容器内部的httpd配置不要和宿主机8081端口冲突。
3. 可以添加-v宿主机日志目录:容器日志目录来收集日志。
4. 可以设置时区等参数对容器进行定制。
5. mkdir /var/www/charts #宿主机创建挂载目录
#2. Helm chart上传到私有仓库
Helm chart上传到私有仓库
helm package mychart/
将mychart目录打包成tgz文件,用于发布。
mkdir myrepo
创建文件夹作为chart仓库目录。 
mv mychart-0.1.0.tgz myrepo/
将打包好的chart移动到仓库目录下。
helm repo index myrepo --url http://172.16.106.11:8088/mingyang/bulid/
在myrepo目录生成index.yaml索引文件,用于仓库查询。
cd myrepo/
scp -r cncp-core-components-v2.3.1.tgz  index.yaml  root@172.16.106.11:/usr/local/httpd/file/mingyang/bulid
将index和chart复制到Web服务器目录下,提供下载。
helm repo remove newrepo
helm repo add newrepo http://172.16.106.11:8088/mingyang/bulid/
在本地Helm中添加这个chart仓库源,名为newrepo。
helm repo list
列出已知的仓库列表,确保newrepo添加成功。
helm repo update 
更新本地缓存的仓库index文件。
helm search repo mychart
在newrepo仓库中搜索mychart是否可用。
helm template cncp-core-components newrepo/cncp-core-components --output-dir cncp-core-components-150
至此,我们就利用Web服务器创建了一个本地的Helm chart仓库,可以通过add repo和update的方式使用了。
#3.自行创建nginx Helm chart方法
自行创建nginx Helm chart方法

helm create nginx-chart
使用helm create命令生成一个名为nginx-chart的chart模板。

vim nginx-chart/values.yaml
编辑values.yaml文件,用于模板中的值替换。

vim nginx-chart/templates/service.yaml
编辑模板文件service.yaml,定义Kubernetes服务。

helm lint nginx-chart/
使用helm lint对chart进行语法检查,确保没有错误。

helm install mynginx ./nginx-chart/
使用helm install命令,以mynginx为release名称安装nginx-chart。

kubectl get pods,svc
获取pod和服务,检查helm安装是否成功。

这样通过自定义chart模板和values,我们就可以打包自己的应用 charts。helm lint和install命令可以确保chart语法正确并能被正常安装。

然后就可以将chart发布到仓库,或者与CI/CD系统集成,实现Kubernetes的声明式部署。
#4. 把chart包转成yaml
helm template vault hashicorp/vault --output-dir vault-manifests/helm-manifests