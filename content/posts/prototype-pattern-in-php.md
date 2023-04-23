---
author: "Wern Ancheta"
title: "PHP Design Patterns: Prototype Pattern"
date: "2023-04-23"
---

This is the 16th post in a series of articles that will walk you through how to implement design patterns in PHP.

In this part, we're going to take a look at how you can implement the Prototype Pattern.


The Prototype Pattern is used for creating new objects by means of cloning. Since you no longer have to create individual classes, this effectively replaces inheritance with composition.

To implement the Prototype Pattern, we make use of the `clone` keyword to make shallow copies of objects. The clone will then have the same properties as its source.

Here we have a `Ninja` class. This is the class whose instance we will be cloning later on. When instantiating a new object, you pass in the name, chakra level, and the techniques of the ninja. We also have the `setChakra` and `setTechniques` methods which we'll be using later on to modify the cloned object. Lastly, we can specify the cloning behavior via the `__clone` magic method. Here we're simply changing the name to include "Copy of " plus the name that was given to the original:

```php 
<?php 
// prototypepattern/app/Ninja.php

namespace PrototypePattern\App;

use PrototypePattern\App\NinjaInterface;

class Ninja implements NinjaInterface
{

    public function __construct(public string $name, public int $chakra, public array $techniques)
    {

    }


    public function setChakra(int $chakra)
    {
        $this->chakra = $chakra;
    }


    public function setTechniques(array $techniques)
    {
        $this->techniques = $techniques;
    }


    public function __clone()
    {
        $this->name = "Copy of " . $this->name;
    }

}
```

We're implementing a `NinjaInterface` because we'll be creating a factory for the `Ninja` class:


```php 
<?php 
// prototypepattern/app/NinjaInterface.php

namespace PrototypePattern\App;

interface NinjaInterface 
{

    
}
```

Here's the `NinjaFactory` which allows us to create copies of the original object that was passed. We're implemeting the prototype pattern through these three methods: `emptySlateNinja`, `ninjaWithChakra`, and `ninjaWithChakraAndTechniques` which is basically returning the same properties as the original:


```php
<?php 
// prototypepattern/app/Factories/NinjaFactory.php

namespace PrototypePattern\App\Factories;

use PrototypePattern\App\NinjaInterface;

class NinjaFactory
{

    public function __construct(private NinjaInterface $ninja)
    {
        
    }

    public function emptySlateNinja()
    {
        $ninja = clone $this->ninja; 
        $ninja->setChakra(0);
        $ninja->setTechniques([]);
        return $ninja;
    }


    public function ninjaWithChakra()
    {
        $ninja = clone $this->ninja;
        $ninja->setTechniques([]);
        return $ninja;
    }


    public function ninjaWithChakraAndTechniques()
    {
        return clone $this->ninja;
    }

}
```

To use it, simply create a new instance of the `Ninja` class and pass it to the `NinjaFactory`. From there, you can call any of the three methods I mentioned earlier to create a clone of the original `$ninja` object with different options:

```php 
<?php 
// prototypepattern/PrototypeTest.php

require_once __DIR__ . '/../vendor/autoload.php';

use PHPUnit\Framework\TestCase;



class PrototypeTest extends TestCase
{
    public function test_prototype() : void 
    {
        $ninja = new PrototypePattern\App\Ninja('Naruto', 100, ['rasengan', 'kage bunshin'], false, false);
        $ninjaFactory = new PrototypePattern\App\Factories\NinjaFactory($ninja);
        
        $this->assertEquals($ninjaFactory->emptySlateNinja()->chakra, 0);
        $this->assertEquals($ninjaFactory->emptySlateNinja()->techniques, []);

        $this->assertEquals($ninjaFactory->ninjaWithChakra()->chakra, 100);
        $this->assertEquals($ninjaFactory->ninjaWithChakra()->techniques, []);

        $this->assertEquals($ninjaFactory->ninjaWithChakraAndTechniques()->chakra, 100);
        $this->assertEquals($ninjaFactory->ninjaWithChakraAndTechniques()->techniques, ['rasengan', 'kage bunshin']);
    }
}
```


To run this. Be sure to add the following on your `composer.json` file then run `composer dump-autoload`:

```
"autoload": {
    "psr-4": {
        "ProtoTypePattern\\App\\": "prototypepattern/app/"
    }
},
```

Also install phpunit (`composer require phpunit/phpunit`) if you haven't done so already.

You can then run it by executing the following:

```bash
vendor/bin/phpunit prototypepattern/app/PrototypeTest.php
```

That's pretty much what the Prototype pattern is all about. It's simply a way to bypass having to create separate objects and passing in different options to it. Instead, we just clone existing objects and modify them accordingly. It's commonly used in tandem with the Factory pattern.


You can find the source code used in this tutorial on this [GitHub repo](https://github.com/anchetaWern/php-design-patterns).

