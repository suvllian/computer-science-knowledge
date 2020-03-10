## 常用shell命令

### 如何查看端口是否被某个进程占用？

``` shell
$ lsof -i:3000
```

### 杀掉制定PID的进程

``` shell
$ sudo kill -9 750
```

### 使用shell SSH连接远程服务器

* 打开终端，进入根目录
* ssh -p 端口号 用户名@ip，输入后回车
* 输入密码

``` shell
$ sudo su 
$ ssh -p 22 username@119.22.33.44
```