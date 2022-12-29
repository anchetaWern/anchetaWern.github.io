---
author: "Wern Ancheta"
title: "A Primer to Service Container in Laravel"
date: "2021-08-15"
---


In this post we'll take a look at how to make use of the service container in Laravel.

Let's say we have a class for paying with Stripe:


```php
<?php

namespace App\Services;

class StripePaymentService {


    public function pay() 
    {
        return 'paid!';
    }
}
``` 

With type-hinting, you can easily make use of this class like so:

```php
<?php

use App\Services\StripePaymentService;

class PaymentsController extends Controller
{

    public function pay(StripePaymentService $stripe)
    {
        return $stripe->pay();
    }
```

This way you don't need to initialize it before you can call the method:

```php
$stripe = new StripePaymentService()
$stripe->pay();
```

Here's the route for those of you who likes to code along:

```php
Route::get('pay', 'PaymentsController@pay'); // it's GET for easy testing, should be POST in the real-world
```

But what if you need to supply arguments to it? This is where the Service Container comes into play. It provides a way for you to manage your class dependencies so you can easily inject them anywhere you need it. All of this without having to initialize the class each time.

Let's say we now have a constructor for our StripePaymentService class. This allows us to specify the payment method and the currency:

```php
class StripePaymentService {

    private $payment_method;
    private $currency;


    public function __construct($payment_method, $currency)
    {
        $this->payment_method = $payment_method;
        $this->currency = $currency;
    }


    public function pay()
    {
        return [
            'payment_method' => $this->payment_method,
            'currency' => $this->currency,
        ];
    }
}
```

With this change, we can no longer make use of the code from earlier since there's no way for us to supply the arguments for the constructor:


![Binding resolution exception](/images/posts/a-primer-to-service-container-in-laravel/exception.png)


The easiest way to solve this is via the `AppServiceProvider` class:

```php
<?php
// app/Providers/AppServiceProvider.php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Facades\Blade;
use App\Services\StripePaymentService; 

class AppServiceProvider extends ServiceProvider
{
   
    public function register()
    {

        $this->app->bind(StripePaymentService::class, function () {
            return new StripePaymentService('card', 'usd');
        });

    }

}
```

The key code is this one. What this does is return a new instance of the class everytime the `StripePaymentService` is injected anywhere:

```php
$this->app->bind(StripePaymentService::class, function () {
     return new StripePaymentService('card', 'usd');
});
```

With this code added, this bit of code will now work like usual:

```php
public function pay(StripePaymentService $stripe)
{
        return $stripe->pay();
}
```

In the real world, this isn't how it usually plays out. We usually have other classes interacting with the StripePaymentService. Let's now add it into our code.

Create an `app/Services/BasketService.php` file. This will handle our basket logic. But at the same time, it also needs to use the StripePaymentService to set the amount (basket total) and discount:

```php
<?php
namespace App\Services;

use App\Services\StripePaymentService;

class BasketService {

    private $stripe;

    // total: 45
    private $items = [
        [
            'title' => 'Biscuit',
            'price' => 2,
            'qty' => 10
        ],
        [
            'title' => 'Oranges',
            'price' => 5,
            'qty' => 5
        ],
    ];

    private $total;

    public function __construct(StripePaymentService $stripe)
    {
        $this->total = 0;
        $this->stripe = $stripe;
    }


    public function summarize()
    {
        $this->total = collect($this->items)->map(function($row) {
            return $row['price'] * $row['qty'];
        })->sum();

        $this->stripe->setAmount($this->total);
        $this->setDiscount();
    }


    public function setDiscount()
    {
        if ($this->total >= 40) {
            $this->stripe->setDiscount(10);
        }
    }

}
```

Open StripePaymentService and update it to handle the amount and the discount:

```php
// app/Services/StripePaymentService.php

private $payment_method;
private $currency;

// add these:
private $amount;
private $discount;

public function __construct($payment_method, $currency)
{
    $this->payment_method = $payment_method;
    $this->currency = $currency;
    
    // add these
    $this->amount = 0;
    $this->discount = 0;
}

public function setAmount($amount)
{
    $this->amount = $amount;
}


public function setDiscount($discount)
{
    $this->discount = $discount;
}

public function pay()
{
    return [
        'payment_method' => $this->payment_method,
        'currency' => $this->currency,
      
        // add these
        'amount' => $this->amount,
        'discount' => $this->discount,
    ];
}
```

