公司实战nginx之rewrite配置域名跳转

1.最近遇到了一个开发提的需求，访问blog.cool360.org的时候跳转到blog.cool360.org/blog的需求。
看着挺简单的，但是还是研究了下才配置出来，主要还是不够深入了解nginx的rewrite规则。
正确的配置如下：
rewrite ^/$ http://blog.cool360.org/blog permanent;
2.朋友询问我一个关于nginx跳转的问题
如下：
访问 http://mapi2.dev.hoge.cn/mxu/xxxx.php 变成 http://mapi2.dev.hoge.cn/api/xxxx
配置如下：
location ~ /mxu/.*\.php {
rewrite "^/mxu/(.*)\.php" /api/$1 permanent;
}

3.公司有一个域名跳转问题

192.168.0.1:8080/test/1234 想通过域名www.abc.com直接访问。

http跳转https：
    rewrite     ^(.*)$ https://$server_name$1 permanent;

https中配置:

  proxy_pass  192.168.0.1:8080;
    if ($document_uri !~ 'admin')
    {
    rewrite ^/(.*)$ https://192.168.0.1:8080/test/1234$1 permanent;
    }
   

备注：
rewrite指令的最后一项参数为flag标记，flag标记有：
1.last 相当于apache里面的[L]标记，表示rewrite。
2.break本条规则匹配完成后，终止匹配，不再匹配后面的规则。
3.redirect 返回302临时重定向，浏览器地址会显示跳转后的URL地址。
4.permanent 返回301永久重定向，浏览器地址会显示跳转后的URL地址