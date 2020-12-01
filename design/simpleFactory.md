## 简单工厂模式
简单工厂模式是一个精简版的工厂模式 ,它与静态工厂模式最大的区别是它不是『静态』的

因为非静态，所以你可以拥有多个不同参数的工厂，你可以为其创建子类.

它比静态工厂模式受欢迎

## 代码
```php
class SimpleFactory
{
    public function createBicycle(){
       return new Bicycle();
    }
}

class Bicycle
{
    public function  dump($brand){
        echo "这是{$brand}品牌的车";
    }
}


$factory = new SimpleFactory();
$bicycle =$factory->createBicycle();
$bicycle->dump('永久');

```