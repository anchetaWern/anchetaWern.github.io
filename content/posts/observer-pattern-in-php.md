---
author: "Wern Ancheta"
title: "PHP Design Patterns: Observer Pattern"
date: "2023-02-26"
---

This is the eight post in a series of articles on how to implement design patterns in PHP.

In this part, we'll see how we can implement the Observer Pattern.

As the name suggests, the Observer Pattern is used for notifying other parts of your code when a specific event happens. It's a way of decoupling your code so that you don't have to explicitly call on methods from different classes whenever something happens.

The Observer Pattern is so useful that PHP has its own class for implementing it.

To implement the Observer Pattern, we have two components:

- Subject - this is the one that performs a specific action.
- Observer - this is the one which listens for when a specific action happens on the subject, thus also triggering its own action.

The relationship between a subject and an observer is one-to-many. So a subject can have many observers.

Let's start by defining the class which will interact with the subject:

```php
<?php 
// observerpattern/app/Client.php

namespace ObserverPattern\App;

use ObserverPattern\App\Store;

class Client 
{

    public function buyFrom(Store $store, string $item)
    {
        $store->sellItem($item);
    }

}
```

Next, we define our subject. As mentioned earlier, PHP has a built-in classes for easily implementing the Observer Pattern:

- [SplObjectStorage](https://www.php.net/manual/en/class.splobjectstorage.php) - instead of a standard array, we use this for storing observers for a specific subject.
- [SplSubject](https://www.php.net/manual/en/class.splsubject.php) - used for implementing the Observer Pattern for dealing with the subject side of things.
- [SplObserver](https://www.php.net/manual/en/class.splobserver.php) - the observer counterpart of `SplSubject`.


All classes which implements `SplSubject` needs to define the following methods:

- `attach` - for attaching observers. This is how observers are linked to a specific subject.
- `detach` - for detaching an observer.
- `notify` - for notifying an observer when an event happens.

```php 
<?php 
// observerpattern/app/Store.php

namespace ObserverPattern\App;

use SplSubject;
use SplObjectStorage;
use SplObserver;

class Store implements SplSubject
{

    private $observers;
    private $item;

    public function __construct()
    {
        $this->observers = new SplObjectStorage(); 
    }


    public function sellItem(string $item)
    {
        $this->item = $item;
        $this->notify();
    }


    public function getItem()
    {
        return $this->item;
    }


    public function attach(SplObserver $observer): void
    {   
        $this->observers->attach($observer);
    }
    

    public function detach(SplObserver $observer): void
    {
        $key = array_search($observer, $this->observers);
        if ($key !== false) {
            $this->observers[$key];
        }
    }

    
    public function notify(): void
    {
        if ($this->observers->count() > 0)
        {
            foreach ($this->observers as $value)
            {
                $value->update($this);
            }
        }
    }
}
```

Observers need to implement the `SplObserver` class. All this requires is for the class to define an `update` method. If you've noticed earlier, we call this method for each observer when the `notify` method is called in the subject class. In the Inventory observer, this will simply get the item that was bought and removes it from the `$items` array:

```php
<?php 
// observerpattern/app/Observers/Inventory.php

namespace ObserverPattern\App\Observers;

use SplObserver;
use SplSubject;

class Inventory implements SplObserver
{
    private $items = [
        'star apple',
        'sugar apple',
        'meyer lemon',
        'mango',
        'java plum',
        'spanish plum',
    ];

    public function update(SplSubject $subject): void
    {
        $key = array_search($subject->getItem(), $this->items);
        if ($key !== false) {
            unset($this->items[$key]);
            $this->items = array_values($this->items);
        }
    }


    public function getItems()
    {
        return $this->items;
    }

}
```

We also have another observer called `Mailer`, all this does is push a new item to the `$sent_mail` array when its `update` method is called:

```php
<?php 
// observerpattern/app/Observer/Mailer.php

namespace ObserverPattern\App\Observers;

use SplObserver;
use SplSubject;

class Mailer implements SplObserver
{
    private $sent_mail = [];

    public function update(SplSubject $subject): void
    {
        $this->sent_mail[] = $subject->getItem();
    }


    public function getSentMail()
    {
        return $this->sent_mail;
    }

}
```

Here's the client code. First, we initialize everything we need then assert the initial value we expect. Then we trigger the action on the subject, in this case it's the `buyFrom` method. This then calls the `update` method in each of the attached observers. We expect that to remove the "mango" from the items in the inventory and add it to the sent mail of the mailer:

```php 
<?php 
// observerpattern/ObserverTest.php

require_once __DIR__ . '/../vendor/autoload.php';

use PHPUnit\Framework\TestCase;

class ObserverTest extends TestCase
{
    public function test_observer() : void
    {
        $client = new ObserverPattern\App\Client;

        $store = new ObserverPattern\App\Store;

        $inventory = new ObserverPattern\App\Observers\Inventory;
        $mailer = new ObserverPattern\App\Observers\Mailer;

        $this->assertEquals($inventory->getItems(), [
            'star apple',
            'sugar apple',
            'meyer lemon',
            'mango',
            'java plum',
            'spanish plum',
        ]);

        $this->assertEquals($mailer->getSentMail(), []);

        $store->attach($inventory);
        $store->attach($mailer);
        $client->buyFrom($store, 'mango');

        $this->assertEquals($inventory->getItems(), [
            'star apple',
            'sugar apple',
            'meyer lemon',
            'java plum',
            'spanish plum',
        ]);

        $this->assertEquals($mailer->getSentMail(), ['mango']);
        
    }
}
```

To run this. Be sure you have the following on your `composer.json` file then run `composer dump-autoload`:

```json
"autoload": {
    "psr-4": {
        "ObserverPattern\\App\\": "observerpattern/app"
    }
},
```

Also install phpunit (`composer require phpunit/phpunit`) if you haven't done so already.

Then you can run the tests:

```bash
vendor/bin/phpunit observerpattern/ObserverTest.php
```

That's it for the Observer Pattern. You basically have to create a one-to-many relationship between between a subject and its observers. So that when something happens to the subject, all of its observers can immediately be notified so they could do their own thing.

You can find the source code used in this tutorial on this [GitHub repo](https://github.com/anchetaWern/php-design-patterns).