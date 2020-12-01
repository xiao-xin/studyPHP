## 静态工厂模式
与抽象工厂模式类似，此模式用于创建一系列相关或相互依赖的对象。 

只使用一个静态方法来创建所有类型对象， 此方法通常被命名为 factory 或 build。


## 代码
```php
final  class StaticFactory
{
    public static function  factory ($type)
    {
        switch ($type) {
            case 'number':
                return new FormatNumber();
                break;
            case 'string':
                return new FormatString();
                break;
            default:
                throw new  \InvalidArgumentException('unknown format given');
        }
    }
}

interface IFormat {
    public function  display($val);
}

class FormatNumber implements  IFormat {
    public function display($val)
    {
        var_dump(intval($val));
    }
}

class FormatString implements  IFormat {
    public function display($val)
    {
        var_dump(strval($val));
    }
}

$number = 100.00;
$format  = StaticFactory::factory('number');
$format->display($number);
$format  = StaticFactory::factory('string');
$format->display($number);
```