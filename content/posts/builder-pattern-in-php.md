---
author: "Wern Ancheta"
title: "PHP Design Patterns: Builder Pattern"
date: "2023-04-16"
---

This is the 15th post in a series of articles that will walk you through how to implement design patterns in PHP.

In this part, we're going to take a look at how you can implement the Builder Pattern.

The Builder Pattern is a creational design pattern which allows for the creation of related objects without having to repeat yourself.

For example, when creating a sandwich you'll have many different types and many different combinations of its components that you can put. But in the end they're all just a sandwich so they're made up of the same type of stuff:

- **Bread** - the type of bread you use as the base of the sandwich.
- **Spread** - what you use to coat the innerside of the bread which will serve as the base flavor.
- **Filling** - this is what usually decides what type of sandwich you're making. If the filling is tuna then its a tuna sandwich. If it's chicken then chicken sandwich.
- **Garnish** - what you put on top of the sandwich.

With that in mind, you can probably create a `Sandwich` class like so:

```php 
<?php 

class Sandwich 
{
    public function __construct($bread, $spread, $filling, $garnish)
    {
        // ...
    }
}
```

Then you can initialize a new object like so:

```php
<?php 

$sandwich = new Sandwich('wheat', 'mayo', 'tuna', 'onion');
```

And that will serve you fine. But what if the garnish and the filling is optional? You can probably have it like so:

```php 
<?php

public function __construct($bread, $spread, $filling = false, $garnish = false)
{

}
```

The disadvantage with this is that you'll have to remember which parameters are required and which ones are not. With this example, it isn't actually too painful since we only have four unique options. But what if you have 30 different options? You can probably create a subclass for that:


```php
<?php
class TunaSandwich extends Sandwich
{

    public function __construct($bread, $spread, $filling, $garnish = false)
    {

    }

}


class PeanutButterSandwich extends Sandwich
{

    public function __construct($bread, $spread, $filling = false, $garnish = false)
    {
        
    }

}

class EggSandwich extends Sandwich
{

    public function __construct($bread, $spread, $filling, $garnish = false)
    {
        
    }

}
```

As you can see, we inadvertently introduced some amount of repetition, especially with those sandwiches that accept the same options. And if we scale this up further (eg. 30 different sandwiches with multiple spreads, fillings, and garnishes) then you'll have something that looks like this:

```php 
<?php

class BanhMi extends Sandwich
{

    public function __construct($bread, $spread, $filling1, $filling2 = false, $filling3 = false, $garnish1, $garnish2 = false, $garnish3 = false, $garnish4 = false)
    {

    }
}
```

The Builder Pattern provides us with a solution to this problem. It involves:

1. Breaking down the object construction into a series of steps - for sandwich construction, this will be adding the bread, spread, filling, and garnish.
2. Representing the different types of sandwiches as their own class - for example: tuna sandwich, falafel, banh mi.
3. Representing the different components as their own class - for example, when creating a Banh Mi (Vietnamese Sandwich) we can have a French baguette, white onion, mayonaise, and cilantro as their own component. 
4. Creating a separate sandwich builder for each type of sandwich - for example: Tuna sandwich builder, egg sandwich builder.
5. The director - responsible for calling all the steps created in step 1 and returning the actual sandwich. 


Now let's proceed to the implementation.

First, we need to create the base representation of a sandwich. This is where all the other types of sandwich will extend from. It has a `setComponent` method for setting the different components mentioned in step 3. This accepts a `$key` which serves as the unique array key where we store the component and the actual component as the second argument.

On the other hand, the `getComponent` method will be mainly used for testing so we can verify if the sandwich got the correct components:

```php 
<?php
// builderpattern/app/Sandwich/Sandwich.php

namespace BuilderPattern\App\Sandwich;

use BuilderPattern\App\Sandwich\Components\Component;

abstract class Sandwich {
  
    private array $components = [];
  
    public function setComponent($key, Component $component) 
    {
        $this->components[$key] = $component;
    }

    public function getComponent($key)
    {
        return $this->components[$key];
    }
}
```

We use a `Component` interface for grouping:

