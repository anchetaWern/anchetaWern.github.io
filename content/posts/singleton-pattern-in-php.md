---
author: "Wern Ancheta"
title: "PHP Design Patterns: Singleton Pattern"
date: "2023-04-02"
---

This is the 13th post in a series of articles that will walk you through how to implement design patterns in PHP.

In this part, we're going to take a look at how you can implement the Singleton Pattern.

The Singleton Pattern is a creational design pattern for ensuring that you're only ever working with a single instance of a class. This makes it very predictable in terms of the values it handles. Accessing it from different parts of your code will yield the same instance all the time. 

You can use the Singleton Pattern anywhere where you only want to be working with a single class instance at any given time anywhere in your code base. A common example if a `Config` class which allows you to set and get different config properties for your entire application.

To prevent instantiation, you need to make the `constructor` into `private`:

```php 
<?php 
// singletonpattern/app/Config.php

namespace SingletonPattern\App;

class Config 
{
    private static Config $instance;

    private array $props = []; // for storing config properties and their values

    private function __construct() 
    {

    }

}
```

This should prevent anyone from doing this:

```php
new SingletonPattern\App\Config();
```

And it would return an error that looks like this:

```
Error: Call to private SingletonPattern\App\Config::__construct() from scope SingletonTest
```

To allow the client coder to use this class, we need to add a method for getting an instance of this class. Here, we first check if an instance already exists. If not, then we simply insantiate it from within the class and assign it to a local variable called `$intance`. Since an instance already exists, this variable will then be the one to be returned for subsequent times that the `getInstance()` method is called:


```php
public static function getInstance(): Config
{
    if (!isset(self::$instance)) {
        self::$instance = new self;
    }

    return self::$instance;
} 
```

Next, add the methods for setting and getting properties. These can just be normal methods as they don't have to be `static`:

```php 
public function setProp(string $name, string $value): void
{
    $this->props[$name] = $value; 
}


public function getProp(string $name): string
{
    return $this->props[$name];
}
```

At this point, we can now test out if our singleton class really works. First, we check that it can't be instantiated:

```php
<?php 
// singletonpattern/app/SingletonTest.php

require_once __DIR__ . '/../vendor/autoload.php';

use PHPUnit\Framework\TestCase;

class SingletonTest extends TestCase
{
    public function test_singleton_cant_be_initialized()
    {
        
        $this->expectErrorMessage('Call to private SingletonPattern\App\Config::__construct() from scope SingletonTest');
        $config = new SingletonPattern\App\Config;

    }

}
```

We can also check that the functionality it intended to implement really works:

```php
public function test_singleton_works() : void
{
    $config = SingletonPattern\App\Config::getInstance();
    $config->setProp('name', 'goku');

    $this->assertEquals($config->getProp('name'), 'goku');
    
} 
```

And that it can't just be unset:

```php 
public function test_singleton_cant_be_unset() : void 
{
    $config = SingletonPattern\App\Config::getInstance();
    $config->setProp('name', 'vegeta');
    unset($config);

    $config2 = SingletonPattern\App\Config::getInstance();
    $this->assertEquals($config2->getProp('name'), 'vegeta');
}
```

Finally, test if the values in the `props` array are indeed overriden even if you store the reference on a different variable:


```php
public function test_singleton_is_really_single() : void 
{
    $config = SingletonPattern\App\Config::getInstance();
    $config->setProp('wife', 'bulma');

    $config2 = SingletonPattern\App\Config::getInstance();
    $config2->setProp('wife', 'chi-chi');

    $this->assertNotEquals($config->getProp('wife'), 'bulma');
    $this->assertEquals($config->getProp('wife'), 'chi-chi');
} 
```


To run this. Be sure to add the following on your `composer.json` file then run `composer dump-autoload`:

```
"autoload": {
    "psr-4": {
        "SingletonPattern\\App\\": "singletonpattern/app/"
    }
},
```

Also install phpunit (`composer require phpunit/phpunit`) if you haven't done so already.

You can then run it by executing the following:

```bash
vendor/bin/phpunit singletonpattern/app/SingletonTest.php
```

That's really all there is to the Singleton Pattern. It's a way to ensure that a class has only one globally available instance. A common usage for this pattern is for helper classes or storing config. Basically anything that doesn't need to work with different instances can be a singleton.

You can find the source code used in this tutorial on this [GitHub repo](https://github.com/anchetaWern/php-design-patterns).

