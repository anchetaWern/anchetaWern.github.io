---
author: "Wern Ancheta"
title: "PHP Design Patterns: Adapter Pattern"
date: "2023-01-22"
---

This is the third post in a series of articles that will walk you through how to implement design patterns in PHP.

In this part, we'll look at how you can implement the adapter pattern. Just like the name suggests, the Adapter pattern is simply a way to translate a third-party code to a common interface that you can use it within your existing codebase.

For example, we have a `Client` class which uses a filesystem library to open a file:

```php 
<?php 

// adapterpattern/app/Client.php

namespace AdapterPattern\App;

class Client
{
    public function readFile(FilesystemInterface $filesystem)
    {
        return $filesystem->open();
    }
}
```

Here's what the `FilesystemInterface` looks like:


```php 
<?php 

// adapterpattern/app/FilesystemInterface.php

namespace AdapterPattern\App; 

interface FilesystemInterface 
{
    public function open();
    
}
```

This is implemented by a class which allows the client to open a file in the local filesystem:

```php
<?php 
// adapterpattern/app/LocalFilesystem.php

namespace AdapterPattern\App;

class LocalFilesystem implements FilesystemInterface
{

    public function open()
    {
        return 'opening local file';
    }

}
```

Here's how we go about using it:

```php
<?php 
// adapterpattern/AdapterTest.php

require_once __DIR__ . '/../vendor/autoload.php';

use PHPUnit\Framework\TestCase;

class AdapterTest extends TestCase
{
    public function test_filesystem_adapter() : void
    {   
        $client = new AdapterPattern\App\Client();

        $local_filesystem = new AdapterPattern\App\LocalFilesystem();
        $this->assertEquals($client->readFile($local_filesystem), 'opening local file');
       
    }
} 
```

Of course this will work as expected since we own the `LocalFilesystem` class and we can easily make modifications to it as needed. But what if we decided to use a third-party library later on? 

In the below example, we want to use a third-party library which talks to the Dropbox API to open a file in a user's Dropbox:

```php 
<?php
public function test_filesystem_adapter() : void
{   
    $client = new AdapterPattern\App\Client();

    $dropbox_filesystem = new AdapterPattern\App\Dropbox();
    $this->assertEquals($client->readFile($dropbox_filesystem), 'opening dropbox file');
}
```

Here's the contents of the `Dropbox` class:

```php
<?php 
// adapterpattern/app/Dropbox.php

namespace AdapterPattern\App;

class Dropbox implements CloudFilesystemInterface
{
    public function openFile()
    {
        return 'opening dropbox file';
    }
}
```

And the interface it adheres to:

```php
<?php

// adapterpattern/app/CloudFilesystemInterface.php

namespace AdapterPattern\App;

interface CloudFilesystemInterface 
{
    public function openFile();
}
```

Obviously, this wouldn't work and you'll get the following error:

```
TypeError: AdapterPattern\App\Client::readFile(): Argument #1 ($filesystem) must be of type AdapterPattern\App\FilesystemInterface, AdapterPattern\App\Dropbox given, called in /design-patterns/adapterpattern/AdapterTest.php on line 18
```

This means that the `Client` class was expecting something that implements the `FilesystemInterface` but instead it got the `Dropbox` class. Moreover, the `open` method is used by the `FilesystemInterface` while the `Dropbox` class has the `openFile` method. So even if you remove the constraint in the `Client` class like so:

```php
<?php
// adapterpattern/app/Client.php

class Client
{
    public function readFile($filesystem) // removed FilesystemInterface type
    {
        return $filesystem->open();
    }
}
```

You would still get an error since the method you're trying to call is undefined:

```
Error: Call to undefined method AdapterPattern\App\Dropbox::open()

/design-patterns/adapterpattern/app/Client.php:9
```

This is where the Adapter pattern comes in. You can implement it by creating a class which would use the third-party library you're trying to use. The key thing here is that this class has to implement the interface which you are already using. In this case it's the `FilesystemInterface`. From there, all you have to do is pass an instance of the third-party library you're trying to use in the constructor then implement all the methods that the interface has specified. In this case, we simply translate the `openFile` method from the `Dropbox` library to the `open` method of the `FilesystemInterface`:

```php
<?php 
// adapterpattern/app/FilesystemAdapter.php

namespace AdapterPattern\App;

class FilesystemAdapter implements FilesystemInterface 
{

    public function __construct(private CloudFilesystemInterface $filesystem)
    {

    }

    public function open()
    {
        return $this->filesystem->openFile();
    }

}
```

At this point, all you need to do is wrap the instance of the `Dropbox` class with the `FilesystemAdapter`:

```php
public function test_filesystem_adapter() : void
{   
    $client = new AdapterPattern\App\Client();

    $local_filesystem = new AdapterPattern\App\LocalFilesystem();
    $this->assertEquals($client->readFile($local_filesystem), 'opening local file');

    // this wont work since dropbox doesnt adhere to the same interface
    /*
    $dropbox_filesystem = new AdapterPattern\App\Dropbox();
    $this->assertEquals($client->readFile($dropbox_filesystem), 'opening dropbox file');
    */
    
    // so we use an adapter which implements the interface we want to adhere to
    $dropbox_filesystem = new AdapterPattern\App\Dropbox();
    $this->assertEquals($client->readFile(new AdapterPattern\App\FilesystemAdapter($dropbox_filesystem)), 'opening dropbox file');
}
```

That's all there is to the Adapter pattern. It's simply a way to create an adapter for third-party code so that you can plug it in to your existing code.


Be sure to add the following on your `composer.json` file then run `composer dump-autoload`:

```
"autoload": {
    "psr-4": {
        "AdapterPattern\\App\\": "adapterpattern/app/"
    }
},
```

Also install phpunit (`composer require phpunit/phpunit`) if you haven't done so already.

You can then run it by executing the following:

```bash
vendor/bin/phpunit adapterpattern/app/AdapterTest.php
```

That's really all there is to the Facade pattern. It's a way to simplify the usage of an otherwise complex. 

You can find the source code used in this tutorial on this [GitHub repo](https://github.com/anchetaWern/php-design-patterns).

