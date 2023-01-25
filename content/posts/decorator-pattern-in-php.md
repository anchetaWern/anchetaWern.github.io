---
author: "Wern Ancheta"
title: "PHP Design Patterns: Decorator Pattern"
date: "2023-01-08"
---

This is the first post in a series of articles that will walk you through how to implement design patterns in PHP.

In this post, we'll take a look at how you can implement the decorator pattern in PHP.
You can use the decorator pattern to change or adjust the behavior of an existing object.

To quickly understand how the decorator pattern works, we will use a customizable product as an example.
We will create an interface and it will serve as the contract for which a product or any of its decorators will abide to.
In this case, all products needs to implement a `getPrice()` method:

```php
<?php 
// decoratorpattern/app/Product.php

namespace App;

interface Product 
{
    public function getPrice(): float;
}
```

Next, we can now create a class for a specific type of `Product`:

```php
<?php 
// decoratorpattern/app/Yoyo.php

namespace App;

class Yoyo implements Product
{
    const PRICE = 10.99;

    public function getPrice(): float
    {
        return self::PRICE;
    }
}
```

Next, we create a decorator for the product type we just created. This is where the bulk of the decorator pattern is implemented:

```php
<?php 
// decoratorpattern/app/YoyoDecorator.php

namespace App;

abstract class YoyoDecorator implements Product
{
    public function __construct(public Product $product) {}

    public function getPrice(): float
    {
        return $this->product->getPrice();
    }
}
```

In the constructor, you need to accept an instance of a `Product` as an argument. If you're new to PHP 8, this uses the constructor property promotion syntax. So it's the shorthand for this syntax:

```php 
<?php

public Product $product;

public function __construct(Product $product) 
{
    $this->product = $product;
}
```

This then allows you to return the price of whatever product was passed to this decorator:

```php
<?php
public function getPrice(): float
{
    return $this->product->getPrice();
} 
```

Next, we can now create decorators for our product. In this case, we want to allow users to add a custom bearing to a yoyo. Here we're simply going to add a custom amount to the price of the original product. We have access to the product because it's being passed as an argument to the constructor of the abstract class we're extending from:

```php
<?php 
// decoratorpattern/app/CustomBearing.php
namespace App;

class CustomBearing extends YoyoDecorator 
{
    const BEARING_PRICE = 2.50;

    public function getPrice(): float
    {
        return $this->product->getPrice() + self::BEARING_PRICE;
    }
} 
```

At this point, we can now test out if the decorator is indeed working. First we verify if the price of the yoyo is correct. Then we apply the custom bearing decorator and see if the price is updated accordingly:

```php
<?php 
// decoratorpattern/DecoratorTest.php

require_once __DIR__ . '/vendor/autoload.php';

use PHPUnit\Framework\TestCase;

class DecoratorTest extends TestCase
{
    public function test_yoyo_price() : void
    {
        $yoyo = new App\Yoyo();
        $this->assertEquals($yoyo->getPrice(), 10.99);

        $yoyo_with_custom_bearing = new App\CustomBearing($yoyo);
        $this->assertEquals($yoyo_with_custom_bearing->getPrice(), 13.49);
    }
}
```

To demonstrate how can we use a combination of different decorators, let's create a couple more decorators:

```php
<?php 
// decoratorpattern/app/Decorators/TransparentCap.php
namespace App;

class TransparentCap extends YoyoDecorator
{
    const TRANSPARENT_CAP_PRICE = 1.25;

    public function getPrice(): float
    {
        return $this->product->getPrice() + self::TRANSPARENT_CAP_PRICE;
    }
} 
```

```php 
<?php 
// decoratorpattern/app/Decorators/CustomSpacers.php
namespace App;

class CustomSpacers extends YoyoDecorator 
{
    const SPACER_PRICE = 5.10;

    public function getPrice(): float
    {
        return $this->product->getPrice() + self::SPACER_PRICE;
    }
}
```

Here's how you can then use different combinations of decorators:

```php
<?php 
// decoratorpattern/DecoratorTest.php

public function test_yoyo_price() : void
{
    // ...

    $yoyo_with_transparent_cap = new App\TransparentCap($yoyo);
    $this->assertEquals($yoyo_with_transparent_cap->getPrice(), 12.24);

    $yoyo_with_custom_bearing_and_transparent_cap = new App\TransparentCap(new App\CustomBearing($yoyo));
    $this->assertEquals($yoyo_with_custom_bearing_and_transparent_cap->getPrice(), 14.74);

    $yoyo_with_custom_bearing_and_custom_spacers = new App\CustomSpacers(new App\CustomBearing($yoyo));
    $this->assertEquals($yoyo_with_custom_bearing_and_custom_spacers->getPrice(), 18.59);

    $yoyo_with_custom_bearing_and_custom_spacers_and_transparent_cap = new App\TransparentCap(new App\CustomSpacers(new App\CustomBearing($yoyo)));
    $this->assertEquals($yoyo_with_custom_bearing_and_custom_spacers_and_transparent_cap->getPrice(), 19.84); 

}
```

This works because the YoyoDecorator class implements the same interface as the original object that we are working with. So no matter how many decorators we wrap the original object with, it will still allow us to get the correct price.

To run this. Be sure you have the following on your `composer.json` file then run `composer dump-autoload`:

```
"autoload": {
    "psr-4": {
        "DecoratorPattern\\App\\": "decoratorpattern/app/"
    }
},
```

Also install phpunit (`composer require phpunit/phpunit`) if you haven't done so already.

Then you can run the tests:

```bash
vendor/bin/phpunit decoratorpattern/DecoratorTest.php
```

You can find the source code used in this tutorial on this [GitHub repo](https://github.com/anchetaWern/php-design-patterns).

