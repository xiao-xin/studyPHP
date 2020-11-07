## 前言
这里简单的记录一下TP6的使用方法，作为以后工作使用。

## TP6新特性
### 启用了TP自带的类文件加载器
TP6使用了Composer提供的类文件加载器，自己实现的那套加载器已经没有使用了。

### 支持严格模式
PHP7新特性严格模式，具体怎么使用不知道，后面慢慢了解。

### 组件独立
把ORM,验证码等等都独立成组件，使用Composer安装使用。

## 统一的返回格式
接口一般返回的内容都是固定的格式，我们可以建立一个方法专门来返回固定的格式。

```php
function status($status,$msg,$data,$httpStatus=200)
{
    $result=[
        'status' => $status,
        'message' => $msg,
        'data' => $data,
    ]
    return json($result,$httpStatus)
}
```
在调用的是否我们会输入返回的状态，这里容易输错，为了防止我们可以添加一个状态配置文件，统一返回的状态。
```php
// status.php
return [
    'success' =>0,
    'error' =>1000,
    'not_login' =>10001
]
```

这样我们就可以这样返回信息给前端`show(config('status.success'),'ok')`,这样我们不用每次输入数值，因为这样的输入第一个可读性差，不便于维护，好处就是大家都可以用，约束数值的意义。


## 数据库和ORM
TP提供了两种可以访问数据库的方式，第一种就是使用Db类提供的方法直接操作，第二种就是ORM数据库关系映射模型来使用，第二种用到的比较多，功能也比较强大。

这里展示一下使用Db类来操作数据库
```php
// Db门面模式
Db::table('user')->where('user_id',10)->find()
// 容器
app('Db')->table()->where....
```
具体的数据库操作可以看文档，这里就不举例了，和TP5没有什么区别的。

## 多应用模块
TP6默认是不支持多应用模块的，这样的话我们只能使用composer去安装多应用扩展包来支持多应用。
```
composer reuiqre topthink/think-multi-app
```
这样我们可以在app目录下创建模块，比如创建admin模块,文件结构如下:
-app
--admin
---controller
---Index.php

这样还不能访问需要设置nginx配置。
```conf
 if (!-e $request_filename) {
        # 注意这行要放在下一行的上面，不然匹配不到这一行
        rewrite ^/index.php(.*)$ /index.php?s=$1 last;
        rewrite ^(.*)$ /index.php?s=$1 last;
}

```
接着在浏览器中输入tp.test/admin就可以访问到admin模块了。

### 多模块下的路由设置
多个模块下可以设置各自的路由方式，比如在admin模块目录下新建一个route目录，然后编写php脚本文件`route.php`。
```php
# 注意这里的前两个参数不要添加模块名，添加也是白家，没有意义
Route::rule('nihao','index/hello','get');
```

## 五层架构分层
架构的目的就是解耦，统一规范，提高开发速度。

之前我们的项目都是MVC三层，但是很多的操作都是放在控制器，导致控制器的代码非常的多，代码复用非常少。应该分层，把控制器中的代码转义到其他层。

这里我们提供5层架构，首先是控制器层Controller,主要是接收客户的请求，然后控制器不是直接调用Model层，而是调用业务逻辑层Business，比如数组的组装就在这层操作，设计到一些基础库公共的部分，我们调用基础库Lib层。业务操作肯定会调用数据库，这把这些代码放在模型层，最后层层返回到控制器，最后由控制器调用View中的模板渲染返回给客户端。

```
#架构流程如下图所示：
Request -> Controller -> Business ->Model,Lib ->... ->View -> Response
```
model,business,lib这些目录我们放在common公共模块下，方便其他模块也可以调用。目录结构如下：
www
├─app    
│  ├─common         
│  │  ├─common.php     
│  │  ├─business
│  │  ├─lib
│  │  ├─model     
│  │  │  ├─mysql     
│  │  │  ├─es     
│  │  │  ├─redis     


## 异常处理