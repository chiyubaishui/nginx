nginx的启停办法中，有一类是通过信号机制来实现的。我们通过给nginx服务的主进程发送信号就可以控制服务的启停了。
TERM或INT	快速停止nginx服务
QUIT	平缓停止nginx服务
HUP	使用新的配置启动进程，然后平缓停止原有的进程，既平滑重启。
USR1	重新打开日志文件，用于日志切割
USR2	使用新版本的nginx文件启动服务，然后平缓停止原有nginx进程，既平滑升级
WINCH	平缓停止worker process,用于nginx服务器平滑升级



具体语法:

Kill -信号选项 nginx的主进程号

例：

Kill -INT 26661  #杀死nignx进程

Kill -HUP 4873  #平滑的读取新配置文件，不必重用nginx



nginx平滑升级：
1、进入到新的nginx目录，编译安装，不要make install
2、替换sbin/nginx二进制文件
mv nginx nginx1_12_1
cp ~/nginx-1.12.2/objs/nginx .

3、检查是否成功，并平滑关闭
./nginx -t

nginx: the configuration file /application/nginx1.12.1/conf/nginx.conf syntax is ok
nginx: configuration file /application/nginx1.12.1/conf/nginx.conf test is successful


kill -USR2 `cat /application/nginx/logs/nginx.pid`
kill -QUIT `cat /application/nginx/logs/nginx.pid.oldbin`

平滑升级的过程：nginx服务接收到USER2信号后，首先将旧的nginx.pid文件添加后缀.oldbin文件，然后执行新版本nginx服务器的二进制文件启动服务。如果新的服务启动成功，系统中将有新旧两个nginx服务共同提供web服务。之后，需要向旧的nginx服务进程发送WINCH信号，使旧的nginx服务平滑停止，并删除nginx.pid.oldbin文件。在发送WINCH信号之前，可以随时停止新的nginx服务。


进行版本nginx-1.15.8二进制文件的备份(当服务升级失败后保证旧的服务依然可以继续工作)

cd   /usr/local/nginx/sbin      ###进入到二进制所在文件夹
co nginx  nginx.old             ###进行二进制文件的备份


 

查看未升级前的nginx的版本

/usr/local/nginx/sbin/nginx     -V    ###查看工作状态下nginx的版本


 

将nginx-1.15.8的二进制文件使用nginx-1.16.0进行强制覆盖

cp -f nginx   /usr/local/nginx/sbin/nginx    ###进行二进制文件的强制覆盖
ps ef | grep nginx     ###进行进程的查看


 

设定旧的服务不再接收用户请求(下线)，新服务启动子进程接收用户请求(上线)

kill -USR2 3760     ###设定新的子进程开始接收用户的访问请求,旧的不再九首用户的访问请求
ps -ef | grep nginx   ###进行进程的查看


 

进行旧服务的进程关闭

kill -WINCH  3760    ###进行旧服务进程的关闭
ps -ef | grep nginx    ###进行进程的查看


 

进行服务版本的查看

/usr/local/nginx/sbin/nginx   -V    ###进行nginx版本号的查看


 

当服务升级失败后进行版本的回调


进行二进制文件的强行覆盖

cp -f nginx.old nginx     ###进行二进制文件的强行覆盖
ps -ef | grep  nginx      ###进行nginx服务进程的查看


 

进行配置文件的动态更新

kill -HUP  3760   ###进行配置文件的动态更新
ps -ef | grep nginx   ###进行服务进程的查看


 

让新版本的服务停止接收用户请求

kill -USR2 15900    ###设定新升级的版本不再接收用户请求
ps -ef | grep nginx   ###进行服务进程的查看


 

进行原始版本子进程的拉起

kill -WINCH 15900    ###进行新版服务的拉起
ps -ef | grep nginx   ###进行服务进程的查看


 

进行回退后版本的查看

/usr/local/nginx/sbin/nginx   -V    ###进行服务版本的查看


 

进行新版本进程的查看，并且进行进程的关闭

ps ax | grep nginx    ###进行服务进程的查看
kill -9 15900         ###进行新版服务进程的结束




 

 

使用浏览器进行访问


--------------------- 
作者：强力胶 
来源：CSDN 
原文：https://blog.csdn.net/weixin_43831670/article/details/90228658 
版权声明：本文为博主原创文章，转载请附上博文链接！