```php
<?php 
// builderpattern/app/Sandwich/Components/Component.php

namespace BuilderPattern\App\Sandwich\Components;

interface Component 
{

} 
```

At this point, you can also create a class for the different types of sandwich mentioned in step 2:

```php
<?php 
// builderpattern/app/Sandwich/TunaSandwich.php

namespace BuilderPattern\App\Sandwich;

use BuilderPattern\App\Sandwich\Sandwich;


class TunaSandwich extends Sandwich 
{

} 
```

For Step 3, you can simply implement the `Component` interface and name each one differently.

Bread:

```php 
<?php 
// builderpattern/app/Sandwich/Components/Bread/WheatBread.php

namespace BuilderPattern\App\Sandwich\Components\Bread;

use BuilderPattern\App\Sandwich\Components\Component;

class WheatBread implements Component
{

}
```

Spread:

```php
<?php 
// builderpattern/app/Sandwich/Components/Spread/Mayo.php

namespace BuilderPattern\App\Sandwich\Components\Spread;

use BuilderPattern\App\Sandwich\Components\Component;

class Mayo implements Component
{

}
```

Filling:

```php 
<?php 
// builderpattern/app/Sandwich/Components/Filling/Tuna.php

namespace BuilderPattern\App\Sandwich\Components\Filling;

use BuilderPattern\App\Sandwich\Components\Component;

class Tuna implements Component
{

}
```

Garnish:

```php
<?php 
// builderpattern/app/Sandwich/Components/Garnish/Cilantro.php

namespace BuilderPattern\App\Sandwich\Components\Garnish;

use BuilderPattern\App\Sandwich\Components\Component;

class Cilantro implements Component
{

} 
```


Next, we can now move on to step 4 which is to create a builder for each type of sandwich we want to make. In this case, we want to make a tuna sandwich builder. But before we proceed to actually implementing that, we first need to create the `Builder` interface which allows us to specify the methods (aka actions) required for creating a sandwich. These are basically the steps we defined on step 1 with the addition of the `createSandwich` (for initiating the creation of a sandwich) and `getSandwich` method (for getting the final state of the sandwich):

```php 
<?php
// builderpattern/app/Builders/Builder.php

namespace BuilderPattern\App\Builders;

use BuilderPattern\App\Sandwich\Sandwich;

interface Builder
{
    public function createSandwich();

    public function addBread();

    public function addSpread();

    public function addFilling();

    public function addGarnish();

    public function getSandwich() : Sandwich;
}
```

Then in the `TunaSandwichBuilder`, we can specify which components we want to add to the sandwich. This is now simple because we've broken down the steps that are necessary to build a sandwich. The `createSandwich` method is where we specify the type of sandwich we want to build. In this case it's a `TunaSandwich`. Note that the `TunaSandwich` class is extending from the abstract `Sandwich` class where we defined the `setComponent` method for setting the different components that makes up a sandwich. That method is what we're calling on every action. Some sandwiches might not even have a filling or a garnish. In those cases, you can simply create a method with no body. Finally, we have the `getSandwich` method which simply returns the finished product:


```php
<?php
// builderpattern/app/Builders/TunaSandwichBuilder.php

namespace BuilderPattern\App\Builders;

use BuilderPattern\App\Builders\Builder;

use BuilderPattern\App\Sandwich\Sandwich;
use BuilderPattern\App\Sandwich\TunaSandwich;

use BuilderPattern\App\Sandwich\Components\Bread\WheatBread;
use BuilderPattern\App\Sandwich\Components\Filling\Tuna;
use BuilderPattern\App\Sandwich\Components\Spread\Mayo;
use BuilderPattern\App\Sandwich\Components\Garnish\Cilantro;

class TunaSandwichBuilder implements Builder 
{

    private $sandwich;


    public function createSandwich()
    {
        $this->sandwich = new TunaSandwich;
    }


    public function addBread()
    {
        $this->sandwich->setComponent('bread', new WheatBread);
    }


    public function addSpread()
    {
        $this->sandwich->setComponent('spread', new Mayo);
    }


    public function addFilling()
    {
        $this->sandwich->setComponent('filling', new Tuna);
    }


    public function addGarnish()
    {
        $this->sandwich->setComponent('garnish', new Cilantro);
    }


    public function getSandwich() : Sandwich
    {
        return $this->sandwich;
    }

} 
```

