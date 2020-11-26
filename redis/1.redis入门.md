## 安装
redis的安装教程有很多，这里就不累赘了，贴一个比较好的安装教程，访问这个地址即可。
https://learnku.com/articles/15128/compile-and-install-redis-under-the-linux-environment

如果有命令不明白的地方，可以查询。
http://redis.cn/commands.html

## 选择数据库
```bash
#客户端登录
redis-cli -h localhosts -p 6379 
#切换数据库,index范围0-x
select index  
#查看数据库大小
dbsize
#清空数据库
flushdb
#清空全部数据库
```

## redis单线程
redis非常的块，基于内存操作，CPU不是reids的性能瓶颈，内存和网络带宽才是，所以单线程就足够了。

为什么单线程还这么快？
1. C语言写的，接近100000+的QPS
2. 多线程不一定比单线程高，对于内存系统来说，没有上下文切换消耗是最高的，多次读写都是在一个CPU上。


## 数据类型
常用的redis-key
```bash
#设置key
set name jack
#获取key
get name
#验证key
exists name
#移动key到其他库
move name 1
#过期
expire name 10
#获取过期时间, -1=永久,-2=过期,大于0表示剩余的过期秒数
ttl name 
#key的类型
type name
```
redis的命令非常的多，如果遇到不会的命令，可以上官网查找。http://redis.cn/commands.html


## 字符串类型

```bash
set name jack
#字符串追加内容，返回字符串的长度
append name " chen"
#字符串长度
strlen name
```

自增相关的操作
```bash
#自增
set article_view 0
#key的增加1，比如保证article_view是一个整数，否则会报错
incr articel_view
#减1
decr articel_view
#增加步长
incrby article_view 10
#减少步长
decrby article_view 10
```
字符串操作相关的命令
```bash
#获取字符串指定起止位置[0,5]的字符串
getrange name 0 5
#获取整个字符串
getrange name 0 -1
#替换字符串指定位置的字符
setrange name 3 xx
```

设置key
```
#设置key 30秒过期,如果存在会覆盖
setex name 30  "jack"
#key不存在才会设置返回1，存在返回0
setnx name "chen"
```

批量操作
```bash
#批量设置多个key
mset name "jack" age 25
#批量获取多个key
mget name age
#当多个key都不存在时才创建,原子操作（要么都成功）
msetnx height "175" address "China"
```
设置对象
在redis中是不支持对象操作的，不过我们可以变相的来实现
```bash
#模拟对象
mset user:1:name "zhansan"  user:1:age "25"
```

先获取key后设置key
```bash
#如果key不存在会返回nil，然后设置值为jack
getset name "jack"
#返回jack,然后设置值为chen
getset name "chen" 
```

## 字符串的应用场景
1. 计数器
2. 统计（粉丝数）