---
author: "Wern Ancheta"
title: "PHP Design Patterns: Template Method Pattern"
date: "2023-02-05"
---

This is the fifth post in a series of articles that will walk you through how to implement design patterns in PHP.

In this part, we're going to take a look at how you can implement the Template Method Pattern.

You can use this pattern to reduce code duplication in cases where you have a set of related classes which does almost the same thing but not quite. This can be because each class needs to do one or more things just a bit differently. 

To run this. Be sure to add the following on your `composer.json` file then run `composer dump-autoload`:

```
"autoload": {
    "psr-4": {
        "TemplateMethodPattern\\App\\": "templatemethodpattern/app/"
    }
},
```

Also install phpunit (`composer require phpunit/phpunit`) if you haven't done so already.

Say we have three classes, each one pertaining to a way to create a smoothie. So we have a `HerbalSmoothie` for creating a smoothie whose base flavor uses herbs. A `VeggieSmoothie` which uses vegetables as base. And a `FruitySmoothie` whose base flavor uses fruits.

```php
<?php 
// templatemethodpattern/app/HerBalSmoothie.php

namespace TemplateMethodPattern\App;

class HerbalSmoothie
{
    private array $ingredients = [];

    public function blend(): array
    {
        $this
            ->addWater()
            ->addHerbs()
            ->addMilk()
            ->addSeeds();
        
        return $this->ingredients;
    }


    public function addWater()
    {
        $this->ingredients[] = 'water';
        return $this;
    }


    public function addHerbs()
    {
        $this->ingredients[] = 'herbs';
        return $this;
    }


    public function addMilk()
    {
        $this->ingredients[] = 'milk';
        return $this;
    }


    public function addSeeds()
    {
        $this->ingredients[] = 'seeds';
        return $this;
    }

} 
```

```php
<?php 
// templatemethodpattern/app/VeggieSmoothie.php

namespace TemplateMethodPattern\App;

class VeggieSmoothie
{
    private array $ingredients = [];

    public function blend(): array
    {
        $this
            ->addWater()
            ->addVeggies()
            ->addMilk()
            ->addSeeds();
        
        return $this->ingredients;
    }


    public function addWater()
    {
        $this->ingredients[] = 'water';
        return $this;
    }


    public function addVeggies()
    {
        $this->ingredients[] = 'veggies';
        return $this;
    }


    public function addMilk()
    {
        $this->ingredients[] = 'milk';
        return $this;
    }


    public function addSeeds()
    {
        $this->ingredients[] = 'seeds';
        return $this;
    }

} 
```

```php
<?php 
// templatemethodpattern/app/FruitySmoothie.php

namespace TemplateMethodPattern\App;

class FruitySmoothie
{
    private array $ingredients = [];

    public function blend(): array
    {
        $this
            ->addWater()
            ->addFruit()
            ->addMilk()
            ->addSeeds();
        
        return $this->ingredients;
    }


    public function addWater()
    {
        $this->ingredients[] = 'water';
        return $this;
    }


    public function addFruit()
    {
        $this->ingredients[] = 'main fruit';
        return $this;
    }


    public function addMilk()
    {
        $this->ingredients[] = 'milk';
        return $this;
    }


    public function addSeeds()
    {
        $this->ingredients[] = 'seeds';
        return $this;
    }

} 
```

The main method for each of class is called `blend`. This returns an array of ingredients that was used in that smoothie.

If you try out the code now, it will work:

```php 
<?php 
// templatemethodpattern/TemplateMethodTest.php

require_once __DIR__ . '/../vendor/autoload.php';

use PHPUnit\Framework\TestCase;

class TemplateMethodTest extends TestCase
{
    public function test_smoothie() : void
    {   
        $herbalSmoothie = new TemplateMethodPattern\App\HerbalSmoothie();
        $this->assertEquals($herbalSmoothie->blend(), [
            'water',
            'herbs',
            'milk',
            'seeds',
        ]);


        $fruitySmoothie = new TemplateMethodPattern\App\FruitySmoothie();
        $this->assertEquals($fruitySmoothie->blend(), [
            'water',
            'fruit',
            'milk',
            'seeds',
        ]);


        $veggieSmoothie = new TemplateMethodPattern\App\VeggieSmoothie();
        $this->assertEquals($veggieSmoothie->blend(), [
            'water',
            'veggies',
            'milk',
            'seeds',
        ]);
    }
}
```

To run this. Be sure to add the following on your `composer.json` file then run `composer dump-autoload`:

```
"autoload": {
    "psr-4": {
        "TemplateMethodPattern\\App\\": "templatemethodpattern/app/"
    }
},
```

