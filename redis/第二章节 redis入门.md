 
## 安装redis用到的命令
```bash
# *********升级gcc***********
# 安装scl源
yum install -y centos-release-scl scl-utils=build
# 安装 gcc 9版本的gcc, gcc-c++  gdb工具链
yum install -y devtoolset-9-tollchain
# 临时覆盖系统现在的gcc 引用
scl enable devtoolset-9 bash
# 查看gcc 的当前版本
gcc -v

  167  gcc --version
  168  gcc -v
  169  cd /usr/local/src
  170  ls
  171  wget http://download.redis.io/releases/redis-6.0.6.tar.gz
  172  tar -zxvf redis-6.0.6.tar.gz 
  173  ls
  174  cd redis-6.0.6
  175  ls
  176  make
  177  make PREFIX=/usr/local/redis install
  178  ls /usr/local/redis/
  179  cp redis.conf  /usr/local/redis/
  180  cd /usr/local/
  181  cd redis/
  182  ls
  183  cp redis.conf redis.conf.bak
  184  vim redis.conf
  185  bin/redis
  186  bin/redis-server 
  193  vim redis.conf
  194  ls
  195  bin/redis-server  ./redis.conf
  196  ps -ef |grep redis
  197  redis-cli
  198  bin/redis-cli
  199  history
```

安装redis.service.
```bash
vim /etc/systemd/system/redis.server
````
```bash

[Unit]
Description=redis-server
After=network.target
[Service]
Type=forking
ExecStart=/usr/local/bin/redis-server /usr/local/redis.conf
PrivateTmp=true
[Install]
WantedBy=multi-user.target
```

启动
```bash
ps -ef | grep redis
222  vim /etc/systemd/system/redis.server
223  vim redis.service 
224  systemctl  daemon-reload
225  systemctl status redis.service
226  systemctl start redis.service
227  systemctl status redis.service
228  ps -ef | grep redis
systemctl enable redis.service
```


设置密码
```bash
vim redis.conf
# 把#去掉并设置密码
requirepass 123456

# 重启
systemctl restart redis

# 进入redis-cli后
# 使用命令登录后才可以执行命令
auth 123456
```

常用配置

Redis的配置文件
Redis支持很多的参数，但都有默认值。
* daemonize默认情况下，redis不是在后台运行的，如果需要在后台运行，把该项的值更改为yes。
* bind指定Redis只是收来自于该IP地址的请求。
* port监听端口，默认为6379。
* databases设置数据库的个数，默认使用的数据库是0。
* save设置Redis进行数据库镜像的频率。
* dbfilename镜像备份文件的文件名。
* dir数据库镜像备份的文件放置的路径。
* requirepass设置客户端连接后进行任何其他指定前需要使用的密码。
* maxclients限制同时连接的客户数量。
* maxmemory设置redis能够使用的最大内存。