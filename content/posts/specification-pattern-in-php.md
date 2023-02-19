---
author: "Wern Ancheta"
title: "PHP Design Patterns: Specification Pattern"
date: "2023-02-19"
---

This is the seventh post in a series of articles on how to implement design patterns in PHP.

This part, we'll go through the Specification Pattern. It is used when you want to upgrade some business rule into a first-class citizen. So you create a separate class for each business rule. At its core, all it does is check for a specific condition and returns either true or false. 

The most common example are the business rules surrounding a user. If you’re running a subscription-based service, you would have a business rule which gives a user specific privileges based on their occupation. And so you create a class that would represent that rule via the Specification pattern.

Here's a `User` class which accepts a name and the occupation when initialized:

```php
<?php 
// specificationpattern\app\User.php

namespace SpecificationPattern\App;

class User
{

    public function __construct(public string $name, public string $occupation)
    {
        
    }

}
```

Next, we need to create an interface in which all the specifications will abide to. This accepts a `User` object as argument:

```php 
<?php 

namespace SpecificationPattern\App\UserSpecifications;

use SpecificationPattern\App\User;

interface UserOccupationSpecification
{

    public function matches(User $user);

}
```

Then have it implemented by each of the user occupations supported in your system:

```php
<?php 

namespace SpecificationPattern\App\UserSpecifications;

use SpecificationPattern\App\User;

class IsPokemonTrainer implements UserOccupationSpecification
{

    public function matches(User $user)
    {
        return $user->occupation === 'pokemon_trainer';
    }
} 
```

```php
<?php 

namespace SpecificationPattern\App\UserSpecifications;

use SpecificationPattern\App\User;

class IsPirate implements UserOccupationSpecification
{

    public function matches(User $user)
    {
        return $user->occupation === 'pirate';
    }
} 
```

```php
<?php 

namespace SpecificationPattern\App\UserSpecifications;

use SpecificationPattern\App\User;

class IsMadScientist implements UserOccupationSpecification
{

    public function matches(User $user)
    {
        return $user->occupation === 'mad_scientist';
    }
}
```

Note that the given example is only for checking a specific user attribute. If for example, you added a new business rule which gives specific privileges to a user based on their age then you will have to create another specification for that (eg. `UserAgeSpecification` then it will be implemented by `IsElderly`, `IsChild`, etc.)


At this point, all that's left is to add the client code. You just need to call the `matches` method in the specification and then pass in the `User` object you want to check:

```php 
<?php 
// specification/pattern/SpecificationTest.php

require_once __DIR__ . '/../vendor/autoload.php';

use PHPUnit\Framework\TestCase;

class SpecificationTest extends TestCase
{
    public function test_specs() : void
    {
       $pirateUser = new SpecificationPattern\App\User(name: 'luffy', occupation: 'pirate');
       $pokemonTrainerUser = new SpecificationPattern\App\User(name: 'ash', occupation: 'pokemon_trainer');
       $madScientistUser = new SpecificationPattern\App\User(name: 'rintaro', occupation: 'mad_scientist');

       $pirateSpecification = new SpecificationPattern\App\UserSpecifications\IsPirate;
       $pokemonTrainerSpecification = new SpecificationPattern\App\UserSpecifications\IsPokemonTrainer;
       $madScientistSpecification = new SpecificationPattern\App\UserSpecifications\IsMadScientist;

       $this->assertTrue($pirateSpecification->matches($pirateUser));
       $this->assertTrue($pokemonTrainerSpecification->matches($pokemonTrainerUser));
       $this->assertTrue($madScientistSpecification->matches($madScientistUser));


       $this->assertFalse($pirateSpecification->matches($pokemonTrainerUser));
       $this->assertFalse($pirateSpecification->matches($madScientistUser));

       $this->assertFalse($pokemonTrainerSpecification->matches($pirateUser));
       $this->assertFalse($pokemonTrainerSpecification->matches($madScientistUser));

       $this->assertFalse($madScientistSpecification->matches($pirateUser));
       $this->assertFalse($madScientistSpecification->matches($pokemonTrainerUser));
    }
}
```


To run this. Be sure you have the following on your `composer.json` file then run `composer dump-autoload`:

```
"autoload": {
    "psr-4": {
        "SpecificationPattern\\App\\": "specificationpattern/app"
    }
},
```

Also install phpunit (`composer require phpunit/phpunit`) if you haven't done so already.

Then you can run the tests:

```bash
vendor/bin/phpunit specificationpattern/SpecificationTest.php
```

That's all there is to the specification pattern. It's actually overkill for the examples we used above. You would only want to reach for this pattern if you have some complicated logic required to implement a specific business rule. 90% of the time, you would only have these conditions under your database model classes (especially the one's we used above).

You can find the source code used in this tutorial on this [GitHub repo](https://github.com/anchetaWern/php-design-patterns).