Next is the fifth step where we create a director which brings everything together. It's sole responsibility is to call all the actions defined in a `Builder` instance and return the final output at the end:

```php 
<?php 
// builderpattern/app/Director.php

namespace BuilderPattern\App;

use BuilderPattern\App\Sandwich\Sandwich;
use BuilderPattern\App\Builders\Builder;

class Director 
{
    public function build(Builder $builder): Sandwich 
    {
        $builder->createSandwich();
        $builder->addBread();
        $builder->addSpread();
        $builder->addFilling();
        $builder->addGarnish();

        return $builder->getSandwich();
    }
}
```

Now we're ready to test things out. First let's take a look at how it's done if you don't use the Builder design pattern:

```php 
<?php 
// builderpattern/BuilderTest.php

require_once __DIR__ . '/../vendor/autoload.php';

use PHPUnit\Framework\TestCase;

use BuilderPattern\App\Director;
use BuilderPattern\App\Builders\TunaSandwichBuilder;

use BuilderPattern\App\Sandwich\TunaSandwich;

use BuilderPattern\App\Sandwich\Components\Bread\WheatBread;
use BuilderPattern\App\Sandwich\Components\Spread\Mayo;
use BuilderPattern\App\Sandwich\Components\Filling\Tuna;
use BuilderPattern\App\Sandwich\Components\Garnish\Cilantro;

class BuilderTest extends TestCase
{
    public function test_non_builder() : void 
    {
        $tuna_sandwich = new TunaSandwich;
        $tuna_sandwich->setComponent('bread', new WheatBread);
        $tuna_sandwich->setComponent('spread', new Mayo);
        $tuna_sandwich->setComponent('filling', new Tuna);
        $tuna_sandwich->setComponent('garnish', new Cilantro);

        $this->assertInstanceOf(WheatBread::class, $tuna_sandwich->getComponent('bread'));
        $this->assertInstanceOf(Mayo::class, $tuna_sandwich->getComponent('spread'));
        $this->assertInstanceOf(Tuna::class, $tuna_sandwich->getComponent('filling'));
        $this->assertInstanceOf(Cilantro::class, $tuna_sandwich->getComponent('garnish'));
    }

}
```

As you can see from the above code, we'll have to call the `setComponent` method within the client code. Which isn't too bad until you have 30 different components to set. And you'll basically have to remember which components makes up the object you want to build.

In contrast, here's how we would do it with the Builder pattern using the `Director` and the `TunaSandwichBuilder`:

```php
public function test_builder() : void
{
    $director = new Director;
    $tuna_sandwich = $director->build(new TunaSandwichBuilder);

    $this->assertInstanceOf(WheatBread::class, $tuna_sandwich->getComponent('bread'));
    $this->assertInstanceOf(Mayo::class, $tuna_sandwich->getComponent('spread'));
    $this->assertInstanceOf(Tuna::class, $tuna_sandwich->getComponent('filling'));
    $this->assertInstanceOf(Cilantro::class, $tuna_sandwich->getComponent('garnish'));

}
```

To run this. Be sure to add the following on your `composer.json` file then run `composer dump-autoload`:

```
"autoload": {
    "psr-4": {
        "BuilderPattern\\App\\": "builderpattern/app/"
    }
},
```

Also install phpunit (`composer require phpunit/phpunit`) if you haven't done so already.

You can then run it by executing the following:

```bash
vendor/bin/phpunit builderpattern/app/BuilderTest.php
```

That's it for the Builder Pattern. You can reach for this pattern whenever you find yourself needing to construct objects that have the same basic construct but is different enough that you would require a different class for each one. In the real-world, you can use the Builder Pattern to well, build anything. From an SQL query builder to an HTML form builder.

You can find the source code used in this tutorial on this [GitHub repo](https://github.com/anchetaWern/php-design-patterns).

