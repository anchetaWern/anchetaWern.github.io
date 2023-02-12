---
author: "Wern Ancheta"
title: "PHP Design Patterns: Chain of Responsibility Pattern"
date: "2023-02-12"
---

This is the sixth post in a series of articles that will walk you through how to implement design patterns in PHP.

In this post, we'll take a look at how you can implement the Chain of Responsibility pattern.

This pattern allows you to chain related code together in such a way that the control is passed to the next link in the chain if the condition you're checking for in the current chain passes. This pattern is commonly used in implementing middleware or validation code. So if the condition or validation rule passes for one middleware, it just passes the control to the next until such time that all the middleware are passing.

Say we have the following `Developer` class which accepts boolean values of the things they already tried when solving a specific problem:

```php
<?php 
// chainofresponsibilitypattern/app/Developer.php

namespace ChainOfResponsibilityPattern\App;

class Developer
{

    public function __construct(public bool $hasReadDocumentation, public bool $hasSearchedGoogle, public bool $hasAskedStackoverflow)
    {

    }

}
```

Next, we create a separate class for each of the ways a problem can be solved: `ReadDocumentation`, `SearchGoogle`, `AskStackOverflow`:

```php 
<?php 
// chainofresponsibilitypattern/app/ReadDocumentation.php

namespace ChainOfResponsibilityPattern\App;

use ChainOfResponsibilityPattern\App\Exceptions\DeveloperDidNotReadDocumentationException;

class ReadDocumentation extends SolveIssue 
{
 
    public function solve(Developer $developer) 
    {
        if (!$developer->hasReadDocumentation) throw new DeveloperDidNotReadDocumentationException;
        
        $this->next($developer);
    }
}
```


```php
<?php 
// chainofresponsibilitypattern/app/SearchGoogle.php

namespace ChainOfResponsibilityPattern\App;
use ChainOfResponsibilityPattern\App\Exceptions\DeveloperDidNotSearchGoogleException;

class SearchGoogle extends SolveIssue 
{

    public function solve(Developer $developer) 
    {
        if (!$developer->hasSearchedGoogle) throw new DeveloperDidNotSearchGoogleException;
        
        $this->next($developer);
    }
} 
```

```php
<?php 
// chainofresponsibilitypattern/app/AskStackOverflow.php

namespace ChainOfResponsibilityPattern\App;
use ChainOfResponsibilityPattern\App\Exceptions\DeveloperDidNotAskStackOverflowException;

class AskStackOverflow extends SolveIssue 
{

    public function solve(Developer $developer) 
    {
        if (!$developer->hasAskedStackoverflow) throw new DeveloperDidNotAskStackOverflowException;
        
        $this->next($developer);
    }

}
```

Since the Chain of Responsibility is usually used for middlewares and input validation, we need to create our own exception classes for each of the classes we created above. This way, we have a specific exception to throw to the user in case the condition fails:

```php
<?php 
// chainofresponsibilitypattern/app/Exceptions/DeveloperDidNotSearchGoogleException.php

namespace ChainOfResponsibilityPattern\App\Exceptions;
use Exception;

class DeveloperDidNotReadDocumentationException extends Exception
{

    public function __construct() {
       
        parent::__construct("You haven't read the documentation yet.", code: 0, previous: null);
    }

    public function __toString() {
        return __CLASS__ . ": [{$this->code}]: {$this->message}\n";
    }

}
```


```php
<?php 
// chainofresponsibilitypattern/app/Exceptions/DeveloperDidNotSearchGoogleException.php

namespace ChainOfResponsibilityPattern\App\Exceptions;
use Exception;

class DeveloperDidNotSearchGoogleException extends Exception
{

    public function __construct() {
       
        parent::__construct("You haven't searched google yet.", code: 0, previous: null);
    }

    public function __toString() {
        return __CLASS__ . ": [{$this->code}]: {$this->message}\n";
    }

} 
```


```php
<?php 
// chainofresponsibilitypattern/app/Exceptions/DeveloperDidNotAskStackOverflowException.php

namespace ChainOfResponsibilityPattern\App\Exceptions;
use Exception;

class DeveloperDidNotAskStackOverflowException extends Exception
{

    public function __construct() {
       
        parent::__construct("You haven't asked stackoverflow yet.", code: 0, previous: null);
    }

    public function __toString() {
        return __CLASS__ . ": [{$this->code}]: {$this->message}\n";
    }

} 
```

Next, we now implement the main bulk of this pattern. As you've seen earlier, we were extending the `SolveIssue` class. This is where we declare the common functionality for each of the classes as well as the method which they need to implement. At its core, the Chain of Responsibility pattern needs a way to:

1. Chain multiple classes together.
2. Call on the next link in the chain.

That's really all there is to it. To implement the first one, we have the `then` method. What this does is assign the next link in the chain. This needs to accept the same type (`SolveIssue`) otherwise it wouldn't work. We simply assign this next class to a local class variable for easy access later on.

