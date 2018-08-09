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

* Arrange: Setup/Instantiate objects and pass data
* Act: Run the code that we want to test the outcome of
* Assert: Verify expected outcome

---
```php
class ViewConcertListingTest extends TestCase
{
    /**
     * @test
     */
    public function user_can_view_concert_listing()
    {
        // Arrange
        $concert = Concert::create($attributes);

        // Act: View the listing in browser
        $this->visit('/concerts/'.$concert->id);

        // Assert
        $this->see('Some concert attribute');
        $this->see('Some other concert attribute');
    }
}
```

---

# Tip

You don't have to worry about CSS while testing.

Just test that the information is displayed on the page.

Some people make an HTML mockup with hardcoded data or the template is already prepared for the backend dev styled and coded out in raw html by a designer and front end dev. I actually really like that way of working because the backend dev knows exactly what data they need, they have less to think about, and all they have to do is make the page dynamic.

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
# create one user
$user = factory(App\User::class)->make();

# create 3 users
$users = factory(App\User::class, 3)->make();
```

---

### Unit Testing Presentation Logic

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

