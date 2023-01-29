---
author: "Wern Ancheta"
title: "PHP Design Patterns: Strategy Pattern"
date: "2023-01-29"
---

This is the fourth post in a series of articles that will walk you through how to implement design patterns in PHP.

In this part, we're going to take a look at how you can implement the strategy pattern.

The strategy pattern can be used when you have multiple ways to implement something. A common example of this is mail. 
To send an email from your application, you most likely need to use mail services like Mailgun, Amazon Simple Email Service, Mandrill, SparkPost, etc.

The Strategy pattern is a way to encapsulate these family of classes and make them interchangeable. 

Say we have a `Person` class which uses a cooking method to cook food:

```php
<?php 
// strategypattern/app/Person.php

namespace StrategyPattern\App;

class Person
{
    public function makeDinner(string $food, $cookingMethod)
    {
        return $cookingMethod->cook($food);
    }
}
```

Now we have multiple ways to cook food, each represented by its own class:

```php 
<?php 
// strategypattern/app/Roast.php

namespace StrategyPattern\App;

class Roast
{
    public function cook($food): string 
    {
        return 'roasting ' . $food;
    }
}
```

```php
<?php 
// strategypattern/app/Boil.php

namespace StrategyPattern\App;

class Boil
{
    public function cook($food): string 
    {
        return 'boiling ' . $food;
    }
}
```

```php
<?php 
// strategypattern/app/Fry.php

namespace StrategyPattern\App;

class Fry
{
    public function cook($food): string 
    {
        return 'frying ' . $food;
    }
}
```

To test this, we call on the different strategies to cook a food:

```php
<?php 
// strategypattern/StrategyTest.php

require_once __DIR__ . '/../vendor/autoload.php';

use PHPUnit\Framework\TestCase;

class StrategyTest extends TestCase
{
    public function test_cooking_methods() : void
    {
        $person = new StrategyPattern\App\Person();
    
        $this->assertEquals($person->makeDinner('fish', new StrategyPattern\App\Fry), 'frying fish');
        $this->assertEquals($person->makeDinner('fish', new StrategyPattern\App\Boil), 'boiling fish');
        $this->assertEquals($person->makeDinner('fish', new StrategyPattern\App\Roast), 'roasting fish');
    }
}
```

Obviously, this will work since all three classes have the same API. But what if later on a new developer comes in and implements a new cooking strategy?

```php
<?php 
// strategypattern/app/Grill.php

namespace StrategyPattern\App;

class Grill
{
    public function cookWithMetalBars($food): string 
    {
        return 'grilling ' . $food;
    }
}
```

Add it to the test and see for yourself what happens:

```php
<?php
public function test_cooking_methods() : void
{
    $person = new StrategyPattern\App\Person();

    // ...

    $this->assertEquals($person->makeDinner('fish', new StrategyPattern\App\Grill), 'grilling fish');
}
```

You get an error that looks like this:

```
Error: Call to undefined method StrategyPattern\App\Grill::cook()
```

Since the `Grill` class is using the `cookWithMetalBars` method instead of the `cook` method.

This is where the strategy pattern comes in. When you have multiple classes that does a similar thing, it's worth having them adhere to a single interface. This is done as a way to constrain all new classes so they need to implement specific methods. In this case, we're creating a `CookingMethod` interface which all the cooking strategies needs to abide to. The only thing we ask of the classes is for them to implement a `cook` method:

```php 
<?php 
// strategypattern/app/CookingMethod.php

namespace StrategyPattern\App;

interface CookingMethod
{
    public function cook(string $food): string;
}
```

So now we can update all the classes to implement the `CookingMethod` interface:

```php 
<?php 
// strategypattern/app/Roast.php

namespace StrategyPattern\App;

class Roast implements CookingMethod
{
    public function cook($food): string 
    {
        return 'roasting ' . $food;
    }
}
```

```php
<?php 
// strategypattern/app/Boil.php

namespace StrategyPattern\App;

class Boil implements CookingMethod
{
    public function cook($food): string 
    {
        return 'boiling ' . $food;
    }
}
```

```php
<?php 
// strategypattern/app/Fry.php

namespace StrategyPattern\App;

class Fry implements CookingMethod
{
    public function cook($food): string 
    {
        return 'frying ' . $food;
    }
}
```

Of course we now need to have the `Grill` class implement the `cook` method:

```php
<?php 
// strategypattern/app/Grill.php

namespace StrategyPattern\App;

class Grill implements CookingMethod
{
    public function cook($food): string 
    {
        return 'grilling ' . $food;
    }
}
```

At this point, when you run the tests it will now pass.


To run this. Be sure to add the following on your `composer.json` file then run `composer dump-autoload`:

```
"autoload": {
    "psr-4": {
        "StrategyPattern\\App\\": "strategypattern/app/"
    }
},
```

Also install phpunit (`composer require phpunit/phpunit`) if you haven't done so already.

You can then run it by executing the following:

```bash
vendor/bin/phpunit strategypattern/app/StrategyTest.php
```

That's really all there is to the Strategy pattern. It's simply a way to create a common API for related classes so that the client coder can easily swap them out.

You can find the source code used in this tutorial on this [GitHub repo](https://github.com/anchetaWern/php-design-patterns).

