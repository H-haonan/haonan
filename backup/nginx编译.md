方法一：直接编译
 tar -zxvf nginx-1.20.2.tar.gz
ll
cd nginx-1.20.2
ll
yum install -y gcc gcc-c++ glibc make autoconf openssl openssl-devel && yum install -y pcre-devel libxslt-devel gd-devel GeoIP GeoIP-devel GeoIP-data
cd ..
 yum install -y unzip zip && unzip ngx_http_substitutions_filter_module-master.zip
cd nginx-1.20.2


./configure --prefix=/usr/local/nginx --with-file-aio --with-http_ssl_module --with-http_realip_module --with-http_addition_module --with-http_xslt_module --with-http_image_filter_module --with-http_geoip_module  --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_auth_request_module --with-http_random_index_module --with-http_secure_link_module --with-http_degradation_module --with-http_stub_status_module --add-module=/home/install/ngx_http_substitutions_filter_module-master && make && make install

cd /usr/local/nginx/sbin/
./nginx
./nginx -s stop
./nginx
cd ..
cd sbin
./nginx -s reload

ps -ef | grep nginx
方法二：通过nginx二进制包安装
需要加入/usr/local/nginx/conf/nginx.conf文件和/usr/local/nginx/logs/error.log文件
执行chmod a+x nginx最后执行./nginx
