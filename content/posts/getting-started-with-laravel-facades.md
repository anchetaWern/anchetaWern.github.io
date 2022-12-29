---
author: "Wern Ancheta"
title: "Getting Started with Laravel Facades"
date: "2021-08-30"
---


Facades in Laravel allows you to call non-static methods as if they were static methods. This makes it possible for you to call methods in certain classes without the need for creating an object everytime you use them. It basically has the same benefit of using the service container.

Before we look into how we can implement our own Facades with Laravel, let's first take a look at a few examples on how Laravel uses Facades.

The Laravel framework stores its Facades in the `vendor/laravel/frameworks/src/Illuminate/Support/Facades` directory. There you will see familiar one's such as `DB.php`, `View.php`, `Validator.php`, and `Request.php`. If you open any of these files, you will see some code which looks like the following:

```php
class DB extends Facade
{
   
    protected static function getFacadeAccessor()
    {
        return 'db';
    }
}
```

This class extends the `Facade` class (`Facade.php` in the same directory). If you open that up, you'll see a bunch of methods for testing purposes. We're not gonna go over those today. Instead, we'll focus on the `__callStatic` method. What this methos does is call the `getFacadeAccesor()` method in the extending class (e.g the `DB` class) and resolves an instance of the class being returned by that method. In this case, it's the `db` class.

That sounds really confusing even if you tried tracing through the method calls in the base `Facade` class. So let's go through a simpler example first. 

Create an `app/ShowName.php` file and add the following:

```php
<?php
namespace App;

class ShowName {

    public static function __callStatic($method, $arguments)
    {
        dd($method);
    }
}
```

Then in your `routes/web.php` file, add the following route. 

```php
Route::get('show', function(App\ShowName $obj) {

    $obj::someMethodName('cat', 'dog');
});
```

That should then output "somemethodName" or whatever method you called. 

You can also get access to the arguments passed to the method from here:

```php
dd($arguments);
```

That should then show the following output:

```
array:2 [▼
  0 => "cat"
  1 => "dog"
]
```

That's the basic idea behind how Facades work. The Facade has this `__callStatic()` method which then resolves the underlying from the service container. This means that Facades aren't really a replacement to using the service container. It's a supplement to make methods even more easier to call.

Now that you know how Facades work behind the scenes, let's now try to implement one ourselves.

Open the `app/ShowName.php` file from earlier and update the `__callStatic()` method with the following:

```php
public static function __callStatic($method, $arguments)
{
    return (self::resolveFacade('ShowName'))->$method(...$arguments);
}
```

Then add this method:

```php
public static function resolveFacade($name)
{
    return app()[$name];
}
```

This method resolves the specific class from the service container. This will only work if you bind the Facade in your app service provider:

```php
// app/Providers/AppServiceProvider.php

$this->app->bind('ShowName', function () {
    return new ShowNameService;
});
```

The `ShowNameService` contains the following:

```php
<?php
// app/Services/ShowNameService.php

namespace App\Services;

class ShowNameService {

    public function hello($name)
    {
        return 'hello ' . $name;
    }
}
```

Next, open your `config/app.php` file and add a new item to the `aliases` array.

```php
'aliases' => [
    // ...
    'ShowName' => App\ShowName::class, // add this
],
```

Once that's done, you should now be able to call methods from the `ShowNameService` class using the `ShowName` facade.

```php
Route::get('hello', function() {

    return ShowName::hello('cat'); // outputs: "hello cat"
});
```

If you then add a new method on your `ShowNameService` class:

```php
public function hi($name)
{
    return 'hi ' . $name;
}
```

You should still be able to call it through the Facade:

```php
ShowName::hi('cat'); // outputs: "hi cat"
```