Next, we have the `next` method. This is accepts the input that we want to check. In this case it's the `Developer`. If there's no longer a next link in the chain then we simply return control to the caller. If there is then we call on the `solve` method of that class. We know for sure that this method exists because we already performed a type-check when the `then` method was called.

Lastly, we leave it to the child class to implement the `solve` method.

```php 
<?php 
// chainofresponsibilitypattern/app/SolveIssue.php

namespace ChainOfResponsibilityPattern\App;

abstract class SolveIssue 
{

    protected $nextSolver;

    public function then(SolveIssue $solver)
    {
        $this->nextSolver = $solver;
    }


    protected function next(Developer $developer) 
    {
        if (!$this->nextSolver) return;

        $this->nextSolver->solve($developer);
    }

    abstract public function solve(Developer $developer);

}
```

At this point, we can now add the client code. Here, the main goal is to assert that the correct exceptions are being triggered based on the parameters we passed when creating a new `Developer` object. This also depends on the order in which the links on the chain is arranged. For example, if the developer did not read the documentation and we have the `ReadDocumentation` as the first link in the chain then surely the `DeveloperDidNotReadDocumentationException` will be triggered first and the chain is terminated immediately after the first class does its check. On the other hand, if it goes by the following order: `SearchGoogle` -> `AskStackOverflow` -> `ReadDocumentation` and we have the following options passed: `hasReadDocumentation: false, hasSearchedGoogle: true, hasAskedStackoverflow: true` then it will have to go through all the classes:

```php
<?php 
// chainofresponsibilitypattern/ChainOfResponsibilityTest.php

require_once __DIR__ . '/../vendor/autoload.php';

use PHPUnit\Framework\TestCase;

class ChainOfResponsibilityTest extends TestCase
{
    public function test_chain_one() : void
    {
        $this->expectException(ChainOfResponsibilityPattern\App\Exceptions\DeveloperDidNotReadDocumentationException::class);

        $developer = new ChainOfResponsibilityPattern\App\Developer(hasReadDocumentation: false, hasSearchedGoogle: false, hasAskedStackoverflow: false);
        
        $readDocumentation = new ChainOfResponsibilityPattern\App\ReadDocumentation;
        $searchGoogle = new ChainOfResponsibilityPattern\App\SearchGoogle;
        $askStackOverflow = new ChainOfResponsibilityPattern\App\AskStackOverflow;

        $readDocumentation->then($searchGoogle);
        $searchGoogle->then($askStackOverflow);

        $readDocumentation->solve($developer);
    }


    public function test_chain_two() : void 
    {
        $this->expectException(ChainOfResponsibilityPattern\App\Exceptions\DeveloperDidNotSearchGoogleException::class);

        $developer = new ChainOfResponsibilityPattern\App\Developer(hasReadDocumentation: true, hasSearchedGoogle: false, hasAskedStackoverflow: true);
        
        $readDocumentation = new ChainOfResponsibilityPattern\App\ReadDocumentation;
        $searchGoogle = new ChainOfResponsibilityPattern\App\SearchGoogle;
        $askStackOverflow = new ChainOfResponsibilityPattern\App\AskStackOverflow;

        $askStackOverflow->then($searchGoogle);
        $searchGoogle->then($readDocumentation);

        $askStackOverflow->solve($developer);
    }


    public function test_chain_three() : void 
    {
        $this->expectException(ChainOfResponsibilityPattern\App\Exceptions\DeveloperDidNotAskStackOverflowException::class);

        $developer = new ChainOfResponsibilityPattern\App\Developer(hasReadDocumentation: true, hasSearchedGoogle: true, hasAskedStackoverflow: false);
        
        $readDocumentation = new ChainOfResponsibilityPattern\App\ReadDocumentation;
        $searchGoogle = new ChainOfResponsibilityPattern\App\SearchGoogle;
        $askStackOverflow = new ChainOfResponsibilityPattern\App\AskStackOverflow;

        $searchGoogle->then($readDocumentation);
        $searchGoogle->then($askStackOverflow);

        $searchGoogle->solve($developer);
    }
} 
```


To run this. Be sure you have the following on your `composer.json` file then run `composer dump-autoload`:

```
"autoload": {
    "psr-4": {
        "ChainOfResponsibilityPattern\\App\\": "chainofresponsibilitypattern/app"
    }
},
```

Also install phpunit (`composer require phpunit/phpunit`) if you haven't done so already.

Then you can run the tests:

```bash
vendor/bin/phpunit chainofresponsibilitypattern/ChainOfResponsibilityTest.php
```

That's the Chain of Responsibility pattern. There's not really much to say about it since it's just a way to chain a bunch of related checks together. It's flexible enough to accomodate any order in which the checks are linked together.

You can find the source code used in this tutorial on this [GitHub repo](https://github.com/anchetaWern/php-design-patterns).