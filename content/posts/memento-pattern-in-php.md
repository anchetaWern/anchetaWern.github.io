---
author: "Wern Ancheta"
title: "PHP Design Patterns: Memento Pattern"
date: "2023-05-07"
---

This is the 18th post in a series of articles on how to implement design patterns in PHP.

In this part, we'll see how we can implement the Memento Pattern.

You can use the Memento pattern if you have a need for keeping track of the state of an object. This allows you to do operations like taking a snapshot of its current state and restoring that snapshot at a later point in time.

The Memento pattern has three components to it:

- **Originator** - the object you want to keep track of. For example, a whiteboard app can have a `Whiteboard` class which keeps track of all the changes made to the whiteboard.
- **Memento** - this is where the state you want to keep for a specific object is saved in memory. 
- **Caretaker** - this handles the different operations for managing the state of an object. Things like taking a snapshot (backup), and restoring a snapshot (restore).


### Implementation

Let's say we have a `Garden` class which we want to keep track of. It has three properties: status, plants, and installations. These will be the state which we want to backup. We also have methods for getting and setting values for them, but they're only used for testing. 

```php 
<?php 
// mementopattern/app/Garden.php

namespace MementoPattern\App;

use MementoPattern\App\GardenMemento;
use MementoPattern\App\Memento;

class Garden
{   

    public function __construct(private string $status, private array $plants, private array $installations)
    {

    }


    public function getStatus() : string 
    {
        return $this->status;
    }


    public function getPlants() : array 
    {
        return $this->plants;
    }


    public function getInstallations() : array 
    {
        return $this->installations;
    }


    public function setStatus($status)
    {
        $this->status = $status;
    }


    public function setPlants($plants)
    {
        $this->plants = $plants;
    }


    public function setInstallations($installs)
    {
        $this->installations = $installs; 
    }
}
```

The only requirement is for the originator to implement the `save` and `restore` methods. The `save` method is responsible for initializing an instance of the memento class that's specifically intended for use with this originator (`GardenMemento`). This is where we pass in the unique ID and the state that we want to backup:

```php
// mementopattern/app/Garden.php

public function save($id) : Memento
{   
    $state = [
        'status' => $this->status, 
        'plants' => $this->plants, 
        'installations' => $this->installations,
    ];
    return new GardenMemento($id, json_encode($state));
}
```

On the other hand, we have the `restore` method which is responsible for restoring the state by using the memento that was passed as an argument:

```php
// mementopattern/app/Garden.php

public function restore(Memento $memento) : void
{
    $gardenState = json_decode($memento->getState(), true);

    $this->status = $gardenState['status'];
    $this->plants = $gardenState['plants'];
    $this->installations = $gardenState['installations'];
}
```

Next, we have the memento interface. This is what all memento classes will abide to. In this case, they all have to implement the `getId` and `getState` methods. The former being the one that returns the unique ID for a specific snapshot, while the latter returns the actual snapshot:


```php 
<?php 
// mementopattern/app/Memento.php

namespace MementoPattern\App;

interface Memento 
{

    public function getId();

    public function getState();


}
```

The `GardenMemento` is pretty much a boilerplate class. You initialize it with the ID and the state your backing up, then you create methods for getting those two values:


```php 
<?php 
// mementopattern/app/GardenMemento.php

namespace MementoPattern\App;

use MementoPattern\App\Memento;

class GardenMemento implements Memento 
{

    public function __construct(private string $id, private string $state)
    {
    }


    public function getId() : string
    {
        return $this->id; 
    }

    public function getState() : string
    {
        return $this->state;
    }

}
```

The `CareTaker` is the one that brings it all together. Initializing it requires the originator to be passed to it. This allows the Caretaker to call the `save` and `restore` methods from the Originator. The `backup` method is responsible for calling the `save` method on the Originator. This returns and instance of the `GardenMemento` which allows you to get the ID to be used as the filename and the state you want to backup:

