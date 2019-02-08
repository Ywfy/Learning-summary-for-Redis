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
之后再运行make<br><br>


## Redis安装
编译成功后，安装redis到目录 /usr/local/redis,运行
```
make PREFIX=/usr/local/redis install
```
![无法加载图片](https://github.com/Ywfy/Learning-summary-for-Redis/blob/master/HelloWorld/%E6%8D%95%E8%8E%B7.PNG)<br>
出现以上提示就是安装成功了<br>

进入安装目录，查看文件<br>
![无法加载图片](https://github.com/Ywfy/Learning-summary-for-Redis/blob/master/HelloWorld/az.png)<br>
<br>
其中：

redis-benchmark      ----性能测试工具

redis-check-aof        ----AOF文件修复工具

redis-check-dump    ----RDB文件检查工具（快照持久化文件）

redis-cli                  ----命令行客户端

redis-server          ----redis服务器启动命令
<br><br>

## 配置redis.conf
在启动redis之前，需要先配置redis.conf文件<br>
进入一开始解压的安装包文件夹，浏览文件，会发现有个redis.conf<br>
![无法加载图片](https://github.com/Ywfy/Learning-summary-for-Redis/blob/master/HelloWorld/conf.png)<br>
我们在根目录下新建一个myredis文件夹，将redis.conf复制到里面<br>
```
mkdir /myredis
cp redis.conf /myredis
```
<br>

进入myredis目录，修改redis.conf
```
cd /myredis
vim redis.conf
```
将 daemonize no 修改为 daemonize yes  ,这样redis就可以后台运行
![无法加载图片](https://github.com/Ywfy/Learning-summary-for-Redis/blob/master/HelloWorld/conf2.png)<br>
<br>

## 启动redis
进入安装目录
```
cd /usr/local/redis/bin
```
启动redis服务
```
./redis-server /myredis/redis.conf
```

启动 redis客户端
```
./redis-cli 
```
![无法加载图片](https://github.com/Ywfy/Learning-summary-for-Redis/blob/master/HelloWorld/qd.png)<br>
上图表示redis服务和客户端均已启动成功<br>

此时可以进行如下测试:<br>
![无法加载图片](https://github.com/Ywfy/Learning-summary-for-Redis/blob/master/HelloWorld/cs.png)<br>
<br>


## 关闭redis
```
./bin/redis-cli shutdown
```
如果是在客户端下操作则
```
shutdown
exit
```
