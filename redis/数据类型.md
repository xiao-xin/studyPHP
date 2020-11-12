## List
基本的数据类型，列表，可以组成队列、栈
list数据结构的命令都是l开头的。

### List常用的命令

插入
```bash
#设置list key，从左边插入
lpush list1 "hello" "word"
#从左边取出
rpush list1 "php"
#在指定元素值处插入值
linsert list1 after two nine
```

删除
```bash
#移除指定的值，移除列表中的的php，最多移除2个
lrem list1 2 "php"
#截取指定范围的值 ltrim key start end
ltrim list1 1 2 
```

查询
```bash

#获取值[0,-1]
lrange list1 0 -1
#从右边插入
lpop list1
#从右边移除
rpop list1
#根据下标获取值
lindex list1 2
#获取列表的长度
llen list1
```

其他
```bash 
#移动列表中最有一个元素到新列表
rpoplpush list1 list2

```
我们看到list中的操作是压入弹出，没有像string那样的set、get操作，一开始对list不熟悉，只要你掌握list的数据结构底层的实现，你就知道为什么有这些命令了。

list底层是一个链表，左边右边都可以插入值到链表中。
如果key存在，就在这个链表中新增元素，反之就新增一个链表。

链表的特点是两边插入弹出值效果高，如果中间插入值效率是比较低的。

### List应用
1. 消息队列 （lpush,rpop）
2. 栈(lpush,lpop)

## Set数据结构

set数据结构，是无须的，元素不可以重复。

具体的知识点可以看下文章之类的。

### Set常用的命令

新增
```bash
sadd set1 "wang" "chen"
```

查询
```bash
#返回集合中的所有元素
smembers set1
#查看集合中是否有指定的元素
sismember set1 "wang"
#查看集合中的元素个数
scard set1 
#随机中集合中取出几个元素
srangemember set1 2
```

删除
```bash
#随机删除几个元素
spop set1 2 
#删除指定元素
srem set1 "chen" "tao"
#移除指定元素到另一个集合中
smove set1 set2 "one"
```

交集、并集
```bash
#set1集合和set2集合的差集
sdiff set1 set2
#求集合的交集
sinter set1 set2
#集合的并集
sunion set1 set2
```
交集和并集可以求共同的好友，具体的用法后面再说。

## Hash
hash也是集合，只不过这种结合可以存储key-value 。

### Hash常用的命令

新增
```bash
hset user:1 name "jack" 
#批量设置
hset user:1  name "jack" age  "25"
hsetnx user:1 age 27
```

修改
```bash
hincrby user:1 age 1
```

查找
```bash
#获取所有的field和value
hgetall user:1
#获取所有的key
hkeys user:1
#获取所有的value
hvals user:1
hexists user:1 age
#获取所有字段的数量
hlen user:1
```

删除
```bash
#删除指定的field
hdel user:1 age 
```

