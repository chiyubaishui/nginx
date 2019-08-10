Nginx使用upstream_check_module模块实现后端节点健康检查功能

nginx自带的针对后端节点健康检查的功能比较简单，通过默认自带的ngx_http_proxy_module模块和ngx_http_upstream_module模块中的相关指令来完成当后端节点出现故障时，自动切换到健康节点来提供访问。

举例:
upstream name {
server 192.168.57.110:8080 max_fails=1 fail_timeout=10s;
server 192.168.57.101:8080 max_fails=1 fail_timeout=10s;
}

参数说明：

max_fails=number # 设定Nginx与服务器通信的尝试失败的次数。在fail_timeout参数定义的时间段内，如果失败的次数达到此值，Nginx就认为服务器不可用。在下一个fail_timeout时间段，服务器不会再被尝试。 失败的尝试次数默认是1。设为0就会停止统计尝试次数，认为服务器是一直可用的。 你可以通过指令proxy_next_upstream、fastcgi_next_upstream和 memcached_next_upstream来配置什么是失败的尝试。 默认配置时，http_404状态不被认为是失败的尝试。

fail_timeout=time # 设定服务器被认为不可用的时间段以及统计失败尝试次数的时间段。在这段时间中，服务器失败次数达到指定的尝试次数，服务器就被认为不可用。默认情况下，该超时时间是10秒。

这种情况Nginx无法主动识别后端节点状态，后端即使有不健康节点， 负载均衡器依然会先把该请求转发给该不健康节点，然后再转发给别的节点，这样就会浪费一次转发，而且自带模块无法做到预警。So 此时使用第三方模块 nginx_upstream_check_module模块

nginx_upstream_check_module模块由淘宝团队开发 淘宝自己的 tengine 上是自带了该模块的，大家可以访问淘宝tengine的官网来获取该版本的nginx，官方地址：http://tengine.taobao.org/。我们使用的是原生Nginx，采用添加模块的方式

部署流程

1、下载nginx_upstream_check_module模块

#进入nginx安装目录
cd /data/xjk/software/
#下载nginx_upstream_check_module模块
wget https://codeload.github.com/yaoweibin/nginx_upstream_check_module/zip/master
#解压
unzip master


下载和解压
cd nginx-1.13.3 # 进入nginx的源码目录

# -p0，是“当前路径” -p1，是“上一级路径”

patch -p0 < ../nginx_upstream_check_module-master/check_1.11.5+.patch

#nginx -V 可以查看原有配置 输出 ./configure --prefix=/usr/local/nginx

#增加upstream_check模块

./configure --prefix=/usr/local/nginx --add-module=../nginx_upstream_check_module-master

make

替换sbin/nginx二进制文件
cd /usr/lcoal/nginx/sbin/
mv nginx nginx-1.13.3.bak
cp /data/xjk/software/nginx-1.13.3/objs/nginx .

/usr/local/nginx/sbin/nginx -t # 检查下是否有问题

注意 check版本和Nginx版本要求有限制 1.13以上版本的nginx，补丁为check_1.11.5+.patch 具体参考github https://github.com/yaoweibin/nginx_upstream_check_module


给nginx打补丁
3.修改配置文件，让nginx_upstream_check_module模块生效

upstream name {
server 192.168.57.207:8090;
server 192.168.57.85:80;
check interval=3000 rise=2 fall=5 timeout=1000 type=http;
}

参数说明：
interval：必要参数，检查请求的间隔时间。
fall：当检查失败次数超过了fall，这个服务节点就变成down状态。
rise：当检查成功的次数超过了rise，这个服务节点又会变成up状态。
timeout：请求超时时间，超过等待时间后，这次检查就算失败。
default_down：后端服务器的初始状态。默认情况下，检查功能在Nginx启动的时候将会把所有后端节点的状态置为down，检查成功后，在置为up。
type：这是检查通信的协议类型，默认为http。以上类型是检查功能所支持的所有协议类型。
4.启动或者重载nginx
