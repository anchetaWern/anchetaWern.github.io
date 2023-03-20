---
author: "Wern Ancheta"
title: "PHP Design Patterns: Composite Pattern"
date: "2023-03-19"
---

This is the eleventh post in a series of articles that will walk you through how to implement design patterns in PHP.

In this part, we're going to take a look at how you can implement the Composite Pattern.

This pattern is used for treating a group of objects the same way you treat any single instance of the object. This is most commonly used for building UI, specifically tree structures which has varying levels of nesting. It's also used for logging, so you can either log a single lines or multiple lines based on the current type of logger.

The basic idea is to have both the group and the individual object abide to the same interface or abstract class so that they're treated the same way:

```php
<?php 
// compositepattern/app/MenuInterface.php

namespace CompositePattern\App;

interface MenuInterface 
{
    public function render() : array;
}
```

Now create another class which will implement it. Here we're creating a Menu class which allows the user to add another instance of a Menu to it (`add` method) and also allows for rendering all the menus that were added to it (the `render` method). Usually you will output something here, but we're constructing an array instead so that we can easily test it out:

```php
<?php 
// compositepattern/app/Menu.php

namespace CompositePattern\App;

class Menu implements MenuInterface
{

    private array $menu_items;
 
    public function __construct(public string $title)
    {
        $this->menu_items = [];
    }   
    

    public function add(MenuInterface $link)
    {
        $this->menu_items[] = $link;
    }

    
    public function render() : array
    {
        $menu = [
            'title' => $this->title,
            'items' => [],
        ];
        foreach ($this->menu_items as $link)
        {
            $menu['items'][] = $link->render();
        }
        return $menu;
    }

} 
```

Zoom in to this part right here. This is where the magic happens:

```php
foreach ($this->menu_items as $link)
{
    $menu['items'][] = $link->render();
}
```

We don't have to check whether the `render` method exists because we've implemented the `MenuInterface`. This is what allows us to nest in varying levels.

Lastly, we have the link. Think of this as the individual leaves on a stem. This represents the individual items in a menu. For its `render` method, we're simply returning an array containing the `link` and `text` passed to its constructor when an object is initialized:


```php
<?php 
// compositepattern/app/Link.php

namespace CompositePattern\App;

class Link implements MenuInterface
{
    
    public function __construct(public string $link, public string $text)
    {

    }


    public function render() : array
    {
        return [
            'link' => $this->link,
            'text' => $this->text,
        ];
    }
}
```

To test this out, let's create a single menu and attach two submenus to it. Each submenu will then have a number of menus attached to it, and each one has a link. We use the `add` method to link them together. Finally, calling the `render` method on the main menu will return every menu that was passed to it:


```php 
<?php 
// compositepattern/CompositeTest.php

require_once __DIR__ . '/../vendor/autoload.php';

use PHPUnit\Framework\TestCase;

class CompositeTest extends TestCase
{
    public function test_composite() : void
    {
        
        $main_menu = new \CompositePattern\App\Menu('Main');

        $herbs_menu = new \CompositePattern\App\Menu('Herbs');
       
        $herbs_menu->add(new \CompositePattern\App\Link('/rosemary', 'Rosemary'));
        $herbs_menu->add(new \CompositePattern\App\Link('/vietnamese-coriander', 'Vietnamese Coriander'));
        $herbs_menu->add(new \CompositePattern\App\Link('/tarragon', 'Tarragon'));

        
        $leafygreens_menu = new \CompositePattern\App\Menu('Leafy Greens');

        $leafygreens_menu->add(new \CompositePattern\App\Link('/bokchoi', 'Bokchoi'));
        $leafygreens_menu->add(new \CompositePattern\App\Link('/arugula', 'Arugula'));
        $leafygreens_menu->add(new \CompositePattern\App\Link('/pechay', 'Pechay'));
        $leafygreens_menu->add(new \CompositePattern\App\Link('/lettuce', 'Lettuce'));

        $main_menu->add($herbs_menu);
        $main_menu->add($leafygreens_menu);

       
        $this->assertEquals($main_menu->render(), [
            
            'title' => 'Main',
            'items' => [
                [
                    'title' => 'Herbs',
                    'items' => [
                        [
                            'link' => '/rosemary',
                            'text' => 'Rosemary',
                        ],
                        [
                            'link' => '/vietnamese-coriander',
                            'text' => 'Vietnamese Coriander',
                        ],
                        [
                            'link' => '/tarragon',
                            'text' => 'Tarragon',
                        ],
                    ]
                ],
                [
                    'title' => 'Leafy Greens',
                    'items' => [
                        [
                            'link' => '/bokchoi',
                            'text' => 'Bokchoi',
                        ],
                        [
                            'link' => '/arugula',
                            'text' => 'Arugula',
                        ],
                        [
                            'link' => '/pechay',
                            'text' => 'Pechay',
                        ],
                        [
                            'link' => '/lettuce',
                            'text' => 'Lettuce',
                        ],
                    ]
                ]
            ]
        ]);
       
    }
}
```


To run this. Be sure you have the following on your `composer.json` file then run `composer dump-autoload`:

```
"autoload": {
    "psr-4": {
        "CompositePattern\\App\\": "compositepattern/app/"
    }
},
```

Also install phpunit (`composer require phpunit/phpunit`) if you haven't done so already.

Then you can run the tests:

```bash
vendor/bin/phpunit compositepattern/CompositePattern.php
```

That's all there is to it to the Composite pattern. Whenever you find yourself working on tree or tree-like structures on your code, try to see if this pattern fits.

You can find the source code used in this tutorial on this [GitHub repo](https://github.com/anchetaWern/php-design-patterns).