## 工厂方法模式
工厂方法模式，会为每个要创建的对象来建立一个子类。

这种模式是「真正」的设计模式， 因为他实现了 S.O.L.I.D 原则中「D」的 「依赖倒置」，在抽象工厂类中。

工厂方法模式取决于抽象类，而不是具体的类。 这是与简单工厂模式和静态工厂模式相比的优势。
## 代码
```php

interface ILogger
{
    public function log(string $messsage) ;
}

class StdoutLogger implements  ILogger {
    public function log (string $message){
        echo $message;
    }
}

class FileLogger implements ILogger {
    protected  $filepath;
    public function  __construct($filepath)
    {
        $this->filepath = $filepath;
    }

    public function  log(string $message) {
        file_put_contents($this->filepath,$message,FILE_APPEND);
    }
}

interface  ILoggerFactory
{
    public function createLogger():ILogger;
}

class StdoutLoggerFactory implements  ILoggerFactory
{
    public function createLogger():ILogger
    {
        return new StdoutLogger();
    }
}

class FileLoggerFactory implements  ILoggerFactory
{
    protected  $filepath ;
    public function __construct(string $filepath)
    {
        $this->filepath  = $filepath;
    }

    public function createLogger():ILogger
    {
        return new FileLogger($this->filepath);
    }
}

$stdoutLoggerFacotry = new StdoutLoggerFactory();
$stdoutLogger = $stdoutLoggerFacotry->createLogger();
$stdoutLogger->log('hello log');


$fileLoggerFacotry = new FileLoggerFactory('./log.text');
$fileLogger = $fileLoggerFacotry->createLogger();
$fileLogger->log('hello log');
```