```php
<?php 
// mementopattern/app/CareTaker.php

namespace MementoPattern\App;

use MementoPattern\App\Memento;
use MementoPattern\App\GardenMemento;

class CareTaker
{
  
    public function __construct(private $originator)
    {
        
    }


    public function backup($id) : void
    {
        $memento = $this->originator->save($id);
       
        $fp = fopen($memento->getId() . '.txt', 'w+');
        fwrite($fp, $memento->getState());
        fclose($fp);
    }


}
```

On the other hand, the `restore` method is responsible for reading the contents of the snapshot file and loading it into the GardenMemento. This then allows us to restore the state of the Originator:


```php
// mementopattern/app/CareTaker.php

public function restore($id) : void
{
    $fp = fopen($id . '.txt', 'r');
    $size = filesize($id . '.txt');
    $json = fread($fp, $size);
    fclose($fp);

    $memento = new GardenMemento($id, $json);
    $this->originator->restore($memento);
}
```

To test this out, we initialize a new Garden object and do the following:

1. Initial assertion to make sure that the state is what we expect. 
2. Call the `backup` method to create a new snapshot using the initial state. 
3. Update the state to something else.
4. Create another backup, this time with the updated state.
5. Assert that the object now has the updated state.
6. Restore to the initial state (using the initial snapshot).
7. Assert that the object is now back its initial state.
8. Restore to the updated state.
9. Assert that the object is now using the updated state again.


```php 
<?php 
// mementopattern/MementoTest.php

require_once __DIR__ . '/../vendor/autoload.php';

use PHPUnit\Framework\TestCase;
use MementoPattern\App\Garden;
use MementoPattern\App\Caretaker;

class MementoTest extends TestCase
{
    public function test_memento() : void
    {
        $initialStatus = 'healthy';
        $initialPlants = ['tomato', 'okra', 'basil', 'pepper', 'chives'];
        $initialInstallations = ['uv net'];

        $garden = new Garden($initialStatus, $initialPlants, $initialInstallations);
        $caretaker = new Caretaker($garden); 
        $firstSnapshotId = 'mygarden-' . uniqid();

        // 1
        $this->assertEquals($garden->getStatus(), $initialStatus);
        $this->assertEquals($garden->getPlants(), $initialPlants);
        $this->assertEquals($garden->getInstallations(), $initialInstallations);

        // 2
        $caretaker->backup($firstSnapshotId);

        $updatedStatus = 'unhealthy';
        $updatedPlants = ['spinach', 'kangkong', 'pechay', 'bokchoy', 'arugula'];
        $updatedInstallations = ['trelis', 'drip'];

        // 3
        $garden->setStatus($updatedStatus);
        $garden->setPlants($updatedPlants);
        $garden->setInstallations($updatedInstallations);

        // 4
        $secondSnapshotId = 'mygarden-' . uniqid();
        $caretaker->backup($secondSnapshotId);

        // 5
        $this->assertEquals($garden->getStatus(), $updatedStatus);
        $this->assertEquals($garden->getPlants(), $updatedPlants);
        $this->assertEquals($garden->getInstallations(), $updatedInstallations);

        // 6
        $caretaker->restore($firstSnapshotId);

        // 7
        $this->assertEquals($garden->getStatus(), $initialStatus);
        $this->assertEquals($garden->getPlants(), $initialPlants);
        $this->assertEquals($garden->getInstallations(), $initialInstallations);

        // 8
        $caretaker->restore($secondSnapshotId);

        // 9
        $this->assertEquals($garden->getStatus(), $updatedStatus);
        $this->assertEquals($garden->getPlants(), $updatedPlants);
        $this->assertEquals($garden->getInstallations(), $updatedInstallations);


    }
}
```


To run this. Be sure you have the following on your `composer.json` file then run `composer dump-autoload`:

```json
"autoload": {
    "psr-4": {
        "MementoPattern\\App\\": "mementopattern/app"
    }
}
```

Also install phpunit (`composer require phpunit/phpunit`) if you haven't done so already.

Then you can run the tests:

```bash
vendor/bin/phpunit mementopattern/MementoTest.php
```

That's it for the Memento pattern. You can use this pattern whenever you find yourself needing to take snapshots of an object and performing operations such as undo, redo, list history, etc.

You can find the source code used in this tutorial on this [GitHub repo](https://github.com/anchetaWern/php-design-patterns).