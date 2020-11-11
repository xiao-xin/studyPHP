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