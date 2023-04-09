---
author: "Wern Ancheta"
title: "PHP Design Patterns: Command Pattern"
date: "2023-04-09"
---

This is the 14th post in a series of articles that will walk you through how to implement design patterns in PHP.

In this part, we're going to take a look at how you can implement the Command Pattern.

This pattern is a behavioral pattern that allows you to reduce coupling between client and business logic. That way you don't have to make changes to the client code everytime you introduce a new business logic.

It has four parts to it:

- **Client**: responsible for instantiating the command object.
- **Invoker**: deploys the object.
- **Receiver**: this is where the command delegates its operations.
- **Command**: this knows how to call the necessary methods from the receiver.

### Receiver

Let's start with the Receiver first (aka the business logic). A good rule of thumb is to always create an interface which the different entities in your program will abide to, especially if they're related. This allows you to specify the interface for type hinting later on.

```php 
<?php 
// commandpattern/app/Receiver/ButtonActionContext.php

namespace CommandPattern\App\Receiver;

interface ButtonActionContext 
{

}
```

Now we're ready to add our business logic. In this example, we have a class which works with some form of database. A `Book` has a diferrent code for saving a book record in the database since it has a different set of fields. That's what this class is trying to implement. To keep things simple, we're just returning a string describing what it does:

```php 
<?php 
// commandpattern/app/Receiver/Book.php

namespace CommandPattern\App\Receiver;

use CommandPattern\App\Receiver\ButtonActionContext;

class Book implements ButtonActionContext 
{

    public function save()
    {
        return 'save book';
    }


    public function update()
    {
        return 'update book';
    }

}
```

Also add a `Video` class:


```php 
<?php 
// commandpattern/app/Receiver/Video.php

namespace CommandPattern\App\Receiver;

use CommandPattern\App\Receiver\ButtonActionContext;

class Video implements ButtonActionContext 
{

    public function save()
    {
        return 'save video';
    }


    public function update()
    {
        return 'update video';
    }
    
}
```

### Command

The Command is responsible for calling the necessary methods from the receiver. Each method in the receiver maps to a specific command. But it should be able to cater for all the related entities in your code. This means, you don't have to create a command specific to each entitiy and method (eg. SaveVideoCommand, UpdateVideoCommand, SaveBookCommand, UpdateBookCommand). Generalize it based on the actions that are taken (save and update). But first, let's create an abstract class which the commands will inherit from. They're specifically going to inherit the constructor. We want all commands to accept an instance of a `ButtonActionContext`. This is the interface we created for grouping the receivers earlier. All commands also needs to implement an `execute` method. This is the method which calls on the corresponding method we need from the receiver:

```php 
<?php 
// commandpattern/app/Command/ButtonCommand.php

namespace CommandPattern\App\Command;

use CommandPattern\App\Receiver\ButtonActionContext;


abstract class ButtonCommand {

    public function __construct(protected ButtonActionContext $context)
    {

    }

    abstract public function execute() : string;

}
```

Once that's done, we can now implement the different commands. First we have the `SaveCommand` which calls on the `save` method from whatever context is passed to it: 


```php 
<?php 
// commandpattern/app/Command/SaveCommand.php

namespace CommandPattern\App\Command;

use CommandPattern\App\Command\ButtonCommand;

class SaveCommand extends ButtonCommand
{

    public function execute() : string
    {
        return $this->context->save();
    }
}
```

Do the same for the `UpdateCommand`:

```php 
<?php 
// commandpattern/app/Command/UpdateCommand.php

namespace CommandPattern\App\Command;

use CommandPattern\App\Command\ButtonCommand;

class UpdateCommand extends ButtonCommand
{

    public function execute() : string
    {
        return $this->context->update();
    }
}
```


### Invoker

The Invoker is where most of the magic happens. It's the secret sauce that allows us to use any combination of Commands and Receivers together. You supply it with a receiver and the action you want to perform upon instantiation and it will automagically load up the Command class which needs to be called and pass it the correct receiver based on the `$action` you specify: 

```php 
<?php 
// commandpattern/app/Invoker.php

namespace CommandPattern\App; 

use CommandPattern\App\Receiver\ButtonActionContext;

class Invoker 
{
    
    public function __construct(private ButtonActionContext $context, private string $action)
    {

    }


    public function run()
    {
        $class = __NAMESPACE__ . "\\command\\" . ucfirst(strtolower($this->action)) . "Command";

        if (class_exists($class)) {
            $command = new $class($this->context);
        } else {
            throw new \Exception('Command not found');
        }
        
        return $command->execute();
    }

}
```

### Client

Lastly, our test class will serve as our Client. This is where we call on the Invoker to do our bidding for us. Note that sometimes, the invoker and the client are merged into one. In this case, it just makes sense for us to separate it so our tests looks cleaner:


```php 
<?php 
// commandpattern/CommandTest.php

require_once __DIR__ . '/../vendor/autoload.php';

use PHPUnit\Framework\TestCase;

use CommandPattern\App\Receiver\Video;
use CommandPattern\App\Receiver\Book;

class CommandTest extends TestCase
{

    public function test_command() : void
    {
        $save_video_invoker = new CommandPattern\App\Invoker(new Video, 'save');
        $this->assertEquals($save_video_invoker->run(), 'save video');

        $update_video_invoker = new CommandPattern\App\Invoker(new Video, 'update');
        $this->assertEquals($update_video_invoker->run(), 'update video');

        $save_book_invoker = new CommandPattern\App\Invoker(new Book, 'save');
        $this->assertEquals($save_book_invoker->run(), 'save book');

        $update_book_invoker = new CommandPattern\App\Invoker(new Book, 'update');
        $this->assertEquals($update_book_invoker->run(), 'update book');
    }
}
```


To run this. Be sure to add the following on your `composer.json` file then run `composer dump-autoload`:

```
"autoload": {
    "psr-4": {
        "CommandPattern\\App\\": "commandpattern/app/"
    }
},
```

Also install phpunit (`composer require phpunit/phpunit`) if you haven't done so already.

You can then run it by executing the following:

```bash
vendor/bin/phpunit commandpattern/app/CommandTest.php
```

### Adding More Commands

Now lets say you want to add another command. For deleting stuff this time:


```php
<?php 

namespace CommandPattern\App\Command;

use CommandPattern\App\Command\ButtonCommand;

class DeleteCommand extends ButtonCommand
{

    public function execute() : string
    {
        return $this->context->delete();
    }
}
```

Of course, this also needs to be implemented on the Receiver:

```php 
<?php 

namespace CommandPattern\App\Receiver;

use CommandPattern\App\Receiver\ButtonActionContext;

class Book implements ButtonActionContext 
{

    // ...

    public function delete()
    {
        return 'delete book';
    }

}
```

But other than that, we don't really need to do anything else. We simply call it where we need to call it by using the Invoker:

```php
$delete_book_invoker = new CommandPattern\App\Invoker(new Book, 'delete');
$this->assertEquals($delete_book_invoker->run(), 'delete book');
```

That's it for the Command Pattern. In the real-world, it is often used for implementing queues. This way, instead of executing immediately, you can store all the data required by the invoker to run then execute it at a later time.

You can find the source code used in this tutorial on this [GitHub repo](https://github.com/anchetaWern/php-design-patterns).

