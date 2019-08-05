1、nginx.conf文件结构：
nginx.conf一共有三部分组成，分别为全局块、events块和http块。在http块中，又包括http全局块、多个server块。每个server块中，可以包含server全局块和多个location块。如果某个指令在两个不同层级的块中同时出现，则采用“就近原则”，即以较低层级块中的配置为准。比如，某个指令同时出现在http全局块中和server块中，并且配置不同，则应该以server块中配置为准。
日志级别：debug | info | notice | warn | error | crit | alert | emerg
2、网络设置：
“惊群”：当某一时刻只有一个网络连接到来时，多个睡眠进程会被同时叫醒，但只有一个进程可获得连接。如果每次唤醒的进程数目太多，会影响一部分系统性能。
nginx服务器在多进程下，就有可能出现这样的问题。通常多数人不会注意Nginx的accept_mutex配置，不过实际上它对系统的吞吐量有一定的影响。
events { 
accept_mutex off; 
} 
让我们看看accept_mutex的意义：当一个新连接到达时，如果激活了accept_mutex，那么多个Worker将以串行方式来处理，其中有一个Worker会被唤醒，其他的Worker继续保持休眠状态；如果没有激活accept_mutex，那么所有的Worker都会被唤醒，不过只有一个Worker能获取新连接，其它的Worker会重新进入休眠状态，这就是「惊群问题」。
3、配置location块：
通过指定模式来与客户端请求的URI相匹配，基本语法如下：location [=|~|~*|^~|@] pattern{……}
1、没有修饰符 表示：必须以指定模式开始，如：
server {
　　server_name baidu.com;
　　location /abc {
　　　　……
　　}
}

那么，如下是对的：
http://baidu.com/abc
http://baidu.com/abc?p1
http://baidu.com/abc/
http://baidu.com/abcde
 
2、=表示：必须与指定的模式精确匹配，匹配成功，立即处理
server {
server_name sish
　　location = /abc {
　　　　……
　　}
}
那么，如下是对的：
http://baidu.com/abc
http://baidu.com/abc?p1
如下是错的：
http://baidu.com/abc/
http://baidu.com/abcde
 
3、~ 表示：指定的正则表达式要区分大小写
server {
server_name baidu.com;
　　location ~ ^/abc$ {
　　　　……
　　}
}
那么，如下是对的：
http://baidu.com/abc
http://baidu.com/abc?p1=11&p2=22
如下是错的：
http://baidu.com/ABC
http://baidu.com/abc/
http://baidu.com/abcde
 
4、~* 表示：指定的正则表达式不区分大小写
server {
server_name baidu.com;
location ~* ^/abc$ {
　　　　……
　　}
}
那么，如下是对的：
http://baidu.com/abc
http://baidu..com/ABC
http://baidu..com/abc?p1=11&p2=22
如下是错的：
http://baidu..com/abc/
http://baidu..com/abcde
 
5、^~ 类似于无修饰符的行为，也是以指定模式开始，不同的是，如果模式匹配，
那么就停止搜索其他模式了。
6、@ ：定义命名location区段，这些区段客户段不能访问，只可以由内部产生的请
求来访问，如try_files或error_page等

root 、alias指令区别
location /img/ {
    alias /var/www/image/;
}
#若按照上述配置的话，则访问/img/目录里面的文件时，ningx会自动去/var/www/image/目录找文件
location /img/ {
    root /var/www/image;
}
#若按照这种配置的话，则访问/img/目录下的文件时，nginx会去/var/www/image/img/目录下找文件。] 
alias是一个目录别名的定义，root则是最上层目录的定义。
还有一个重要的区别是alias后面必须要用“/”结束，否则会找不到文件的。。。而root则可有可无~~