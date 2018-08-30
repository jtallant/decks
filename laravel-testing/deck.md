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

# Updating test code

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