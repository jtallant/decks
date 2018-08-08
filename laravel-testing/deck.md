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

