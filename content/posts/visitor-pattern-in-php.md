---
author: "Wern Ancheta"
title: "PHP Design Patterns: Visitor Pattern"
date: "2023-04-30"
---

This is the 17th post in a series of articles that will walk you through how to implement design patterns in PHP.

In this part, we're going to take a look at how you can implement the Visitor Pattern.

The Visitor Pattern is used when you have two or more related entities where you need to do some processing to create some output.

For example, an HTML form builder class where you need to be able to break down the components which makes it up. The form hierarchy usually looks like this:

```
form > fieldset -> input field
```

So a form is made up of fieldsets, and a fieldset can have multiple input fields. We want to create a class that can process all of those components.
To implement that, we need to put something in place to treat those individual components as if they were the same entity. That way, we don't have to add conditional statements checking for types and whatnot. 

Here it is represented in code:

```php
<?php 
class Form
{
    public function __construct(private string $id, private array $fieldsets) 
    {

    }

    public function getFieldsets() : array 
    {
        return $this->fieldsets;
    }
}

class Fieldset
{
    public function __construct(private string $id, private array $inputFields) 
    {

    }

    public function getInputFields() : array 
    {
        return $this->inputFields;
    }
}

class InputField
{
    public function __construct(private string $id, private string $type) 
    {

    }

    public function getType() : string 
    {
        return $this->type;
    }
}
```

To process those three entities, you have a FormRenderer class:


```php 
<?php
class FormRenderer
{

    public function handle($entity)
    {
        if ($entity instanceOf Form) {
            // ..
        } else if ($entity instanceOf Fieldset) {
            // ..
        } else if ($entity instanceOf InputField) {
            // ..
        }
    }

}
```

You probably already see where this is going. Basically, for every new entity that you add, you're also going to need to update this conditional.

This is where the Visitor Pattern comes in. It allows you to treat these individual entities as if they were the same entity. That way, you can eliminate the type checks and have a cleaner-looking code in general.


### Implementation

To implement the Visitor pattern, we first need to create an interface in which all the entities will adhere to. In this case, we need to add the `accept` method which accepts a `Visitor` instance. The visitor is the class that will process each of the different entities. In our example above, we have the `FormRenderer` as a type of visitor. But you can also have a `FormTreeGenerator` or a `FormJsonGenerator`.


```php
<?php 
// visitorpattern/app/FormComponent.php

namespace VisitorPattern\App;

use VisitorPattern\App\Visitor;


interface FormComponent {
    
    public function accept(Visitor $visitor) : string;
}
```

Next, we can now have our entities implement the `FormComponent` method. This is where you call the method from the visitor for specifically visiting this entity. So you're basically passing an instance of the entity so the visitor can do whatever it's meant to do to the entity. We also have a couple of other methods: `getId` and `getFieldsets` whose sole purpose is to return whatever arguments are passed in the constructor:

```php 
<?php 
// visitorpattern/app/Form.php

namespace VisitorPattern\App;

use VisitorPattern\App\FormComponent;
use VisitorPattern\App\Visitor;


class Form implements FormComponent
{

    
    public function __construct(private string $id, private array $fieldsets) 
    {

    }


    public function getId() : string
    {
        return $this->id;
    }


    public function getFieldsets() : array 
    {
        return $this->fieldsets;
    }


    public function accept(Visitor $visitor) : string
    {
        return $visitor->visitForm($this);
    }

}
```

Do the same for the Fieldset:


```php 
<?php 
// visitorpattern/app/Fieldset.php

namespace VisitorPattern\App;

use VisitorPattern\App\FormComponent;
use VisitorPattern\App\Visitor;


class Fieldset implements FormComponent
{

    public function __construct(private string $id, private array $inputFields) 
    {

    }


    public function getId() : string
    {
        return $this->id;
    }


    public function getInputFields() : array 
    {
        return $this->inputFields;
    }


    public function accept(Visitor $visitor) : string
    {
        return $visitor->visitFieldset($this);
    }

}
```

And the InputField:

```php 
<?php 
// visitorpattern/app/InputField.php

namespace VisitorPattern\App;

use VisitorPattern\App\FormComponent;
use VisitorPattern\App\Visitor;


class InputField implements FormComponent
{

    
    public function __construct(private string $id, private string $type) 
    {

    }


    public function getId() : string
    {
        return $this->id;
    }


    public function getType() : string 
    {
        return $this->type;
    }


    public function accept(Visitor $visitor) : string
    {
        return $visitor->visitInputField($this);
    }

}
```

