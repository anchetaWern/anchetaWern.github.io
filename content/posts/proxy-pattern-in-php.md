---
author: "Wern Ancheta"
title: "PHP Design Patterns: Proxy Pattern"
date: "2023-05-14"
---

This is the 19th post in a series of articles on how to implement design patterns in PHP.

In this part, we'll see how you can implement the Proxy pattern in PHP. 
The Proxy pattern, as the name suggests is used for creating a proxy for another object. The proxy implements the same interface as the original so it can just be swapped out easily. This way, the proxy can either add or remove functionality from the original. This is for the purpose of consuming less resources or adding extra method calls to other parts of the system (eg. logging or caching).


### Decorator vs Proxy

The Proxy pattern very similar to the [Decorator pattern](/posts/decorator-pattern-in-php/). They both accept an instance of the original object in their constructor and modify its functionality. The difference is in their use case. The Adapter pattern is meant to have multiple slightly differing implementations of the original, while the Proxy pattern usually have just one proxy for the original. This is because it's use case is mostly for 'lazy-instantiating' the original. This way, it doesn't use as much resources. This does it by skipping a few method calls or swapping out one resource from another (eg. using array instead of a database). On the other hand, the Decorator pattern is usually used for changing up the variables so as to modify the response returned by the method its trying to override. So it's basically doing the same thing as opposed to adding or removing functionality.


### Implementation

Let's start by defining the interface that both the original and the proxy will abide to:

```php
<?php 
// proxypattern/app/GardenerInterface.php
namespace ProxyPattern\App;


interface GardenerInterface
{

    public function waterPlants();
   
}
```

Next, define the original. This is a class which executes some methods before watering a plant. The methods can be set using the `setBeforeActions` method. The `$actions` parameter is an array containing the list of methods to be called:


```php
<?php 
// proxypattern/app/Gardener.php
namespace ProxyPattern\App;

use ProxyPattern\App\GardenerInterface;

class Gardener implements GardenerInterface
{
    private $before_actions = [];

    public function waterPlants()
    {
        if (!empty($this->before_actions)) {
            foreach ($this->before_actions as $action) {
                $this->$action();
            }
        }

        if ($this->isSoilDry()) {
            return 'water';
        }

        return 'dont water';
    }


    public function setBeforeActions($actions)
    {
        $this->before_actions = $actions;
    }

    
    public function getBeforeActions()
    {
        return $this->before_actions;
    }


    private function isSoilDry()
    {
        return true;
    }


    private function addOrganicMatter()
    {

    }


    private function addMulch()
    {
        
    }

}
```

Next, we now define the proxy. In this case, it accepts an instance of an original gardener object on its constructor. But with proxy pattern, you can also create an instance of the original from within the class itself. The `waterPlants` method is where we perform some modifications on the way it works. First, we're setting a couple of 'before actions'. The methods that return the method names are defined within the proxy itself (they're all set to `private` so you can easily see which ones), or it can also call upon another class to perform something either before or after calling the original method. `getBeforeActions` is simply a helper method for testing purposes. That way, the test code can easily check which actions were set:

```php
<?php 
// proxypattern/app/GardenProxy.php

namespace ProxyPattern\App;

use ProxyPattern\App\GardenerInterface;
use ProxyPattern\App\Gardener;

class GardenerProxy implements GardenerInterface
{

    private $gardener;

    public function __construct(Gardener $gardener)
    {
        $this->gardener = $gardener;
    }


    public function waterPlants() 
    {
        $before_actions[] = $this->checkPlantType();
        $before_actions[] = $this->checkWeeds();
       
        $this->gardener->setBeforeActions(array_filter($before_actions));
        return $this->gardener->waterPlants();
    }


    public function getBeforeActions()
    {
        return $this->gardener->getBeforeActions();
    }


    private function checkPlantType()
    {
        if ($this->checkSoilPh() === 'alkaline') {
            return 'addOrganicMatter';
        }
    }


    private function checkWeeds()
    {
        if ($this->hasWeeds()) {
            return 'addMulch';
        }
    }


    private function hasWeeds()
    {
        return true;
    }


    private function checkSoilPh()
    {
        return 'alkaline';
    }
    
}
```

Lastly, here's the test code. The main difference between the original and the proxy is that the original doesn't perform any before actions, thus the array is empty. While the proxy performs some actions. So the use case for the proxy pattern in this case is for adding some extra method calls to perform some custom action. That's a perfectly normal use case for the proxy pattern, and you won't actually have to reach for the adapter pattern in this case (although you can) because this particular use case simply doesn't match that of the adapter pattern:


```php
<?php 
// proxypattern/ProxyTest.php

require_once __DIR__ . '/../vendor/autoload.php';

use PHPUnit\Framework\TestCase;

use ProxyPattern\App\Gardener;
use ProxyPattern\App\GardenerProxy;

class ProxyTest extends TestCase
{
    public function test_proxy() : void 
    {
        
        $gardener = new Gardener;
        $gardenerResult = $gardener->waterPlants();
        $this->assertEquals($gardenerResult, 'water');
        $this->assertEquals($gardener->getBeforeActions(), []);

        $gardenerProxy = new GardenerProxy($gardener);
        $gardenerProxyResult = $gardenerProxy->waterPlants();
        $this->assertEquals($gardenerProxyResult, 'water');
        $this->assertEquals($gardenerProxy->getBeforeActions(), ['addOrganicMatter', 'addMulch']);

    }

}
```


To run this. Be sure you have the following on your `composer.json` file then run `composer dump-autoload`:

```json
"autoload": {
    "psr-4": {
        "ProxyPattern\\App\\": "proxypattern/app"
    }
}
```

Also install phpunit (`composer require phpunit/phpunit`) if you haven't done so already.

Then you can run the tests:

```bash
vendor/bin/phpunit proxypattern/ProxyTest.php
```

That's it for the Proxy pattern. In summary, you can reach out for this pattern whenever there's a need to swap out existing functionalities for something lighter. This makes it a really good candidate for implementing mocks for tests. So instead of making a request to an external API and returning the result, you can simply implement a proxy that returns some hard-coded result.

You can find the source code used in this tutorial on this [GitHub repo](https://github.com/anchetaWern/php-design-patterns).