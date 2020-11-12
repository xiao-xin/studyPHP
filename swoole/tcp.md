## swoole 实现一个tcp服务器

```php
// 创建一个$server对象，监听127.0.0.1
$server = new Swoole\Server('0.0.0.0','9500');
// 监听连接事件
$server->on('Connect',function($server,$fd){
	echo "Client connect\n";
});
// 监听数据接收事件
$server->on('Recver',function($server,$fd,$from_id,$data){
	$server->send($fd,"server:".$data);
});
// 监听连接关闭事件
$server->on('Close',function($server,$fd){
	echo "Client close\n";
});
// 启动一个tcp服务器
$server->start();
```