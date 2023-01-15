---
author: "Wern Ancheta"
title: "PHP Design Patterns: Facade Pattern"
date: "2022-01-15"
---

This is the second post in a series of articles that will walk you through how to implement design patterns in PHP.

In this part, we're going to take a look at how you can implement the facade pattern.

The Facade pattern is a way of providing a simple and clear interface to a complex system. Because when you have a bunch of classes and they each have their own complex code, it may not be always clear which methods should be called by the client coder. The Facade pattern provides a solution to this problem. It does this by providing a gateway to which complex code is accessed via simple methods that achieve clear ends.

Take the following class for example. Just imagine that this is making calls to various endpoints of the [PokeAPI](https://pokeapi.co/) to get various Pokemon data:

```php 
<?php 
// facadepattern/app/PokemonData.php

namespace FacadePattern\App;

class PokemonData
{

    public function details()
    {
        return 'the details';
    }


    public function species()
    {
        return 'the species';
    }


    public function evolutionChain()
    {
        return 'the evolution chain';
    }


}
```

If you've ever tried using the PokeAPI previously, you know that it returns a whole bunch of data. So the above class might even call on a parser to get to the relevant data. Or it can have other methods it needs to call to construct the data that the user is asking for.

This is where the Facade pattern comes in. As it will be the one to provide a clear interface which the client coder can call on to easily achieve what they're trying to achieve.

In this case, you have create a PokemonDataGetter class which allows the user to get a combination of various Pokemon data without concerning themselves with how the underlying class works:

```php 
<?php 
// facadepattern/app/PokemonDataGetter.php

namespace FacadePattern\App;

class PokemonDataGetter
{   

    public static function fetch(array $methods): array 
    {
        $data = [];
        foreach ($methods as $method) {
            $data[$method] = (new PokemonData)->$method();
        }
        return $data;
    }

}
```

Here's the test code:

```php
<?php 
// facadepattern/FacadeTest.php

require_once __DIR__ . '/../vendor/autoload.php';

use PHPUnit\Framework\TestCase;

class FacadeTest extends TestCase
{
    public function test_pokemon_facade() : void
    {
        $details_response = FacadePattern\App\PokemonDataGetter::fetch(['details']);
        $this->assertEquals($details_response, [
            'details' => 'the details',
        ]);

        $species_response = FacadePattern\App\PokemonDataGetter::fetch(['species']);
        $this->assertEquals($species_response, [
            'species' => 'the species',
        ]);

        $evolution_chain_response = FacadePattern\App\PokemonDataGetter::fetch(['evolutionChain']);
        $this->assertEquals($evolution_chain_response, [
            'evolutionChain' => 'the evolution chain',
        ]);

        
        $details_species_response = FacadePattern\App\PokemonDataGetter::fetch(['details', 'species']);
        $this->assertEquals($details_species_response, [
            'details' => 'the details',
            'species' => 'the species',
        ]);

        $details_evolution_chain_response = FacadePattern\App\PokemonDataGetter::fetch(['details', 'evolutionChain']);
        $this->assertEquals($details_evolution_chain_response, [
            'details' => 'the details',
            'evolutionChain' => 'the evolution chain',
        ]);

        $species_evolution_chain_response = FacadePattern\App\PokemonDataGetter::fetch(['species', 'evolutionChain']);
        $this->assertEquals($species_evolution_chain_response, [
            'species' => 'the species',
            'evolutionChain' => 'the evolution chain',
        ]);

        $all_response = FacadePattern\App\PokemonDataGetter::fetch(['details', 'species', 'evolutionChain']);
        $this->assertEquals($all_response, [
            'details' => 'the details',
            'species' => 'the species',
            'evolutionChain' => 'the evolution chain',
        ]);
        
    }
} 
```

To run this. Be sure to add the following on your `composer.json` file then run `composer dump-autoload`:

```
"autoload": {
    "psr-4": {
        "FacadePattern\\App\\": "facadepattern/app/"
    }
},
```

Also install phpunit (`composer require phpunit/phpunit`) if you haven't done so already.

You can then run it by executing the following:

```bash
vendor/bin/phpunit facadepattern/app/FacadeTest.php
```

That's really all there is to the Facade pattern. It's a way to simplify the usage of an otherwise complex class by creating a facade class which accomplishes specific goals. 

You can find the source code used in this tutorial on this [GitHub repo](https://github.com/anchetaWern/php-design-patterns).

