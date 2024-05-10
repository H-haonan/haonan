环境：
Centos7环境
内核版本号：4.19.94-300.el7.x86_64
1.安装系统包和部分软件包
yum -y install patch make gcc gcc-c++ gcc-g77 flex* bison file  
yum -y install libtool libtool-libs libtool-ltdl-devel* autoconf kernel-devel automake libmcrypt*  
yum -y install libjpeg libjpeg-devel libpng libpng-devel libpng10 libpng10-devel gd gd-devel  
yum -y install freetype freetype-devel libxml2 libxml2-devel zlib zlib-devel  
yum -y install glib2 glib2-devel bzip2 bzip2-devel libevent libevent-devel  
yum -y install ncurses ncurses-devel curl curl-devel e2fsprogs  
yum -y install e2fsprogs-devel krb5 krb5-devel libidn libidn-devel  
yum -y install openssl openssl-devel vim-minimal nano sendmail  
yum -y install fonts-chinese gettext gettext-devel  
yum -y install gmp-devel pspell-devel   
yum -y install readline* libxslt* pcre* net-snmp* gmp* libtidy*  
yum -y install ImageMagick* subversion*

2.安装mysql
rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
yum install mysql-server
启动mysql服务并查看运行状态
service mysqld start
systemctl status mysqld
查看初始密码：grep 'A temporary password' /var/log/mysqld.log
登录：mysql -uroot –p
密码就是查看初始密码
然后先需要改密码，要不会有提醒
先设置长度和等级
mysql> set global validate_password_policy=LOW;
mysql> set global validate_password_length=6;
mysql> ALTER USER USER() IDENTIFIED BY '123456';#123456为密码
创建数据库
create database redmine character set utf8;  
create user 'redmine'@'localhost' IDENTIFIED BY '123456'; #用户名: redmine; 密码:123456  
grant all privileges on redmine.* to 'redmine'@'localhost'; #最大权限  
flush privileges;
exit退出

3.安装rvm
1.创建目录
cd /usr/local
mkdir rvm
cd rvm
command curl -sSL https://rvm.io/mpapis.asc | gpg2 --import -
command curl -sSL https://rvm.io/pkuczynski.asc | gpg2 --import -
curl -L get.rvm.io | bash -s stable
或者
1.执行curl -L get.rvm.io | bash -s stable， 一般会报错，并提示
gpg --keyserver hkp://keys.gnupg.net
–recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
7D2BAF1CF37B13E2069D6956105BD0E739499BDB

2.这时便获取密钥，然后执行gpg --keyserver hkp://keys.gnupg.net
–recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
7D2BAF1CF37B13E2069D6956105BD0E739499BDB
3.执行curl -sSL https://get.rvm.io | bash -s stable
4. 查看rvm版本 rvm –v
不好用执行source /etc/profile.d/rvm.sh

4.安装ruby
查看是否有ruby2.6版本
rvm list known
rvm install 2.6

5.安装rails和相关依赖包
安装rails5.2
gem install rails -v 5.2  
安装rake
gem install rake  
安装mysql2
gem install mysql2  
安装activesupport
gem install activesupport -v 4.2.6
安装cocoapods
gem install cocoapods

6.下载redmine并安装依赖
cd /home
https://www.redmine.org/projects/redmine/wiki/Download #下载地址
将下载的安装包放入home路径下解压
cd redmine-4.2.0
bundle install

7.配置redmine并初始化
配置文件
cp config/database.yml.example config/database.yml  
选择production部分修改数据库密码
vim config/database.yml 
安装mysql2
gem install mysql2
生成Rails使用的随机密钥，用于编码存储会话数据的cookie，从而防止其被篡改。 
rake generate_secret_token  
生成表结构
RAILS_ENV=production rake db:migrate 
初始化数据选择zh
RAILS_ENV=production rake redmine:load_default_data

8.启动redmine
手动运行，如果不指定地址默认为localhost
bundle exec rails server webrick -e production -b 172.16.106.24（ip看服务器地址）

9.访问登录
访问 
http://172.16.106.24:3000/
默认账号和密码 admin admin
10.插件安装补充说明
首先把插件安装包放到redmine下的plugn里面，解压安装包
yum install unzip
unzip redmine_agile-1_6_5-light.zip
ls
cd ..
bundle install --without development test --no-deployment
bundle exec rake redmine:plugins NAME=redmine_agile RAILS_ENV=production
touch tmp/restart.txt
11.问题
问题1：sql2数据库安装不上问题，提示 /usr/bin/ld: 找不到 -lmysqlclient
有可能是链接情况不对
1. 查找与mysqlclient相关的文件
find /usr -name '*mysqlclient*'
2.查看一下链接的情况
ll /usr/lib64/libmysqlclient.so
3.  删除旧的链接并创建新的链接
rm -rf /usr/lib64/libmysqlclient.so
ln -s /usr/lib64/mysql/libmysqlclient.so.18 /usr/lib64/libmysqlclient.so
 再次查看链接的情况
ll /usr/lib64/libmysqlclient.so
问题2：mysql.h：没有那个文件或目录
缺少libmysqlclient-dev
ubuntu下  ：  audo apt-get install libmysqlclient-dev
centos下 : yum install mysql-devel

问题3：待补充