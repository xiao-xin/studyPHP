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
```
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
```
## 异常处理
传统的异常处理采用的是try/catch,但是这带来一个问题，如果我们不处理异常，PHP会以错误的信息显示这个异常，这肯定不是我们想要的结果，所以我们的做法就是定义一个全局异常处理器来帮助我们处理未捕获的异常。

怎么做的呢，我们可以修改app目录下的`provider.php`容器配置文件，在这里可以注册一个自己的异常处理器，默认TP提供一个异常处理器。

我们不要去修改TP默认的异常处理器，我们应该自己建立一个，因为我们之前使用多应用，所以我们可以在每个应用下建立新的异常处理器。

我们先复制`app\provider.php`文件到`app\api`目录下，之后在`app\api`目录下新建`exception`目录，然后在这个目录下新建一个`ExceptionHandle.php`文件。结构如下：
```
api
├─provider.php    
├─exception    
│  ├─ExceptionHandle.php          
```
新建的`ExceptionHandle.php`文件内容，我们可以参考`app`目录下的同名文件。
```php
// app/api/exception/ExceptionHandle
namespace app\api\exception;


use think\exception\Handle;
use think\Response;
use Throwable;

class ExceptionHandle extends  Handle
{

    /**
     * Render an exception into an HTTP response.
     *
     * @access public
     * @param \think\Request   $request
     * @param Throwable $e
     * @return Response
     */
    public function render($request, Throwable $e): Response
    {
        // 添加自定义异常处理机制
         return show(config('status.success'),$e->getMessage());
    }
}
```
上面就是我们新建的异常类，异常类要注册到容器中，不然使用的还是app下的异常处理器，我们在上面新建的`provider.php`文件修改即可。
```php
// 容器Provider定义文件
return [
    'think\exception\Handle' => app\api\exception\ExceptionHandle::class,
];
```
这样在我们访问`tp.test/api/dasf`使就会响应一个500状态的json数据。

这里需要注意的一点就是，如果我们的异常处理器类出现了错误，是不会显示错误信息的，这个时候浏览器只会看到500错误，看错误信息需要到PHP的日志里查看。

### 查看PHP的错误日志
由于我们对PHP-FPM不了解，导致PHP中即便设置了日志的路径还是不能生成日志。这里我把自己的思路记录一下，首先使编辑php.ini文件，这个文件在哪里可以用命令查看。`php -i | grep php.ini`,这里我们的配置文件在`/etc/php/7.2/cli/php.ini`,在文件中新增如下内容：
```
# 开启日志记录
log_errors = On
error_log = "/var/log/php/7.2/error_log.log"
# 设置错误报告级别
error_reporting = E_ALL

# 关闭错误显示
display_errors = Off
```
上面设置好了后发现并没有错误日志在文件中生成，我们发现需要修改php-fpm的配置,配置文件`/etc/php/7.2/fpm/php-fpm.conf`,添加如下内容：
```php

; php-fpm的错误日志
error_log = /var/log/php7.2-fpm.log
;打开worker进程的错误输出
catch_workers_output = yes
```
这样的问题就是PHP的错误信息放在了php-fpm的错误日志里，具体原因现在还没有找到，网上把如下参数注释了就回去加载php.ini中的配置,改了还是显示在fpm的错误日志里。
```
;php_admin_value[sendmail_path] = /usr/sbin/sendmail -t -i -f www@my.domain.com
;php_flag[display_errors] = off
;php_admin_value[error_log] = /var/log/fpm-php.www.log
;php_admin_flag[log_errors] = on
;php_admin_value[memory_limit] = 32M
```
后续遇到这方面的问题我们再讨论，这样一来我们就可以查看我们在编写异常处理器时遇到的问题。

如果异常被我们捕获处理了或者在上面我们编写的异常处理器中处理了，就不会把错误写入到PHP的错误日志中了。

## 页面布局
静态资源我们都会放在`public/static`目录下，比如后台的静态资源会在`static/admin`下，然后我们在`config/view.php`文件中添加替换规则。
```php
// 替换规则
    'tpl_replace_string' =>[
        '{__STATIC_PATH}' => '/static/'
    ],
```
这样定义后我们就可以去需改模板文件中的静态资源路径了，这样的好处就在我们可以动态的修改静态资源存放的路径，而不用去修改html。
```html
     <link rel="stylesheet" href="{__STATIC_PATH}admin/css/layuimini.css" media="all">
```
TP6的模板引擎是模块话设置，需要安装才可以使用。
```
composer require topthink/think-view
```
这样我们就可以渲染模板，并加载静态资源了。