Also install phpunit (`composer require phpunit/phpunit`) if you haven't done so already.

You can then run it by executing the following:

```bash
vendor/bin/phpunit templatemethodpattern/app/TemplateMethodTest.php
```


As you might have noticed, each of the classes are pretty much doing the same thing since there are ingredients that are common to each one (water, milk, and seeds). 

This is where the Template Method Pattern can help us.

The first step would be to create an abstract class which contains all the variables and methods common to all of the related classes:

```php 
<?php 
// templatemethodpattern/app/Smoothie.php

namespace TemplateMethodPattern\App;

abstract class Smoothie 
{
    protected array $ingredients = [];

    protected function addWater()
    {
        $this->ingredients[] = 'water';
        return $this;
    }


    protected function addMilk()
    {
        $this->ingredients[] = 'milk';
        return $this;
    }


    protected function addSeeds()
    {
        $this->ingredients[] = 'seeds';
        return $this;
    }

}
```

Then in each of the classes, you can now extend the parent class and then call the methods you need:

```php
<?php 
// templatemethodpattern/app/HerbalSmoothie.php

namespace TemplateMethodPattern\App;

class HerbalSmoothie extends Smoothie
{

    public function blend(): array
    {
        $this
            ->addWater()
            ->addHerbs()
            ->addMilk()
            ->addSeeds();
        
        return $this->ingredients;
    }


    public function addHerbs()
    {
        $this->ingredients[] = 'herbs';
        return $this;
    }
} 
```


```php
<?php 
// templatemethodpattern/app/VeggieSmoothie.php

namespace TemplateMethodPattern\App;

class VeggieSmoothie extends Smoothie
{

    public function blend(): array
    {
        $this
            ->addWater()
            ->addVeggies()
            ->addMilk()
            ->addSeeds();
        
        return $this->ingredients;
    }


    public function addVeggies()
    {
        $this->ingredients[] = 'veggies';
        return $this;
    }

}
```

```php
<?php 
// templatemethodpattern/app/FruitySmoothie.php

namespace TemplateMethodPattern\App;

class FruitySmoothie extends Smoothie
{

    public function blend(): array
    {
        $this
            ->addWater()
            ->addFruit()
            ->addMilk()
            ->addSeeds();
        
        return $this->ingredients;
    }


    public function addFruit()
    {
        $this->ingredients[] = 'fruit';
        return $this;
    }

} 
```

If you run the tests now, it will work but we're not quite there yet. If you noticed, we're still duplicating the `blend()` method across each class. They each have a different implementation, but they're still pretty much doing the same thing. So how could we get rid of this duplication?

In your parent class, you can delegate the implementation of the "odd" method to each of its subclasses:

```php
abstract class Smoothie 
{
    // ..

    abstract protected function addMainFlavor(); // add this

} 
```

Once you've done that, you can now move the `blend` method over to your main class and simply call the odd method from there:

```php
abstract class Smoothie 
{
    // ..
    public function blend(): array
    {
        $this
            ->addWater()
            ->addMainFlavor() // add this
            ->addMilk()
            ->addSeeds();
        
        return $this->ingredients;
    }
}
```

At this point, you can now update each of your subclasses to implement that method:

```php
<?php 
// templatemethodpattern/app/HerbalSmoothie.php

namespace TemplateMethodPattern\App;

class HerbalSmoothie extends Smoothie
{

    public function addMainFlavor()
    {
        $this->ingredients[] = 'herbs';
        return $this;
    }

} 
```

```php
<?php 
// templatemethodpattern/app/VeggieSmoothie.php

namespace TemplateMethodPattern\App;

class VeggieSmoothie extends Smoothie
{

    public function addMainFlavor()
    {
        $this->ingredients[] = 'veggies';
        return $this;
    }

} 
```


```php
<?php 
// templatemethodpattern/app/FruitySmoothie.php

namespace TemplateMethodPattern\App;

class FruitySmoothie extends Smoothie
{

    public function addMainFlavor()
    {
        $this->ingredients[] = 'fruit';
        return $this;
    }

} 
```

At this point when you run the tests it will run smoothie-ly (pun intended). 

That's really all there is to the Template Method Pattern. It's simply a way to DRY up your code by means of using an abstract class to implement the methods that are common to all of the related classes, and then leaving the methods that are unique to each class for them to implement.

Now the name "template method pattern" should make a bit sense because you're essentially creating the template of a method in the parent class so that its sub classes can implement it.

You can find the source code used in this tutorial on this [GitHub repo](https://github.com/anchetaWern/php-design-patterns).