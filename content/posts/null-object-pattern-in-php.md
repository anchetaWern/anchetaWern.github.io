---
author: "Wern Ancheta"
title: "PHP Design Patterns: Null Object Pattern"
date: "2023-03-12"
---

This is the tenth post in a series of articles that will walk you through how to implement design patterns in PHP.

In this part, we're going to take a look at how you can implement the Null Object Pattern. 

This pattern is used for getting rid of null checks. So instead of conditionals all over your code, you create a separate class which will be used for instances wherein the things you're checking simply doesn't exist. You'll then have it return some sort of placeholder content just to keep things from breaking.

The most common example for implementing this pattern is for checking user subscription. Here we have an abstract class for a `User`. This requires a subscription string to be passed when you initialize a new object. We also have a `getSub` method which will be used for the tests later:

```php
<?php 
// nullobjectpattern/app/User.php

namespace NullObjectPattern\App;

abstract class User
{   
    protected $sub;

    public function __construct(string $sub = null)
    {
        $this->sub = $sub;
    }

    public abstract function getSub() : string;
    
} 
```

Here's the class which only considers the "happy path" (eg. a subscription is always passed in the constructor):

```php 
<?php 
// nullobjectpattern/app/SubscribedUser.php

namespace NullObjectPattern\App;

class SubscribedUser extends User
{
    public function getSub(): string
    {
        return $this->sub;
    }
}
```

Of course, this works as expected when you run the following code:

```php
<?php 
// nullobjectpattern/NullObjectTest.php

require_once __DIR__ . '/../vendor/autoload.php';

use PHPUnit\Framework\TestCase;

class NullObjectTest extends TestCase
{
    public function test_null_object() : void
    {
        
        $subscribed_user = new NullObjectPattern\App\SubscribedUser('gold_subscription');
        $this->assertEquals($subscribed_user->getSub(), 'gold_subscription');

    } 
}
```

But for scenarios wherein the user doesn't have a subscription, nothing is passed when initializing an object:

```php 
<?php 
// nullobjectpattern/NullObjectTest.php

require_once __DIR__ . '/../vendor/autoload.php';

use PHPUnit\Framework\TestCase;

class NullObjectTest extends TestCase
{
    public function test_null_object() : void
    {
        
        $subscribed_user = new NullObjectPattern\App\SubscribedUser();
        $this->assertEquals($subscribed_user->getSub(), '');

    } 
}
```

Thus you will get the following error:

```
TypeError: NullObjectPattern\App\SubscribedUser::getSub(): Return value must be of type string, null returned
```

Or this if you didn't specify the return type as string:

```
Failed asserting that '' is identical to null.
```

To solve this, you would often reach for conditionals like so:

```php 
<?php 
// nullobjectpattern/app/SubscribedUser.php

class SubscribedUser extends User
{
    public function getSub()
    {
        if ($this->sub) {
            return $this->sub;
        }
        return '';
    }
}
```

But this situation is what we're trying to avoid by using the Null Object Pattern. Conditionals aren't evil, but they often add to the cognitive load when reading code. It's better if we can just have a fallback for the type of situations wherein the thing we're expecting doesn't exist. Here's how you can implement the Null Object Pattern in this situation:

```php 
<?php 
// nullobjectpattern/app/UnsubscribedUser.php

namespace NullObjectPattern\App;

class UnSubscribedUser extends User
{
    public function getSub(): string
    {
        return '';
    }
}
```

Now you can simply use the `UnsubscribedUser` class when a user isn't subscribed:

```php 
public function test_null_object() : void
{
    $unsubscribed_user = new NullObjectPattern\App\UnSubscribedUser();
    $this->assertEquals($unsubscribed_user->getSub(), '');
}
```


To run this. Be sure you have the following on your `composer.json` file then run `composer dump-autoload`:

```
"autoload": {
    "psr-4": {
        "NullObjectPattern\\App\\": "nullobjectpattern/app/"
    }
},
```

Also install phpunit (`composer require phpunit/phpunit`) if you haven't done so already.

Then you can run the tests:

```bash
vendor/bin/phpunit nullobjectpattern/NullObjectPattern.php
```

You can find the source code used in this tutorial on this [GitHub repo](https://github.com/anchetaWern/php-design-patterns).