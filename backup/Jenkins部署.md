Jenkins部署
基于centos7环境，默认docker已经安装
以docker方式部署jenkins
启动安装
1.启动docker，下载Jenkins镜像文件
docker pull jenkins/jenkins
 2.创建Jenkins挂载目录并授权权限
（我们在服务器上先创建一个jenkins工作目录 /var/jenkins_mount，赋予相应权限，稍后我们将jenkins容器目录挂载到这个目录上，这样我们就可以很方便地对容器内的配置文件进行修改。 如果我们不这样做，那么如果需要修改容器配置文件，将会有点麻烦，因为虽然我们可以使用docker exec -it --user root 容器id /bin/bash 命令进入容器目录，但是连简单的 vi命令都不能使用）
mkdir -p /var/jenkins_mount
chmod 777 /var/jenkins_mount
3.创建并启动Jenkins容器
　　-d 后台运行镜像
　　-p 10240:8080 将镜像的8080端口映射到服务器的10240端口。
　　-p 10241:50000 将镜像的50000端口映射到服务器的10241端口
　　-v /var/jenkins_mount:/var/jenkins_mount /var/jenkins_home目录为容器jenkins工作目录，我们将硬盘上的一个目录挂载到这个位置，方便后续更新镜像后继续使用原来的工作目录。这里我们设置的就是上面我们创建的 /var/jenkins_mount目录
　　-v /etc/localtime:/etc/localtime让容器使用和服务器同样的时间设置。
　　--name myjenkins 给容器起一个别名
docker run -d -p 10240:8080 -p 10241:50000 -v /var/jenkins_mount:/var/jenkins_home -v /etc/localtime:/etc/localtime --name myjenkins jenkins/Jenkins
4.查看jenkins是否启动成功
docker ps –l
浏览器输入ip加端口号进入

部署可参考https://www.cnblogs.com/xs233/p/15994703.htm
Node安装
curl -sL https://rpm.nodesource.com/setup_14.x | bash -
yum install -y nodejs
查看用node -v  npm –v
直接设置淘宝的源：npm config set registry http://registry.npm.taobao.org/ 
安装vue 
npm install -g @vue/cli
配置工作空间Manage nodes and clouds
在managejenkins/nodes下
加一个MainAgentNode
Number of executors=1
Remote root directory=/
Launch method=launch agent by connecting to the controller
保存
然后在部署的服务器上安装java11，低版本会出现不适配问题
安装java11
先查看系统有无自带java环境
java –version
查看java版本情况
yum list installed | grep java    或    rpm -qa | grep java
卸载自带jdk，以1.7为例（如有1.7和1.8则需要输入两次）
yum -y remove java-1.7.0-openjdk* 
再次查看，是否卸载成功
java –version
两种安装方式，第一种（不建议）
查看yum库中的Java安装包，输入：yum -y list java* 
输入安装（安装前看清库中是否有对应版本的jdk）：yum -y install java-11.0.8-openjdk* 。 
当结果显示为Complete！即安装完毕，无需后续操作，直接查看java -version是否成功。
第二种，手动安装
官网下载：https://www.oracle.com/java/technologies/javase-jdk11-downloads.html
下载完上传至指定文件夹（比如：/usr/local)
tar -zxvf jdk-11.0.8_linux-x64_bin.tar.gz  
设置环境变量
 export JAVA_HOME=/usr/local/jdk-11.0.8
  export CLASSPATH=.:$JAVA_HOME/jre/lib:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
  export PATH=$PATH:$JAVA_HOME/bin:$JAVA_HOME/jre/bin
按esc，再输入":wq"保存退出
环境生效：source /etc/profile  
查看是否安装成功
java –version

安装完Java后，找个目录在下面输入node提示的信息
例如
curl -sO http://172.16.102.15:8080/jnlpJars/agent.jar
java -jar agent.jar -jnlpUrl [http://172.16.102.15:8080/manage/computer/MainAgentNode/jenkins-agent.jnlp -secret 591900c6ef85b29fa8100a23e8c7adf11dc221d3d73b3722b05dc995a7bb7a1d -workDir "/](http://172.16.102.15:8080/manage/computer/MainAgentNode/jenkins-agent.jnlp%20-secret%20591900c6ef85b29fa8100a23e8c7adf11dc221d3d73b3722b05dc995a7bb7a1d%20-workDir%20%22/)"
没有报错则安装完成
Go安装：略
Gitlab配置Webhook
Jenkins中 http://172.16.1.195:8080/job/backend/configure设置Build Triggers->Authentication Token
Gitlab中 https://172.16.106.11/fengtianyang/namonitoring-clould/-/hooks 添加URL
在jenkins里面
1.在managejenkins/credentials里面加使用者权限信息，账号密码就行，id会自动生成
在项目里面Source Code Management，选git,然后加url和用户，Branches to build选择*/develop，若有报错则在jenkins容器里面执行git config --global http.sslverify false
Configure Global Security配置，Security Realm=Jenkins’own user database,authorization=anyone can do anything,githooks俩选项都选，apitoken三个选项全选，sshserver=disable, Git Host Key Verification Configuration=no verification
2.Gitlab操作：设置webhooks,
网址=http://172.16.102.15:8080/job/front/build?token=fronted
选推送事件和和并请求事件保存就行
3.项目里面Build Triggers，设置Authentication Token=fronted
若有403问题：在脚本中输入
hudson.security.csrf.GlobalCrumbIssuerConfiguration.DISABLE_CSRF_PROTECTION = true
若时间不同步问题：脚本执行
System.setProperty('org.apache.commons.jelly.tags.fmt.timeZone', 'Asia/Shanghai')
SSH配置:
在managejenkins/configuresystem里面配置ssh remote hosts
Hostname=172.16.102.13
Port=22
加一个root用户
然后在项目里面Execute shell script on remote host using ssh
就能连到服务器里面了
前端脚本备份：
#!/bin/bash
npm config set registry https://registry.npm.taobao.org
npm config get registry
npm cache clean --force
npm install -g cnpm --registry=https://registry.npm.taobao.org
npm install
npm run build:prod
pwd
docker rmi 172.16.106.11:10080/mingyang/cncp-dashboard:v1.0.0
docker build -f Dockerfile -t 172.16.106.11:10080/mingyang/cncp-dashboard:v1.0.0 .
docker push 172.16.106.11:10080/mingyang/cncp-dashboard:v1.0.0

cd /home/cncp
pwd
kubectl delete -f cncp-dashboard-deploy.yaml
kubectl create -f cncp-dashboard-deploy.yaml
后端脚本备份：
#!/bin/bash
go mod tidy
docker rmi 172.16.106.11:10080/mingyang/cncp-apiserver:1.0.0
docker rmi 172.16.106.11:10080/mingyang/cncp-controller-manager:1.0.0
docker build -f ./docker/cncp-apiserver/Dockerfile -t 172.16.106.11:10080/mingyang/cncp-apiserver:1.0.0 .
docker build -f ./docker/cncp-controller-manager/Dockerfile -t 172.16.106.11:10080/mingyang/cncp-controller-manager:1.0.0 .
docker push 172.16.106.11:10080/mingyang/cncp-apiserver:1.0.0 
docker push 172.16.106.11:10080/mingyang/cncp-controller-manager:1.0.0