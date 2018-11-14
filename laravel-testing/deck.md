# Laravel Testing Notes
## @jtallant

---

# Programming by Wishful Thinking

We will follow this philosophy all throughout.

Just write the code/method names you wish you already had
and then go and make the code actually functional.

[Link - Progamming by Wishful Thinking](https://dev.to/_bigblind/quick-tip-programming-by-wishful-thinking-3hn)

---

# Configure Test DB in Laravel
1. Create a 'testing' DB connection in config/database.php
1. Set PHPUnit env DB_CONNECTION to testing

---

# Create new DB connection for testing

```php
// config/database.php
[
    'testing' => [
        'driver' => 'sqlite',
        'database' => ':memory',
    ],
];
```

---

# Set PHPUnit env DB_CONNECTION to testing
### phpunit.xml

```xml
<php>
    <env name="DB_CONNECTION" value="testing">
</php>
```

---

# Configure Test DB in Laravel

* Now when we run phpunit, it will using our new DB connection called "testing"
* The testing connection utilizes an in memory sqlite database for now
* Later we may swap to an actual mysql DB used strictly for testing

---

# Choosing the first feature

- Don't get caught in the weeds!
- Write and test the feature most important to the business first
- You can integrate the core feature in with the other minor features later

---

# Arrange, Act, Assert

- **Arrange**: Setup/Instantiate objects and pass data
- **Act**: Run the code that we want to test the outcome of
- **Assert**: Verify expected outcome

---
```php
class ViewConcertListingTest extends TestCase
{
    /**
     * @test
     */
    public function user_can_view_concert_listing()
    {
        # Arrange
        $concert = Concert::create($attributes);

        # Act: Make request to concert show
        $response = $this->get('/concerts/'.$concert->id);

        # Assert: Ensure concert data exists in the response body
        $response->assertSee('Some concert attribute');
        $response->assertSee('Some other concert attribute');
    }
}
```

---

# Tip

You don't have to worry about CSS while testing.

Just test that the information is displayed on the page.

Some people make an HTML mockup with hardcoded data or the template is already prepared for the backend dev styled and coded out in raw html by a designer and front end dev. I actually really like that way of working because the backend dev knows exactly what data they need, they have less to think about, and all they have to do is make the page dynamic.

---

# Tip

When working alone, you may decide to code all of your templates out in raw HTML with fake data, and do all the CSS first, or you may choose to drive out the backend implentation and do the design later with the functionality already in place.

---

# Model Factories

## The Problem 

When writing our tests, we need to construct an environment filled with objects that have data. This way we can simulate how the application works in production. Constructing objects for test purposes and filling them up with fake data that simulates real data is tedious and time consuming.

---

# Model Factories

Manually instantiating objects and typing out all the values with realistic data sucks! Model factories to the rescue!

```php
# instead of this in every test method...
$user = User::create([
    'name' => 'Jane Doe',
    'email' => 'jane@example.com',
    'password' => bcrypt('some-password'),
    'remember_token' => str_random(10)
]);

# we can just do this...
$user = factory(App\User::class)->make();
```

---

# Model Factories

Model factories act as **factories** for the objects in your application. They take care of constructing objects for you and filling their properties with values that simulate real world data.

### Factories are essentially templates for objects

---

# Defining a Factory

```php
use Faker\Generator as Faker;

$factory->define(App\User::class, function (Faker $faker) {
    return [
        'name'           => $faker->name,
        'email'          => $faker->unique()->safeEmail,
        'password'       => $secret,
        'remember_token' => str_random(10),
    ];
});
```

---

# Using a Factory

```php
# create one user instance, do not persist to DB (make vs create)
$user = factory(App\User::class)->make();

# create 3 user instances, do not persist to DB (make vs create)
$users = factory(App\User::class, 3)->make();
```

---
```php
// tests/Unit/ConcertTest.php
class ConcertTest extends TestCase
{
    /**
     * @test
     */
    public function can_get_formatted_date()
    {
        # Arrange: Create a concert with a known date
        $concert = Concert::create([
            'date' => Carbon::parse('2018-12-01 8:00pm')
        ]);

        # Act: Retrieve the formatted date
        $date = $concert->formatted_date;

        # Assert: Verify the data is formatted as expected
        $this->assertEquals('December 1, 2018', $date);
    }
}

// app/Concert.php
class Concert extends Model
{
    # This special property specifies which columns
    # should be mutated to Carbon instance.
    protected $dates = ['created_at', 'updated_at', 'date'];

    /**
     * Create a formatted_date attribute on the model
     * $this->formatted_date is now accessible
     * 
     * @return string
     */
    public function getFormattedDateAttribute()
    {
        return $this->date->format('F j, Y');
    }
}
```

---

# Resetting The Database After Each Test

> It is often useful to reset your database after each test so that data from a previous test does not interfere with subsequent tests
-- Laravel Docs

---

# Resetting The Database After Each Test

In other words we have to explicitly define the state of the database to ensure the "expected" part of our testing assertions are reliable.

---

# RefreshDatabase Trait

> The RefreshDatabase trait takes the most optimal approach to migrating your test database depending on if you are using an in-memory database or a traditional database. Use the trait on your test class and everything will be handled for you
-- Laravel Docs

---

# RefreshDatabase Trait

```php
use Tests\TestCase;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Foundation\Testing\WithoutMiddleware;

class ExampleTest extends TestCase
{
    use RefreshDatabase;

    public function testBasicExample()
    {
        $response = $this->get('/');

        // ...
    }
}
```

---

# Relative Test Data

If you are truncating or recreating the database before each test and populating it with fresh data everytime, then it makes sense to use "relative data" in some circumstances.

Let's say you need to make sure concert tickets aren't still for sale after the concert is over.

---

# Relative Test Data

```php
# Instantiate a concert that was yesterday using relative date
$concert = Concert::make(['date' => Carbon::parse('-1 day')]);

# Some service class that has a method
# for comparing concert start date to today
# It probably also checks to see if tickets are sold out
# so in reality you would also need to instantiate some tickets
# and relate them to the concert.
$ticketManager = new TicketManager;

$this->assertEquals('closed', $ticketManager->ticketStatus($concert));
```

---

# Make vs Create (Eloquent and Factories)

```php
# Instantiate a model instance w/o persisting to DB
Model::make($attributes);

# Instantiate and persist to DB
Model::create($attributes);

# Use factory to create instance w/o persisting
factory(Model::class)->make();

# Use factory to create model instance and persist
factory(Model::class)->create();

# Use factory to create 4 model instances and persist
factory(Model::class, 4)->create();
```

---

# Increasing Test Speed with Make

If you can properly test your code and make your assertions without persisting to the database or refreshing the database at all, then just use make.

If this is the case, then remove the `RefreshDatabase` trait and swap any calls to create with make.

---

# Laravel Assertions

* Laravel provides many assertions on top of the built in PHPUnit assertions
* These assertions are very useful when building a web application
* PHPUnit doesn't know or care that you are building a web app
* The only function of PHPUnit is to test code
* PHPUnit knows nothing about HTTP

---

# Laravel Assertions

There are 3 main categories of additional assertions that Laravel provides.

* HTTP Assertions
* Browser Assertions
* Database Assertions

---

# HTTP Assertions

Never use **see()** `$this->see($text)` to assert response statuses.

Instead use the proper HTTP verb.

```php
$this->get('/users/1');
$this->post('/users');
$this->put('/users/1');
$this->delete('/users/1');
```
---

# Special JSON HTTP method

The JSON method is useful when the route expects json and returns json.

```php
$this->json('POST', '/user', ['name' => 'Sally']);
```

---

# Special JSON HTTP method

There is nothing magical about the json method. It simply does some grunt work for you like setting headers and json encoding the request data. Under the hood...

```php
/**
 * Call the given URI with a JSON request.
 *
 * @param  string  $method
 * @param  string  $uri
 * @param  array  $data
 * @param  array  $headers
 * @return \Illuminate\Foundation\Testing\TestResponse
 */
public function json($method, $uri, array $data = [], array $headers = [])
{
    $files = $this->extractFilesFromDataArray($data);

    $content = json_encode($data);

    $headers = array_merge([
        'CONTENT_LENGTH' => mb_strlen($content, '8bit'),
        'CONTENT_TYPE' => 'application/json',
        'Accept' => 'application/json',
    ], $headers);

    return $this->call(
        $method, $uri, [], [], $files, $this->transformHeadersToServerVars($headers), $content
    );
}
```

---

# JSON Method Example

```php
class ExampleTest extends TestCase
{
    /**
     * @test
     */
    public function test_user_create()
    {
        $response = $this->json('POST', '/user', ['name' => 'Sally']);

        $response
            # assert we received 201 created HTTP status
            ->assertStatus(201)
            # assert the json response matches the exact data we pass here
            ->assertExactJson([
                'created' => true,
            ]);
    }
}
```

---

# Asserting Response Status

```php
class ExampleTest extends TestCase
{
    public function testBasicExample()
    {
        $response = $this->get('/some/uri/that/should/404');    

        # Assert we received 404 status (method 1)
        $response->assertNotFound();

        # Assert we received 404 status (method 2)
        $response->assertStatus(404);
    }
}
```

---

# Testing Query Scopes

```php
# Definition
class Concert extends Model
{
    public function scopePublished($query)
    {
        return $query->whereNotNull('published_at');
    }
}

# Usage
$publishedConcerts = Concert::published();
```

---

# Testing Query Scopes

```php
/** @test */
function concerts_with_a_published_at_date_are_published()
{
    $publishedConcertA = factory(Concert::class)
        ->create(['published_at' => Carbon::parse('-1 week')]);

    $publishedConcertB = factory(Concert::class)
        ->create(['published_at' => Carbon::parse('-1 week')]);

    $unpublishedConcert = factory(Concert::class)
        ->create(['published_at' => null]);

    $publishedConcerts = Concert::published()->get();

    $this->assertTrue($publishedConcerts->contains($publishedConcertA));
    $this->assertTrue($publishedConcerts->contains($publishedConcertB));
    $this->assertFalse($publishedConcerts->contains($unpublishedConcert));
}
```

---

## Hiding an implementation detail with more expressive code

* By looking at the code we can see that the published_at attribute value determines whether or not the concert is published. If the value is not null, it is published, else it is not published.
* We can be more expressive and not leak that information out in our tests.
* Factory states are one way of being more expressive.

---

## Factory States

```php
# Define the Concert factory
$factory->define(Concert::class, function (Faker $faker) {
    return [
        // default attributes here
    ];
});

# Add a factory "state"
$factory->state(Concert::class, 'published', [
    'published_at' => Carbon::parse('-1 day'),
]);

# Add an unpublished state just for clarity (being explicit)
$factory->state(Concert::class, 'unpublished', [
    'published_at' => null,
]);

# Use the factory
$publishedConcert = factory(Concert::class)->states('published')->make();

# Make unpublished concert
$unpublishedConcert = factory(Concert::class)->states('unpublished')->make();

```

---

# Updating Test Code

```php
/** @test */
function concerts_with_a_published_at_date_are_published()
{
    $publishedConcertA = factory(Concert::class)
        ->create(['published_at' => Carbon::parse('-1 week')]);

    $publishedConcertB = factory(Concert::class)
        ->create(['published_at' => Carbon::parse('-1 week')]);

    $unpublishedConcert = factory(Concert::class)
        ->create(['published_at' => null]);

    $publishedConcerts = Concert::published()->get();

    $this->assertTrue($publishedConcerts->contains($publishedConcertA));
    $this->assertTrue($publishedConcerts->contains($publishedConcertB));
    $this->assertFalse($publishedConcerts->contains($unpublishedConcert));
}
```

```php
/** @test */
function concerts_with_a_published_at_date_are_published()
{
    $publishedConcertA = factory(Concert::class)->states('published')->create();

    $publishedConcertB = factory(Concert::class)->states('publised')->create();

    $unpublishedConcert = factory(Concert::class)->states('unpublished')->create();

    $publishedConcerts = Concert::published()->get();

    $this->assertTrue($publishedConcerts->contains($publishedConcertA));
    $this->assertTrue($publishedConcerts->contains($publishedConcertB));
    $this->assertFalse($publishedConcerts->contains($unpublishedConcert));
}
```

---

# Browser Testing vs Endpoint Testing

---

## Browser Testing

* Using a tool like selenium or PhantomJS to simulate a user's actions
inside the browser.

----

```php
function testBasicExample()
{
    $user = factory(User::class)->create([
        'email' => 'taylor@laravel.com',
    ]);

    $this->browse(function ($browser) use ($user) {
        $browser->visit('/login')
                ->type('email', $user->email)
                ->type('password', 'secret')
                ->press('Login')
                ->assertPathIs('/home');
    });
}
```

---

### Pros
* High confidence. Simulates exactly how a user would interact with app.
* Verifies working end-to-end (client => server)

### Cons
* Introduct new tool to stack (Selenium)
* Slower
* Brittle (depends on UI)
* Complex setup
* Often can't interact with code directly, have to make assertions through UI.

---

## Endpoint Testing (HTTP Tests)

* Make HTTP requests directly to an endpoint, simulating how the browser would interact with the server instead of how user interacts with app.

```php
function testBasicExample() {
    $response = $this->get('/');
    $response->assertStatus(200);
}
```

---

## Endpoint Testing (HTTP Tests)

### Pros
* Faster
* Doesn't require additional tooling
* Interacts with more stable data structures (DOM doesn't matter)
* Can interact directly with code, more flexible assertions

### Cons
* Untested gap between front-end and back-end

---

# What do we need from our tests?
* Confidence that the system works. This is #1!!
* Reliable, don't break for unimportant reasons
* Fast, run often
* Simple, limit tools used, easily recreate the env

---

# Faking Payment Gateway

* Sometimes you have a package that depends on some external service
* Stripe is a package that interacts with stripe.com API for billing
* Don't use the actual package in your tests
* Otherwise you're testing stripe and your code, not just your code
* You'd have to make actual requests to Stripe (requires internet, slow)

---

# Faking Payment Gateway

So how do we test our code without using the actual payment gateway that we will have to depend on?

---

# Faking Payment Gateway

* For now we just create a fake payment gateway
* **only implement what you need right now**
* Don't just go writing all the methods you think you'll need

```php
class FakePaymentGateway
{
    proteced $charges;

    public function __construct()
    {
        $this->charges = collect();
    }

    public function getValidTestToken()
    {
        return 'valid-token';
    }

    public function charge($amount, $token)
    {
        $this->charges[] = $amount; 
    }

    public function totalCharges()
    {
        $this->charges->sum();
    }
}
```

---

# Faking Payment Gateway

* Now that we have a fake, we need to use it on our controller so we can actually test our code
* Instead of just injecting the fake we should inject an interface so we can swap it with a real payment gateway in production

```php
use App\Billing\PaymentGatewayInterface;

class ConcertOrdersController
{
    /**
     * @var PaymentGatewayInterface
     */
    protected $paymentGateway;

    public function __construct(PaymentGatewayInterface $paymentGateway)
    {
        $this->paymentGateway = $paymentGateway;
    }
}
```

---

# Faking Payment Gateway

* Now that we have our interface, laravel will throw an error saying that the interface is not instantiable. That is because interfaces are of course, not instantiable
* We need to tell Laravel which implementation of the interface to use

```php
// tests/features/PurchaseTickets.php
class PurchaseTicketsTest extends TestCase
{
    /** @test */
    function customer_can_purchase_tickets()
    {
        $paymentGateway = new FakePaymentGateway;

        # Tell Laravel...
        # When I ask for PaymentGatewayInterface, give me fake payment gateway
        # In production, we'll bind the actual stripe payment gateway
        $this->app->instance(PaymentGatewayInterface::class, $paymentGateway);
    }
}
```

---

# Encapsulting Relationship Logic (Refactor)

```php
class ConcertOrdersController
{
    public function store($id)
    {
        $concert = Concert::findOrFail($id);

        // Charging the customer
        $ticketQuantity = request('ticket_quantity');
        $amount = $ticketQuantity * $concert->ticketPrice;
        $token = request('payment_token');
        $this->paymentGateway->charge($amount, $token);

        // Creating the order
        $order = $concert->orders->create(['email' => request('email')]);

        foreach(range(1, $ticketQuantity) as $i) {
            $order->tickets()->create([]);
        }

        return response()->json([], 201);
    }
}
```

---

# Encapsulting Relationship Logic (Refactor)

```php
class ConcertOrdersController
{
    public function store($id)
    {
        $concert = Concert::findOrFail($id);

        $tq = request('ticket_quantity');

        // Charging the customer
        $this->paymentGateway->charge($tq * $concert->ticket_price, request('payment_token'));

        // Creating the order
        $order = $concert->orders->create(['email' => request('email')]);

        foreach(range(1, $tq) as $i) {
            $order->tickets()->create([]);
        }

        return response()->json([], 201);
    }
}
```

---

# Encapsulting Relationship Logic (Refactor)

```php
class ConcertOrdersController
{
    public function store($id)
    {
        $concert = Concert::findOrFail($id);
        
        $tq = request('ticket_quantity');

        # Charging the customer
        $this->paymentGateway->charge($tq * $concert->ticket_price, request('payment_token'));

        # Creating the order
        $order = $concert->orderTickets($email, $tq);
        // $order = $concert->orders->create(['email' => request('email')]);
        // foreach(range(1, $tq) as $i) {
        //     $order->tickets()->create([]);
        // }


        return response()->json([], 201);
    }
}
```

---

# Refactor

```php
// use unit test to verify new orderTickets method workds
class ConcertTest extends TestCase
{
    /** @test */
    function can_order_concert_tickets()
    {
        $concert = factory(Concert::class)->create();

        $order = $concert->orderTickets('jane@example.com', 3);

        $this->assertEquals('jane@example.com', $order->email);
        $this->assertEquals(3, $order->tickets()->count());
    }
}

class Concert extends Model
{
    public function orderTickets($email, $ticketQuantity)
    {
        $order = $this->orders->create(['email' => $email]);

        foreach(range(1, $ticketQuantity) as $i) {
            $order->tickets()->create([]);
        }

        return $order;
    }
}
```

---

# Encapsulting Relationship Logic (Refactor)

```php
class ConcertOrdersController
{
    public function store($id)
    {
        $concert = Concert::findOrFail($id);
        
        # Charging the customer
        $this->paymentGateway->charge(request('ticket_quantity') * $concert->ticket_price, request('payment_token'));

        # Creating the order
        $order = $concert->orderTickets(request('email'), request('ticket_quantity'));

        return response()->json([], 201);
    }
}
```

---

# Detour

---

# Laravel Exception Handling

* Laravel converts model not found exceptions to 404
* Laravel converts validation errors to 302 redirect back
* Model not found is triggered when you call Model::findOrFail($id) and the model is not found
* When this happens, a ModelNotFoundException is thrown
* When exception handling is on, the laravel exception handler will convert that ModelNotFoundException into a 404 response to the client

---

Benefit of Laravel exception handler converting the exception to response...

```php
class SomeController
{
    // do this...
    function show($id)
    {
        $model = Model::findOrFail($id);

        return view('some.view', ['model' => $model]);
    }

    // instead of this...
    function show($id)
    {
        $model = Model::find($id);

        if (empty($model)) {
            abort(404);
        }

        return view('some.view', ['model' => $model]);
    }
}
```

---

# Exception Handling

* As convenient as all that is in production, it's not convenient when we are testing. While testing, we usually want the original exception that was thrown.
* The exception handler can make it very confusing as to what is actually going on or what the error is.

```php
class SomeTest extends TestCase
{
    /** @test */
    function it_does_a_thing()
    {
        $this->withoutExceptionHandling();

        // now ModelNotFoundException is actually thrown
        // now validation errors don't give 302, ValidationException is thrown
    }
}
```

---

# Testing Validation Logic

```php
class HabitsController extends Controller
{
    public function store(Request $request)
    {
        $data = $request->validate([
            'title'      => 'required|string',
            'start_date' => 'required',
        ]);

        $habit = Habit::create([
            'title'      => request('title'),
            'start_date' => request('start_date'),
        ]);

        return redirect()->route('habits.index');
    }
}

class CreateHabitTest extends TestCase
{
    /** @test */
    public function it_validates_attributes()
    {
        $user = factory(\App\User::class)->create();

        $response = $this->post('/habits', [
            'title' => null,
            'start_date' => null,
        ]);

        $response->assertSessionHasErrors(['title', 'start_date']);

        $response->assertStatus(302); // redirect back
    }
}
```

---

# Reducing Duplication With Custom Assertions

* Being lazy on notes here
* Basically try and remove duplication and create private methods on test class when it makes sense

---

# Asserting Exceptions
^ Handling failed charges

* phpunit supports docblock for expected exception
* this is not our preferred method

```php
/**
 * @test
 * @expectedException \App\Billing\PaymentFailedException
 */
function it_fails_without_a_valid_payment_token()
{
    // code that attemts to charge w/o payment token here
}
```

---

# Assert Exceptions

* A better way

```php
/** @test */
function it_fails_without_a_valid_payment_token()
{
    try {
        $paymentGateway = new PaymentGateway;
        $paymentGateway->charge(2500, 'invalid-payment-token');
    } catch (PaymentFailedException $e) {
        // we just return out of the try block
        // so we can assert the test failed below
        return;
    }

    $this->fail();
}
```

---

# Why this approach?

* It allows us to make assertions about the failed charge if need be
* This is done in the catch part of the try block

```php
/** @test */
function it_fails_without_a_valid_payment_token()
{
    try {
        $paymentGateway = new PaymentGateway;
        $paymentGateway->charge(2500, 'invalid-payment-token');
    } catch (PaymentFailedException $e) {
        $this->assertEquals(2500, $e->failedChargeAmount());
        return;
    }

    // some other assertions here
}
```

---

# Preventing Ticket Purchases for Unpublished Concerts

```
// Assert response status 404 in the test, should not be a valid endpoint
response = make request to order tickets on unpublished concert
response->assertStatus(404)

// Assert no orders were created for the concert just to be thorough
$this->assertEquals(0, $concert->orders()->count());

// Assert customer isn't actually charged!!!
$this->assertEquals(0, $paymentGateway->totalCharges());
```


---

# Clean Up Tests

* Tests need to be refactored and kept healthy just like production code
* Look for duplication as usual

---

# Asserting Against JSON

* `assertJson`
    - verify partial match
    - PHPUnit::assertArraySubset
    - test passes if other key/values exist in response
* `assertExactJson`
    - verify all response key/values match provided key/values

---

# Asserting Against JSON

* Most the time you're just going to be using `assertJson`
* You're not gonna wanna test the whole response in every test

---

# Filtering Tests

* It's often useful to run only one test class or even one test method at a time

---

# Testing one class

* Just type the whole path to the file
* phpunit /path/to/my/test/class

---

# Testing one method

* You can use the `--filter` option in phpunit
* phpunit /path/to/test/class --filter target_test_method_name

---

# This Design Sucks

* Nothing interesting here
* Just obvious design flaws

---

# Persisting the Order Amount

* Only make changes while the tests are passing
* I assume this helps avoid getting into a hole
* Find a way to change the code that keeps your tests passing

---

# Removing the need to cancel orders

* When starting changes, find a test to use as a reference point to ensure the code still works.

---

# Preparing for Extraction

* Removes a foreign key column in favor of belongsToMany

---

# Extracting a Named Constructor

* `Order::forTickets($tickets, string $email)`

---

# Precomputing the Order Amount

Adds an order 'amount' param to the `Order::forTickets` named constructor.

* Makes the param optional
* updates all calls to forTickets to provide the param
* Makes the param required

---

# Uncovering a New Domain Object

```php
# Old
$this->payments->charge($tickets->sum('price'));
$order = Order::forTickets($tickets, $email, $tickets->sum('price'));

# New
$reservation = new Reservation($tickets);
$this->payments->charge($reservation->totalCost());
$order = Order::forTicket($tickets, $email, $reservation->totalCost());
```

---

# What I would do

```
// Hold the tickets until order complete or failed
$this->reservations->reserve($tickets);

try {
    // Attempt to charge the customer
    $this->payments->charge($order->amount);

    // success, create order
    $order = new Order($user, $tickets);
} catch (\Exception $e) {
    // failure, lift the reserverations
    $this->reservations->lift($tickets);
    // return error message
}
```

---

# You Might Not Need a Mocking Framework

The reservation class calculates the total cost of the tickets without having to store them in the DB. We can now test the totalCost functionality without persisting models. Domain objects can help simplify our code by reducing dependencies. A side effect of this can be faster and simpler tests.

----

# Tip

You can use plain objects to avoid database inserts.

```php
$tickets = collect([
    (object) ['price' => 1200],
    (object) ['price' => 1200],
]);

$reservation = new Reservation($tickets);

$this->assertEquals(2400, $reservation->totalCost());
```

---

# Uh Oh, a Race Condition

Just explains the race condition for purchasing tickets.

---

# Requestception

Talks about how you would assert a customer can't order tickets that someone else already has reserved.

Why couldn't you just do this?

```php
if ($ticketCollection->hasReservedTickets()) {
    // redirect back, "Some of your ticket selections have already been reserved"
}

$ticketsCollection->reserveAll();

if ($this->paymentGateway->charge($ticketCollection)) {
    // success! create order
} {
    // fail
    $ticketCollection->liftReservation();
}
```

---

# Hooking into Charges

Creates a method inside the the payment gateway that allows you to hook into the gateway and call a function before the first charge.

```php
function beforeFirstCharge(callable $callback)
{
    $this->beforeFirstChargeCallback = $callback;
}

function charge($amount)
{
    $this->beforeFirstChargeCallback->__invoke();

    // you could also do this
    // $beforeFirstCharge = $this->beforeFirstChargeCallback;
    // $beforeFirstCharge();
}
```

---

# Uh oh a segfault

...

---

# Replicating the Failure at the Unit Level

Gotcha..

When nested inside request, Laravel will catch a PHPUnit test failure
and try to convert it into an HTTP error response.

---

# Reserving Individual Tickets

When writing factories, always wrap attributes that persist a new record in an anonymous function.

```php
$factory->define(Ticket::class, function($faker) {
    return [
        'concert_id' => function() {
            return factory(Ticket::class)->create()->id;
        }
    ];
});
```

---

# Reserved Means Reserved

* can_reserve_available_tickets
* cannot_reserve_tickets_that_have_already_been_purchased
* cannot_reserve_tickets_that_have_already_been_reserved

---

# That Guy Stole My Tickets

Explains how to preserve request data when using sub requests within laravel tests. I'm just going to find a way to do the tests properly without using subrequests, seems to messy.

---

# Section: Hunting for Stale Code

---

# Cancelling Reservations

---

