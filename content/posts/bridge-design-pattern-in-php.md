---
author: "Wern Ancheta"
title: "PHP Design Patterns: Bridge Pattern"
date: "2023-03-26"
---

This is the twelfth post in a series of articles that will walk you through how to implement design patterns in PHP.

In this part, we're going to take a look at how you can implement the Bridge Pattern.

This pattern kinda looks similar to the [Strategy Pattern](/posts/strategy-pattern-in-php/), but on a wider scale.
The basic idea is to separate the implementations from the different strategies so that each strategy can use those implementations interchangeably. 

Like always, it's best to just dive right into the code so we can see how this works.

Say we have a camera app which emulates different kinds of camera: Mirrorless, DSLR, etc. We want it to emulate every setting that cameras have. Things like the lens type: wide angle, telephoto, macro, etc. And also the priority: shutter and aperture.

We want to be able to mix and match these in different ways. For example, we can have a mirrorless camera which has telephoto lens and set to shutter priority. But we can also have a mirrorless camera with macro lens that uses aperture priority.

If you try to implement it manually, you'll probably have something like this:

```php
<?php 

class MirrorlessCamera 
{

    public function __construct(private string $lens_type, private string $lens_priority)
    {

    }

    public function takePicture() : array
    {
        return [
            'camera' => 'mirrorless',
            'priority' => $this->getPriority(),
            'type' => $this->getType(),
        ];
    }

    private function getPriority()
    {
        if ($this->lens_priority === 'shutter') {
            // some complicated logic
        } else if ($this->lens_priority === 'aperture') {
            // some complicated logic
        }
    }

    private function getType()
    {
        if ($this->lens_type === 'macros') {
            // some complicated logic
        } else if ($this->lens_type === 'telephoto') {
            // some complicated logic
        } else if ($this->lens_type === 'wide_angle') {
            // some complicated logic
        }

    }
}

class DSLRCamera 
{

    public function __construct(private string $lens_type, private string $lens_priority)
    {

    }

    public function takePicture() : array
    {
        return [
            'camera' => 'dslr',
            'priority' => $this->getPriority(),
            'type' => $this->getType(),
        ];
    }

    private function getPriority()
    {
        // some complicated logic
    }

    private function getType()
    {
        // some complicated logic
    }
}
```

Then you use it like so:

```php
<?php

$mirrorless = new MirrorlessCamera('macro', 'aperture');
$mirrorless->takePicture();
```

Yes, you can probably put the repeated code in an abstract class and have both classes inherit from it like so:

```php 
<?php

abstract class Camera
{

    public function __construct(protected string $lens_type, protected string $lens_priority)
    {

    }

    abstract protected function getPriority();

    abstract protected function getType();


    public function takePicture() : array
    {
        return [
            'camera' => get_class($this),
            'priority' => $this->getPriority(),
            'type' => $this->getType(),
        ];
    }
}


class MirrorlessCamera extends Camera
{
    protected function getPriority()
    {
        // some complicated logic
    }

    protected function getType()
    {
        // some complicated logic
    }
}


class DSLRCamera extends Camera
{
    protected function getPriority()
    {
        // some complicated logic
    }

    protected function getType()
    {
        // some complicated logic
    }
}
```

The above code is fine. But what if you want to reuse the logic across different camera types? For that, you can probably use traits:

```php 
<?php

trait ShutterPriority
{

    public function getShutterPriority()
    {
        // some complicated logic
    }

}

trait AperturePriority
{

    public function getAperturePriority()
    {
        // some complicated logic
    }

}


class DSLRCamera extends Camera
{

    use ShutterPriority, AperturePriority;

    protected function getPriority()
    {
        if ($this->lens_priority === 'shutter') {
            return $this->getShutterPriority();
        } else if ($this->lens_priority === 'aperture') {
            return $this->getAperturePriority();
        }
    }

}
```

At this point, it looks fine. You can even try to map the class variables to actual methods so you can get rid of the conditional code.
But what if you have different implementations of the shutter and aperture priority on different types of cameras? Not to mention different implementations of lens type. By then you'll have to repeat some part of the logic across different camera types.

This is where the Bridge design pattern comes in. It allows you to combine different implementations in different ways.

To use it, you first need to create an abstract class which the different camera types will inherit from. This contains the `setLensType` and `setLensPriority` methods. Think of these as the different "strategies" for implementing the logic for getting the lens type and lens priority. We're using the `LensType` and `LensPriority` interface so that it's not bound to a single type:

```php 
<?php 
// bridgepattern/app/Camera.php

namespace BridgePattern\App;

use BridgePattern\App\Implementations\LensType\LensType;
use BridgePattern\App\Implementations\LensPriority\LensPriority;

abstract class Camera
{
    public function setLensType(LensType $type) : void
    {
        $this->lens_type = $type;
    }

    public function setLensPriority(LensPriority $priority) : void
    {
        $this->lens_priority = $priority;
    }

    abstract public function takePicture() : array;

}
```

Next, we now create the classes for the different camera types. Each one has to add their own implementation of the `takePicture` method. This is where we use the strategies that were passed via the `setLensType` and `setLensPriority` earlier:

```php 
<?php 
// bridgepattern/app/DSLRCamera.php

namespace BridgePattern\App;

use BridgePattern\App\Camera;

class DSLRCamera extends Camera {

    public function takePicture() : array
    {
        return [
            'camera' => 'dslr',
            'priority' => $this->lens_priority->getPriority(),
            'type' => $this->lens_type->getType(),
        ];
    }

}
```


```php 
<?php 
// bridgepattern/app/MirrorLessCamera.php

namespace BridgePattern\App;

use BridgePattern\App\Camera;

class MirrorLessCamera extends Camera {

    public function takePicture() : array
    {
        return [
            'camera' => 'mirrorless',
            'priority' => $this->lens_priority->getPriority(),
            'type' => $this->lens_type->getType(),
        ];
    }

}
```

