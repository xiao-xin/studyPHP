##  抽象工厂方法模式
在不指定具体类的情况下创建一系列相关或依赖对象。

通常创建的类都实现相同的接口。

抽象工厂的客户并不关心这些对象是如何创建的，它只是知道它们是如何一起运行的。

## 代码
```php
interface IProduct
{
    public function  calculatePrice():int;
}

class ShippableProduct implements  IProduct
{
    protected  $productPrice;
    protected  $shippingCosts;

    public function __construct(int $productPrice, int $shippingCosts)
    {
        $this->productPrice = $productPrice;
        $this->shippingCosts= $shippingCosts;
    }
    public function  calculatePrice():int{
        return $this->productPrice + $this->shippingCosts;
    }
}

class DigitalProduct implements  IProduct
{
    protected  $productPice ;

    public function  __construct($productPice)
    {
        $this->productPice = $productPice;
    }

    public function  calculatePrice():int{
        return $this->productPice;
    }
}

class ProductFactory
{
    public function createShippingProduct(int $price, int $shoppingCost)
    {
        return new ShippableProduct($price,$shoppingCost);
    }


    public function createDigitalProduct(int $price)
    {
        return new DigitalProduct($price);
    }
}


$productFactory = new ProductFactory();
$shoppingProduct = $productFactory->createShippingProduct(11,3);
echo $shoppingProduct->calculatePrice();

$digitalProduct = $productFactory->createDigitalProduct(11);
echo $digitalProduct->calculatePrice();
```