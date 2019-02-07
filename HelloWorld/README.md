## Redis下载
官方网站有下载链接，或者从以下地址下载:<br>
https://github.com/antirez/redis/releases<br>
本教程使用的是Linux下的Redis，所以要下载tar.gz版本<br>
<br>
顺带一提，本教程使用的Redis版本是3.X<br>
<br>

## Redis编译
### GCC
redis是C语言开发，安装redis需要先将官网下载的源码进行编译，编译依赖gcc环境。<br>
输入命令 
```
yum install gcc-c++
```
若是显示没有安装过，则根据提示安装gcc<br>

解压redis-3.0.4.tar.gz
```
tar -zxvf redis-3.0.4
```

解压成功，进入 redis-3.0.4 文件夹
```
cd redis-3.0.4
```

输入make,编译redis<br>
如果没有安装gcc，编译将出现错误提示。<br>
若此时去安装好gcc后，再安装会出现以下错误提示
```
Jemalloc/jemalloc.h:没有那个文件或目录
```
此时必须先运行
```
make distclean
```
之后再运行make


## Redis安装
编译成功后，运行
```
make PREFIX=/usr/local/redis install
#安装redis 到 目录 /usr/local/redis
```
