# Linux系统下Apache服务器的配置

## 一、设置404页面
* 修改httpd文件，设置AllowOverride All
* 在服务器配置的网站根目录下添加.htacess文件，添加内容ErrorDocument 404 /404.html(404.html为编辑好的404页面)
* 重启服务器即可

## 二、设置403Forbidden禁止访问目录，关闭目录浏览权限
有些情况下，如果没有对某些目录设置403权限，用户可直接根据路径查看到当前文件夹下所有文件，这对于网站安全来说是不利的，所以需要设置403，让用户不能直接读取文件夹下所有文件。

* 在文件.htaccess中添加：ErrorDocument 403 /403.html
* 修改httpd文件

将httpd文件中的：
``` 
<Directory />
    Options Indexes FollowSymLinks #主要是这句中的Indexs
    AllowOverride None
    Order deny,allow
    Deny from all
</Directory>
```
修改为：
```
<Directory />
    Options FollowSymLinks
    AllowOverride All
    Order deny,allow
    Allow from all
</Directory>
```
下面还有一处也是Options Indexes FollowSymLinks
改为 Options FollowSymLinks

* 重启服务器

## 三、配置多站点   
参考文章：http://baijunyao.com/article/9  
操作流程：开启服务器**虚拟主机**配置；修改服务器hosts文件，将域名解析到服务器；修改虚拟主机配置。

### 第一步：修改httpd.conf
* 命令行输入：`vim /etc/httpd/conf/httpd.conf`进入apache配置文件
* 删除`NameVirtualHost *:80`前的注释

### 第二步：修改hosts文件
* 输入ifconfig查看ip地址，inet addr
* 输入vim /etc/hosts；添加ip地址域名。
```
115.28.17.191 suvllian.com
115.28.17.191 suvllian.top
```

### 第三步：修改httpd-vhosts.conf文件
* 输入`vim /etc/httpd/conf.d/virtual.conf`，若没有则添加。
* 编辑文件
```
<VirtualHost *:80>
  DocumentRoot /var/www/html/V1
  ServerName suvllian.top
</VirtualHost>
<VirtualHost *:80>
  DocumentRoot /var/www/html/
  ServerName suvllian.com
</VirtualHost>
```

### 第四步：重启服务器即可

## 四、开启gzip压缩
### 第一步：修改服务器配置
httpd.conf中打开deflate_Module和headers_Module模块，具体做法为将 如下两句前面的#去掉：
```
LoadModule deflate_module modules/mod_deflate.so
LoadModule headers_module modules/mod_headers.so
```

### 第二步：在httpd.conf文件底部加入如下代码配置需要压缩的文件
```
<IfModule deflate_module>
SetOutputFilter DEFLATE
# Don’t compress images and other
SetEnvIfNoCase Request_URI .(?:gif|jpe?g|png)$ no-gzip dont-vary
SetEnvIfNoCase Request_URI .(?:exe|t?gz|zip|bz2|sit|rar)$ no-gzip dont-vary
SetEnvIfNoCase Request_URI .(?:pdf|doc)$ no-gzip dont-vary
AddOutputFilterByType DEFLATE text/html text/plain text/xml text/css
AddOutputFilterByType DEFLATE application/x-javascript
</IfModule>
```

## 五、开启缓存
### 第一步：在配置文件中开启：
```
LoadModule headers_module modules/mod_headers.so
```
### 第二步：在.htaccess中添加：
包含这些后缀的资源都缓存33秒
```
<FilesMatch ".(flv|gif|jpg|jpeg|png|ico|swf)$">
Header set Cache-Control "max-age=33"
</FilesMatch>
```