Then in the PaymentsController, we're now ready to make use of the BasketService:

```php
// app/Http/Controllers/PaymentsController.php

use App\Services\StripePaymentService;
use App\Services\BasketService;

class PaymentsController extends Controller
{
    // update this
    public function pay(StripePaymentService $stripe, BasketService $basket)
    {

        $basket->summarize();
        return $stripe->pay();
    }
    // ...
}
```

When you run it on the browser though, you'll see that it doesn't actually perform as expected:

![payment method output](/images/posts/a-primer-to-service-container-in-laravel/payment-method-output.png)



The problem with this is the way we're binding the StripePaymentService class. Instead of `$this->app->bind()`, we should call `$this->app->singleton()` instead. This way, it wouldn't create a new instance everytime the class is injected anywhere else in the code. Using a singleton means that it will check to see if a class had already been instantiated before creating a new instance again. If it has already been instantiated then it will simply return the old one.

```php
// app/Providers/AppServiceProvider.php
public function register()
{

    $this->app->singleton(StripePaymentService::class, function () {
        return new StripePaymentService('card', 'usd');
    });

}
```

With that change, we now see the expected output:

![payment method output](/images/posts/a-primer-to-service-container-in-laravel/payment-method-output2.png)

Note that we didn't really need to add BasketService into the AppServiceProvider like so:

```php
$this->app->singleton(BasketService::class, function () {
    return new BasketService;
});
```

This is because we don't need it yet. Most likely, you'll be using sessions to store the items data anyway, so you don't really need a singleton for that.

Now what if we need to introduce a new method of collecting payments? PayPal for example.

Most likely, the code should be pretty similar like so. Note that `payment_method` is removed since it's assumed that the PayPal balance will be used:

```php
<?php

namespace App\Services;

class PaypalPaymentService {

    private $currency;

    private $amount;
    private $discount;


    public function __construct($currency)
    {
        $this->currency = $currency;

        $this->amount = 0;
        $this->discount = 0;
    }


    public function setAmount($amount)
    {
        $this->amount = $amount;
    }


    public function setDiscount($discount)
    {
        $this->discount = $discount;
    }


    public function pay()
    {
        return [
            'currency' => $this->currency,
            'amount' => $this->amount,
            'discount' => $this->discount,
        ];
    }
}
```

Then you'll have to add it again to your AppServiceProvider:

```php
// app/Providers/AppServiceProvider.php

$this->app->singleton(PaypalPaymentService::class, function () {
    return new PaypalPaymentService('card', 'usd');
});
```

As well as everywhere else you used StripePaymentService earlier.

And then you'll have to do this over and over as you accept more types of payments. Not really ideal in the long run.

We can solve this by creating an interface in which all payment types (Stripe, PayPal, etc.) will be based on. This way, we can inject the interface instead of the individual classes wherever we need it.

Create an `app/Services/PaymentServiceInterface.php`. This will serve as the "blue-print" for all our payment types:

```php
<?php
namespace App\Services;

interface PaymentServiceInterface {

    public function setAmount($amount);

    public function setDiscount($discount);

    public function pay();
}
```

Then in your individual classes, all you have to do is implement the PaymentServiceInterface. The rest of the code will remain the same:

```php
<?php

namespace App\Services;

use App\Services\PaymentServiceInterface; // add this

class StripePaymentService implements PaymentServiceInterface { // implement the PaymentServiceInterface


    private $payment_method;
    private $currency;

    private $amount;
    private $discount;


    public function __construct($payment_method, $currency)
    {
        $this->payment_method = $payment_method;
        $this->currency = $currency;

        $this->amount = 0;
        $this->discount = 0;
    }


    public function setAmount($amount)
    {
        $this->amount = $amount;
    }


    public function setDiscount($discount)
    {
        $this->discount = $discount;
    }


    public function pay()
    {
        return [
            'service' => 'stripe', // add this so we can see which class is being used
            'payment_method' => $this->payment_method,
            'currency' => $this->currency,
            'amount' => $this->amount,
            'discount' => $this->discount,
        ];
    }
}
```

Do the same for the PaypalPaymentService:

```php
<?php

namespace App\Services;

use App\Services\PaymentServiceInterface;

class PaypalPaymentService implements PaymentServiceInterface {

    private $currency;

    private $amount;
    private $discount;


    public function __construct($currency)
    {
        $this->currency = $currency;

        $this->amount = 0;
        $this->discount = 0;
    }


    public function setAmount($amount)
    {
        $this->amount = $amount;
    }


    public function setDiscount($discount)
    {
        $this->discount = $discount;
    }


    public function pay()
    {
        return [
            'service' => 'paypal',
            'currency' => $this->currency,
            'amount' => $this->amount,
            'discount' => $this->discount,
        ];
    }
}
```

Then update your AppServiceProvider to return a new instance of the correct class based on the request input that's passed in:

```php
// app/Providers/AppServiceProvider.php

use App\Services\PaymentServiceInterface;
use App\Services\StripePaymentService;
use App\Services\PaypalPaymentService;

class AppServiceProvider extends ServiceProvider
{

    public function register()
    {
        $this->app->singleton(PaymentServiceInterface::class, function () {

            if (request()->has('stripe')) {
                return new StripePaymentService('card', 'usd');
            }
            return new PaypalPaymentService('usd');
        });
    }
}
```

The final step is to update all instances where you used the StripePaymentService and PaypalPaymentService to use the PaymentServiceInterface instead.

First, we have the BasketService:

```php
<?php
// app/Services/BasketService.php

namespace App\Services;

use App\Services\PaymentServiceInterface;

class BasketService {

    private $payor;

    private $items = [
        [
            'title' => 'Biscuit',
            'price' => 2,
            'qty' => 10
        ],
        [
            'title' => 'Oranges',
            'price' => 5,
            'qty' => 5
        ],
    ];

    private $total;

    public function __construct(PaymentServiceInterface $payor)
    {
        $this->total = 0;
        $this->payor = $payor;
    }


    public function summarize()
    {
        $this->total = collect($this->items)->map(function($row) {
            return $row['price'] * $row['qty'];
        })->sum();

        $this->payor->setAmount($this->total);
        $this->setDiscount();
    }


    public function setDiscount()
    {
        if ($this->total >= 40) {
            $this->payor->setDiscount(10);
        }
    }

}
```

Next, we have the controller:

```php
use App\Services\PaymentServiceInterface;

class PaymentsController extends Controller
{

    public function pay(PaymentServiceInterface $payor, BasketService $basket)
    {

        $basket->summarize();
        return $payor->pay();
    }
}
```

Now, when you supply `stripe` in the request, you'll trigger the StripePaymentService:


![payment method output stripe](/images/posts/a-primer-to-service-container-in-laravel/payment-method-output-stripe.png)

Otherwise, you get the PayPal one:


![payment method output paypal](/images/posts/a-primer-to-service-container-in-laravel/payment-method-output-paypal.png)

Now, anytime you need to add a new payment type, all you have to do is create a new class which adheres to the PaymentServiceInterface then add the corresponding check on AppServiceProvider.

One final thing before I let you go is the use of service provider classes. Sooner or later your AppServiceProvider file will fill with a bunch of codes under the `register()` method. To avoid that problem, you can create a separate service provider class for each interface.

Execute the following on your terminal:

```bash
php artisan make:provider PaymentServiceProvider
```

That will generate a new file `app/Providers/PaymentServiceProvider.php`. Here, you will just copy and paste the specific binding code you have for the interface that you want this provider to cater to. In this case, we have the PaymentServiceInterface:

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;

use App\Services\PaymentServiceInterface;
use App\Services\StripePaymentService;
use App\Services\PaypalPaymentService;

class PaymentServiceProvider extends ServiceProvider
{
   
    public function register()
    {
        $this->app->singleton(PaymentServiceInterface::class, function() {
            if (request()->has('stripe')) {
                return new StripePaymentService('card', 'usd');
            }
            return new PaypalPaymentService('usd');
        });
    }

  
    public function boot()
    {
        //
    }
}
```

The final step is add that provider to the `providers` array in the `config/app.php` file:

```php
'providers' => [
    // ...
    App\Providers\PaymentServiceProvider::class,
]
```

## Summary

Laravel's Service Container is a useful tool to add in our arsenal to keep our code tidy. By using it in combination with interfaces, you can easily manage your class dependencies. Be sure to read the official documentation for more information regarding the [Service Container](https://laravel.com/docs/8.x/container).

*Cover Image from Guillaume Bolduc: https://unsplash.com/photos/uBe2mknURG4*