Next, we create the Visitor interface. Each Visitor class has to be abide to this interface so this is where you specify all the methods that each visitor class needs to implement. In this case, we want each one to have a separate method for visiting a form, a fieldset, and an input field. If you have other entities that needs visiting, this is also where you specify them:

```php 
<?php 
// visitorpattern/app/Visitor.php

namespace VisitorPattern\App;

use VisitorPattern\App\Form;
use VisitorPattern\App\Fieldset;
use VisitorPattern\App\InputField;

interface Visitor 
{

    public function visitForm(Form $form) : string;

    public function visitFieldset(Fieldset $fieldset) : string;

    public function visitInputField(InputField $inputField) : string;

}
```

Next, have the FormRenderer implement the Visitor interface. This is where you add the logic for visiting each entity. In this case, we're simply returning the HTML equivalent of the arguments that were passed when each entity is instantiated:


```php 
<?php 
// visitorpattern/app/FormRenderer.php

namespace VisitorPattern\App;

use VisitorPattern\App\Visitor;
use VisitorPattern\App\Form;
use VisitorPattern\App\Fieldset;
use VisitorPattern\App\InputField;


class FormRenderer implements Visitor
{

    public function visitForm(Form $form) : string
    {
        $html = '';
        foreach ($form->getFieldsets() as $fieldset)
        {   
            $html .= $this->visitFieldset($fieldset);
        }

        return $html;
    }


    public function visitFieldset(Fieldset $fieldset) : string
    {
        $html = '<fieldset id="' . $fieldset->getId() . '">';
        foreach ($fieldset->getInputFields() as $field)
        {
            $html .= $this->visitInputField($field);
        }
        $html .= "</fieldset>";

        return $html;
    }


    public function visitInputField(InputField $inputField) : string
    {
        if ($inputField->getType() === 'textarea')
        {
            return '<textarea name="' . $inputField->getId() . '"></textarea>';
        }

        return '<input type="' . $inputField->getType() . '" name="' . $inputField->getId() . '" />';
    }

}
```


Here's the test code:


```php
<?php 
// visitorpattern/VisitorTest.php

require_once __DIR__ . '/../vendor/autoload.php';

use PHPUnit\Framework\TestCase;

use VisitorPattern\App\Form;
use VisitorPattern\App\Fieldset;
use VisitorPattern\App\InputField;

use VisitorPattern\App\FormRenderer;

class VisitorTest extends TestCase
{

    public function test_visitor()
    {

        $businessDetailsFieldset = new Fieldset('businessDetails', [
            new InputField('name', 'text'),
            new InputField('address', 'text'),
            new InputField('about_text', 'textarea'),
        ]);

        $businessContactFieldset = new Fieldset('businessContact', [
            new InputField('contact', 'text'),
            new InputField('email', 'email'),
            new InputField('phone_number', 'number'),
        ]);

        $businessSignupForm = new Form('businessSignup', [
            $businessDetailsFieldset,
            $businessContactFieldset,
        ]);

        $formRenderer = new FormRenderer();

        $html = $businessSignupForm->accept($formRenderer);
        $this->assertEquals($html, '<fieldset id="businessDetails"><input type="text" name="name" /><input type="text" name="address" /><textarea name="about_text"></textarea></fieldset><fieldset id="businessContact"><input type="text" name="contact" /><input type="email" name="email" /><input type="number" name="phone_number" /></fieldset>');

    }

}
```

To run this, update the `composer.json` file to add the path:


```
"autoload": {
    "psr-4": {
        // ..
        "VisitorPattern\\App\\": "visitorpattern/app/"
    }
},
```

Also install phpunit (`composer require phpunit/phpunit`) if you haven't done so already.

You can then run it by executing the following:

```bash
vendor/bin/phpunit visitorpattern/app/VisitorTest.php
```

That's all there is to it to the Visitor pattern. This is one of the few patterns that doesn't have much application in PHP. But if you do encounter some problem that looks like what we solved above then you now knkow which pattern to use. This can be used mostly for report generation for different entities in your system.


You can find the source code used in this tutorial on this [GitHub repo](https://github.com/anchetaWern/php-design-patterns).