Here's the `LensType` interface and its implementations:


```php 
<?php 
// bridgepattern/app/Implementations/LensType/LensType.php

namespace BridgePattern\App\Implementations\LensType;

interface LensType 
{
    public function getType() : string;

}
```


```php 
<?php 
// bridgepattern/app/Implementations/LensType/Macro.php

namespace BridgePattern\App\Implementations\LensType;

use BridgePattern\App\Implementations\LensType\LensType;

class Macro implements LensType 
{
    public function getType() : string
    {
        return 'macro';
    }

}
```


```php
<?php 
// bridgepattern/app/Implementations/LensType/Telephoto.php

namespace BridgePattern\App\Implementations\LensType;

use BridgePattern\App\Implementations\LensType\LensType;

class Telephoto implements LensType 
{
    public function getType() : string
    {
        return 'telephoto';
    }

}
```


```php 
<?php 
// bridgepattern/app/Implementations/LensType/WideAngle.php

namespace BridgePattern\App\Implementations\LensType;

use BridgePattern\App\Implementations\LensType\LensType;

class WideAngle implements LensType 
{
    public function getType() : string
    {
        return 'wide_angle';
    }

}
```

Here's the `LensPriority` interface and its implementations:


```php 
<?php 
// bridgepattern/app/Implementations/LensPriority/LensPriority.php

namespace BridgePattern\App\Implementations\LensPriority;

interface LensPriority 
{

    public function getPriority() : string;

}
```


```php 
<?php 
// bridgepattern/app/Implementations/LensPriority/AperturePriority.php

namespace BridgePattern\App\Implementations\LensPriority;

use BridgePattern\App\Implementations\LensPriority\LensPriority;

class AperturePriority implements LensPriority
{

    public function getPriority() : string 
    {
        return 'aperture';
    }

}
```


```php 
<?php 
// bridgepattern/app/Implementations/LensPriority/ShutterPriority.php

namespace BridgePattern\App\Implementations\LensPriority;

use BridgePattern\App\Implementations\LensPriority\LensPriority;

class ShutterPriority implements LensPriority
{

    public function getPriority() : string 
    {
        return 'shutter';
    }

}
```

At this point, we can now use the Bridge pattern like so:


```php 
<?php 
// bridgepattern/BridgeTest.php

require_once __DIR__ . '/../vendor/autoload.php';

use PHPUnit\Framework\TestCase;

class BridgeTest extends TestCase
{
    public function test_bridge() : void
    {
        $dslr = new BridgePattern\App\DSLRCamera();
        $dslr->setLensType(new BridgePattern\App\Implementations\LensType\Macro);
        $dslr->setLensPriority(new BridgePattern\App\Implementations\LensPriority\AperturePriority);

        $this->assertEquals([
            'camera' => 'dslr',
            'type' => 'macro',
            'priority' => 'aperture',
        ], $dslr->takePicture());


        $mirrorless = new BridgePattern\App\MirrorlessCamera();
        $mirrorless->setLensType(new BridgePattern\App\Implementations\LensType\Telephoto);
        $mirrorless->setLensPriority(new BridgePattern\App\Implementations\LensPriority\ShutterPriority);
        
        $this->assertEquals([
            'camera' => 'mirrorless',
            'type' => 'telephoto',
            'priority' => 'shutter',
        ], $mirrorless->takePicture());
       
    }
}
```

As you can see from the above code, we are able to combine different implementations of the lens type and lens priority strategies together. We're also able to avoid conditionals in our code. Best of all, when a new camera type is added, we can simply create a separate class for it and then have it delegate the heavy-lifting to whatever implementation strategy is passed to it:

```php
<?php 

namespace BridgePattern\App;

use BridgePattern\App\Camera;

class PhoneCamera extends Camera {

    public function takePicture() : array
    {
        return [
            'camera' => 'phone',
            'priority' => $this->lens_priority->getPriority(),
            'type' => $this->lens_type->getType(),
        ];
    }

}
```

We can even add new lens types:

```php
<?php 

namespace BridgePattern\App\Implementations\LensType;

use BridgePattern\App\Implementations\LensType\LensType;

class FishEye implements LensType 
{
    public function getType() : string
    {
        return 'fish_eye';
    }

}
```

Or lens priorities:

```php
<?php 
namespace BridgePattern\App\Implementations\LensPriority;

use BridgePattern\App\Implementations\LensPriority\LensPriority;

class ManualPriority implements LensPriority
{

    public function getPriority() : string 
    {
        return 'manual';
    }

}
```

And you don't have to change a thing on any of the existing camera types. You can simply call the new strategies from your client code and you're good to go:

```php
<?php
$dslr = new BridgePattern\App\DSLRCamera();
$dslr->setLensType(new BridgePattern\App\Implementations\LensType\FishEye);
$dslr->setLensPriority(new BridgePattern\App\Implementations\LensPriority\ManualPriority);
$dslr->takePicture();
```


To run this. Be sure you have the following on your `composer.json` file then run `composer dump-autoload`:

```
"autoload": {
    "psr-4": {
        "BridgePattern\\App\\": "bridgepattern/app/"
    }
},
```

Also install phpunit (`composer require phpunit/phpunit`) if you haven't done so already.

Then you can run the tests:

```bash
vendor/bin/phpunit bridgepattern/BridgePattern.php
```

That's all there is to it to the Bridge Pattern. It's the next level to the Strategy pattern when you find yourself working with many class hierarchies with different implementations to the things you want them to accomplish.

You can find the source code used in this tutorial on this [GitHub repo](https://github.com/anchetaWern/php-design-patterns).