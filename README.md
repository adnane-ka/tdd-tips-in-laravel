When creating an application or new functionality, tests can be written in parallel during its development or only at the very end. But In the case of TDD, Test Driven Developement, the opposite is true. First, we write a test for non-existent functionality, and then we write a code that will make our test pass.
Here's what the TDD cycle looks like at a glance:

1. We write a test for non-existent functionality.
2. We run the test, which of course ends with an error. Based on the generated error, we create the minimum code that allows you to pass the test.
3. After passing the test, we develop our code and run the tests again.

In This repo we're not going to talk about how to do TDD in Laravel with phpunit . but instead , we're going to list a curated tips & tricks to do it better . 

---
#### Table of contents : 
* 1. Use sqlite as a Database Connection .
* 2. Set a Memory Limit.
* 3. use the DatabaseMigrations trait.
* 4. Properly name your Tests.
* 5. Use The Appropriate assertions & expectations
* 6. Create tests for each validation rule

---
#### 1. Use sqlite as a Database Connection . 
as it's not highly recommended to use the same Database that your application is using as a Default database connection .
```php
# phpunit.xml :  
<env name="DB_CONNECTION" value="sqlite"/>
<env name="DB_DATABASE" value=":memory:"/>
<env name="API_DEBUG" value="false"/>
```
#### 2. Set a Memory Limit . 
```php 
# phpunit.xml : 
<ini name="memory_limit" value="512M" />
```

#### 3.use the DatabaseMigrations trait:
so in every run of the test, the migration files are also being run. You may also notice that we have the setUp() and tearDown() methods that are needed to complete that cycle of the application during the test.

```php 
namespace Tests;

use Illuminate\Foundation\Testing\DatabaseMigrations;
use Illuminate\Foundation\Testing\DatabaseTransactions;
use Illuminate\Foundation\Testing\TestCase as BaseTestCase;
use Faker\Factory as Faker;

/**
 * Class TestCase
 * @package Tests
 * @runTestsInSeparateProcesses
 * @preserveGlobalState disabled
 */
abstract class TestCase extends BaseTestCase
{
    use CreatesApplication, DatabaseMigrations, DatabaseTransactions;

    protected $faker;

    /**
     * Set up the test
     */
    public function setUp()
    {
        parent::setUp();
        $this->faker = Faker::create();
    }

    /**
     * Reset the migrations
     */
    public function tearDown()
    {
        $this->artisan('migrate:reset');
        parent::tearDown();
    }
}
```

#### 4.Properly name your Tests : 
- I like to follow the ResourceThing naming convention like : ItemTest , PackageTest ,ProductTest and so on .. That's not a rule or an obligation of course , but it's highly recommended that you unify the way you name things .

- Please notice that a test in PHPUNIT is known only if you put ```/** @test */``` annotation on the docblock or ```test_``` as a prefix. 

- It's So much better if The methods Contained in the test had significant and clear names like : (a_product_price_is_required , teachers_list_can_be_retrieved .. ect) , and not like (validate , index) .. 

```php 
namespace Tests\Feature;

class ProductTest extends TestCase
{
    ## this is known 
    /** @test */
    public function a_product_can_be_created()
    {
        $data = [
          'product_name' => $this->faker->sentence,
          'product_description' => $this->faker->paragraph
        ];
      
        $response = $this->postJson(route('product.store'), $data);
    
        $response->assertStatus(200);
    }

    ## this is also known 
    public function test_if_a_product_can_be_created()
    {
        $data = [
          'product_name' => $this->faker->sentence,
          'product_description' => $this->faker->paragraph
        ];
      
        $response = $this->postJson(route('product.store'), $data);
    
        $response->assertStatus(200);
    }
}
```
#### 5. Use The Appropriate assertions & expectations :
For Example , in our previous example . We excpect the code to work following a certain procces . 

so the code behaviour will be much understandable if the application gave us a status of 200 after receiving the sent data. as we may assert that a new row was inserted in DB using ```assertCount```. 

#### 6. Create tests for each validation rule .




 