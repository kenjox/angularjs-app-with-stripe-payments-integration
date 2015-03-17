## 1 Introduction

Stripe is a popular online payment processing service used by Lyft, TaskRabbit, Instacart, and many other well-known companies. They charge a fixed fee of 2.9% + $0.30 per transaction, and their service can be easily integrated into web and mobile applications.

In this tutorial, you are going to learn how to integrate Stripe into your AngularJS web application. We will build a simple web application that displays a credit card form and uses Stripe for billing.

## 2  Before we get started

### 2.1  Prerequisites

In order to process credit card details, your web application must be hosted on an HTTPS server. You will need to get a valid HTTPS certificate from a certification authority, but don't worry &mdash; a certificate can cost as low as 10-20$ and can be issued within minutes. Check out [Stripe's SSL page](https://stripe.com/help/ssl) for information.

### 2.2  AngularJS modules for working with Stripe

If you perform a quick search on GitHub, you will find that there are at least six different libraries / modules aiming to bring Stripe payments to Angular. This is not uncommon &mdash; in fact, for most tasks you want to perform, you will usually end up with several open source alternatives, and choosing is not always easy.

There is the list of all the alternatives that I could find at the time of writing this article:

* [https://github.com/laurihy/angular-payments](https://github.com/laurihy/angular-payments)
* [https://github.com/seandesmond/angular-payment](https://github.com/seandesmond/angular-payment)
* [https://github.com/gtramontina/stripe-angular](https://github.com/gtramontina/stripe-angular)
* [https://github.com/bendrucker/angular-stripe](https://github.com/bendrucker/angular-stripe)
* [https://github.com/Venturocket/angular-stripe-js](https://github.com/Venturocket/angular-stripe-js)
* [https://github.com/raaspazasu/ngPayments](https://github.com/raaspazasu/ngPayments)

I decided to go with laurihy/angular-payments, as it has been starred much more than all the others. Open source projects with many stars are usually a better choice, as the amount of stars is usually a good indicator of how many happy users the project has.

### 2.3  The demo code

The source code that is presented in this tutorial is also available on [GitHub](https://github.com/airpair/ng-stripe-tutorial). There are three sub-directories corresponding to three tutorial steps, and there is also a [nice demo project](http://www.airpair.com/wp-content/themes/epik/stripe-angular/) which implements a pet shop with a simple cart module and a checkout form.

[![](http://airpair-blog.s3.amazonaws.com/wp-content/uploads/2014/09/stripe-angular-demo-project.jpg)](http://www.airpair.com/wp-content/themes/epik/stripe-angular/)

## 3  The integration

### 3.1  The Stripe.js library

Stripe provides a JavaScript client library for interacting with their API. Add the following line to include their library in your HTML page:

<!-- code lang=markup linenums=true -->

    <script type="text/javascript" src="https://js.stripe.com/v2/"></script>

After loading the library, you need to initialize it with your publisher key. You can initialize it even before you load AngularJS, but I personally prefer to initialize it within my AngularJS config block:

<!-- code lang=javascript linenums=true -->

    angular.module('myApp').config(function($window) {
        $window.Stripe.setPublishableKey('YOUR-KEY-COMES-HERE');
    });

An alternative approach would be to load the Stripe.js library asynchronously, after your Angular.js application has loaded and initialized. You can use [angular-load](https://github.com/urish/angular-load) — an open-source Angular module written by the author of this article — to achieve that.

### 3.2  Angular-payment

The next step would be to load the angular-payment module to your page. The preferred method is to use the [Bower Package Manager](http://bower.io/) install the library:

<!-- code lang=bash linenums=true -->

    $ bower install --save angular-payments

Unfortunately, angular-payments still doesn't have any version tags in its repository, which means that your code may suddenly fail if they release a new version and break some APIs. However, there is a way to get around this — you can define a specific commit hash to lock the dependency to a particular version:

<!-- code lang=bash linenums=true -->

    $ bower install --save angular-payments#2472bc9befa25678

Depending on your project setup, you may also need to include the library in your HTML page file:

<!-- code lang=markup linenums=true -->

    <script type="text/javascript" 
            src="bower_components/angular-payments/lib/angular-payments.js">
    </script>

Finally, you need to add the `angularPayments` module as a dependency of your AngularJS application:

<!-- code lang=javascript linenums=true -->

    angular.module('myApp', ['angularPayments'])

In the demo application we also have a few other dependencies:

* [mm.foundation](https://github.com/pineconellc/angular-foundation) - AngularJS directives for [Zurb's Foundation CSS Framework](http://foundation.zurb.com/)
* [ngAnimate](https://docs.angularjs.org/guide/animations) - Used for animating the entrance of the checkout pane.
* [angularSpinner](https://github.com/urish/angular-spinner) - Displays a nice spinner animation during checkout (this is actually a module that I wrote)

Now that we're all set, we are ready to move on and create our checkout form.

### 3.3  Creating the checkout form

Angular-payment uses a declarative approach for setting up the checkout form. Let's start with the basics — add the following code to your view template:

<!-- code lang=markup linenums=true -->

    <form stripe-form="stripeCallback">
        <input ng-model="number" placeholder="Card Number" />
        <input ng-model="expiry" placeholder="Expiration" />
        <input ng-model="cvc" placeholder="CVC" />
        <button type="submit">Submit</button>
    </form>

...and create a callback on your scope:

<!-- code lang=javascript linenums=true -->

    // Stripe Response Handler
    $scope.stripeCallback = function (code, result) {
        if (result.error) {
            window.alert('it failed! error: ' + result.error.message);
        } else {
            window.alert('success! token: ' + result.id);
        }
    };

#### 3.3.1  How does this work?

The `stripe-form` directive is the brain behind this checkout form. It attaches to the form's submit event, extracts all the values from the form's scope, and submits them to Stripe. Finally, when Stripe returns a response, it invokes the specified callback with the response.

If you want to call your Stripe response handler a different name, make sure you change the value of the `stripe-form` attribute to match the name of your Stripe response handler function.

#### 3.3.2  Optional fields & demo

According to [Stripe's documentation](https://stripe.com/docs/stripe.js), you could also include the following fields in your form:

* `name` - The full name of the card's holder
* `address_line1`, `address_line2`, `address_city`, `address_state`, `address_zip`, `address_country`- The billing address.

Those fields are not required and do not affect the validation of the credit card during the creation of the token.

Check out the [online demo](http://www.airpair.com/wp-content/themes/epik/stripe-angular/) for our checkout form. Input the following information to get a positive response:

Card number:  **4242-4242-4242-4242**
Expiry:      **12/16** (or any other date in the near future)
CVC:         **999**

### 3.4  Field formatting

Angular-payments provides a directive for formatting the card number, expiration and CVC input fields. The directive will take care of enforcing numerical input, splitting the card number into groups of four digits as the user types it, splitting the expiration date into month and year, and will also set the maximum input length for each of the fields automatically.

Adding the payments-format directive to our form is very simple:

<!-- code lang=javascript linenums=true -->

    <input ng-model="number" placeholder="Card Number" payment-format="card" />
    <input ng-model="expiry" placeholder="Expiration" payment-format="expiry"/>
    <input ng-model="cvc" placeholder="CVC" payment-format="cvc" />
    
Check out the [online demo](http://urish.github.io/ng-stripe-tutorial/step2/step2.html) for this step.

### 3.5  Form Validation

There is one more step before we can declare our checkout form complete — adding input validation for the fields. Credit card numbers [have a very specific format](https://en.wikipedia.org/wiki/Bank_card_number) which includes a set of predefined issuer prefixes, as well as a final digit which is used for validating the number by applying the [Luhn algorithm](https://en.wikipedia.org/wiki/Luhn_algorithm).

Fortunately, you don't have to deal with all those details yourself, as angular-payments already includes all the necessary validation mechanisms. In addition, it integrates nicely with Angular's [Form Validation mechanism](https://docs.angularjs.org/guide/forms#binding-to-form-and-control-state).

We start by adding the payments-validate directive to our input elements:

<!-- code lang=javascript linenums=true -->

    <input ng-model="number" placeholder="Card Number" payments-format="card"
       payments-validate="card" />
    <input ng-model="expiry" placeholder="Expiration" payments-format="expiry"
       payments-validate="expiry" />
    <input ng-model="cvc" placeholder="CVC" payments-format="cvc"
       payments-validate="cvc" />
    
Then, we add a new CSS rule that will be triggered for invalid input values:

<!-- code lang=css linenums=true -->

    .ng-invalid {
        color: red;
    }
    
We can also display an error message in case of invalid input data. In order to do this, we give our form and each of those inputs a name:

<!-- code lang=javascript linenums=true -->

    <form stripe-form="stripeCallback" name="checkoutForm">>
        <input ng-model="number" placeholder="Card Number" 
                 payments-format="card" payments-validate="card" name="card" />
        <input ng-model="expiry" placeholder="Expiration" 
                 payments-format="expiry" payments-validate="expiry"                
                 name="expiry" />
        <input ng-model="cvc" placeholder="CVC" payments-format="cvc" payments-validate="cvc" name="cvc" />
        <button type="submit">Submit</button>
    </form>    

Finally, we can check the validation status of each of the fields and display an error message as needed. For example, here is the code for displaying an error message based on the validity of the card number:

<!-- code lang=javascript linenums=true -->

    <div ng-if="checkoutForm.card.$invalid">
        Error: invalid card number!
    </div>

You can experiment with the validation code in the [online demo](http://urish.github.io/ng-stripe-tutorial/step3/step3.html) for step 3.

### 3.6  Sending the token to your server

Once you have successfully acquired a single-use credit card token, you need to send this token to your server to complete the billing process. You will probably want to use Angular's built-in [`$http module`](https://docs.angularjs.org/api/ng/service/$http) or a third-party REST API library (such as [RestAngular](https://github.com/mgonto/restangular)) for sending the credit card token to your server.

## 4  Next steps and wrapping up

### 4.1  Making the actual charge

Check out [this tutorial from Stripe](https://stripe.com/docs/tutorials/charges) for an explanation how to actually charge the user using the single-use credit card token. They have code samples for Node.js, PHP, Python and Ruby. If you use a different language for the backend code, check out their [API Libraries page](https://stripe.com/docs/libraries) or interface directly with their [REST API](https://stripe.com/docs/api/curl).

### 4.2  Recurring payments

Stripe has built-in support for subscriptions, which you can use for charging recurring payments. For your AngularJS application the flow is essentially the same, and concludes by sending a single user credit card token to your back end code. On the back end, you will need to create a Subscription Plan first, and then you will be able to subscribe customers to the plan and make recurring payments. You can find the complete details in the [creating your first subscription tutorial](https://stripe.com/docs/tutorials/subscriptions) by Stripe.

### 4.3  Demo project

The source code for demo project of this tutorial is available [here](https://github.com/airpair/ng-stripe-tutorial). You can see the actual checkout form in `checkout.html` and the source code for the controller inside `scripts.js`.

### 4.4  Final words

I hope that you enjoyed reading this tutorial, and I wish you many successful Stripe transactions and a lot of success with your project. Until the next time!