---
author: "Wern Ancheta"
title: "PHP Design Patterns: Factory Pattern"
date: "2023-03-05"
---

This is the ninth post in a series of articles on how to implement design patterns in PHP.

In this part, we'll see how we can implement the Factory Pattern.

This pattern is used for returning an instance of another classe. Why would anyone want to create a separate class which just returns another class you say? That’s because oftentimes you’re not just dealing with a single class. More often that not, two or more class instances (or just their properties) is required to create an instance of another class. That’s when the Factory Pattern comes in handy. You don't want to be creating an object out of those dependencies everytime you need to create an object for a specific class.

This pattern is commonly used for creating dummy data for seeding your database. But you can basically use it anywhere which requires you to put together multiple objects to create another object.

Let's say we have a class for creating potting mixes for planting. Potting mixes are composed of garden soil, carbonized rice hull, and vermicast. Initializing an object for each one requires a `ratio` to be passed to it so let's create an abstract class which allows us to inherit this behavior:

```php
<?php 
// factorypattern/app/GrowingMedium.php

namespace FactoryPattern\App;

abstract class GrowingMedium
{
    public function __construct(public int $ratio) {

    }
} 
```

Next, create a separate class for each of the growing mediums:


```php
<?php 
// factorypattern/app/GardenSoil.php

namespace FactoryPattern\App;

class GardenSoil extends GrowingMedium 
{
    
} 
```

```php
<?php 
// factorypattern/app/CarbonizedRiceHull.php

namespace FactoryPattern\App;

class CarbonizedRiceHull extends GrowingMedium 
{

} 
```


```php
<?php 
// factorypattern/app/Vermicast.php

namespace FactoryPattern\App;

class Vermicast extends GrowingMedium 
{
 
}
```

Lastly, create the `PottingMix` class. This requires three growing mediums to be passed to it when initializing. We also have a `getTotal` function which returns the sum of the ratios for all of the growing mediums that was passed in the constructor:

```php
<?php 
// factorypattern/app/PottingMix.php

namespace FactoryPattern\App;

use FactoryPattern\App\GrowingMedium;

class PottingMix 
{

    public function __construct(public GrowingMedium $medium_one, public GrowingMedium $medium_two, public GrowingMedium $medium_three)
    {

    }

    public function getTotal() : int
    {
        return array_sum([$this->medium_one->ratio, $this->medium_two->ratio, $this->medium_three->ratio]);
    }

} 
```

With the normal setup, you would have to initialize an object for all three classes and then passing them as an argument to the `PottingMix` class:


```php
<?php 
// factorypattern/app/FactoryTest.php

require_once __DIR__ . '/../vendor/autoload.php';

use PHPUnit\Framework\TestCase;

class FactoryTest extends TestCase
{
    public function test_nonfactory() : void
    {
        $garden_soil = new FactoryPattern\App\GardenSoil(40);
        $crh = new FactoryPattern\App\CarbonizedRiceHull(30);
        $vermicast = new FactoryPattern\App\Vermicast(30);

        $potting_mix = new FactoryPattern\App\PottingMix($garden_soil, $crh, $vermicast);

        $this->assertEquals($potting_mix->getTotal(), 100);
    }

}
```

The code above is fine, as long as you only use this type of call once. But as soon as you find yourself repeating the same thing, that's when you reach for the Factory Pattern. The basic idea is for you to create a class which does the initialization for you and passing all the dependencies it needs:


```php
<?php 
// factorypattern/app/PottingMixFactory.php

namespace FactoryPattern\App;

class PottingMixFactory 
{
    public static function create()
    {
        $garden_soil = new GardenSoil(40);
        $crh = new CarbonizedRiceHull(30);
        $vermicast = new Vermicast(30);

        return new PottingMix($garden_soil, $crh, $vermicast);
    }

}
```

Now, you will only have to reach for the PottingMixFactory to create a new potting mix:


```php 
<?php 
// factorypattern/FactoryTest.php

require_once __DIR__ . '/../vendor/autoload.php';

use PHPUnit\Framework\TestCase;

class FactoryTest extends TestCase
{
    // ...

    public function test_factory() : void
    {
        $potting_mix = FactoryPattern\App\PottingMixFactory::create();

        $this->assertEquals($potting_mix->getTotal(), 100);
    }

}
```

If the dependencies change for the main class you're trying to initialize, or the arguments passed to any of its dependencies changes then you can simply make that change in the corresponding factory.


To run this. Be sure you have the following on your `composer.json` file then run `composer dump-autoload`:

```json
"autoload": {
    "psr-4": {
        "FactoryPattern\\App\\": "factorypattern/app"
    }
}
```

Also install phpunit (`composer require phpunit/phpunit`) if you haven't done so already.

Then you can run the tests:

```bash
vendor/bin/phpunit factorypattern/FactoryTest.php
```

That's it for the Factory Pattern. It is simply a utility class for initializing objects.

You can find the source code used in this tutorial on this [GitHub repo](https://github.com/anchetaWern/php-design-patterns).