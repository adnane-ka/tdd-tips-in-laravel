### Test Driven Developement in Laravel Using PHPunit , Tricks & Tips

---
#### Table of contents : 
* [Overview](#overview)
* [Proccess](#proccess)
* [Why TDD ?](#why-tdd-)
* [Important Note](#important-note)
* [Guideline](#guideline)
    * 1.[Use SQLite as a Database Connection](#1-use-sqlite-as-a-database-connection-)
    * 2.[Set a Memory Limit](#2-set-a-memory-limit-)
    * 3.[Use the DatabaseMigrations trait](#3use-the-databasemigrations-trait)
    * 4.[Properly name your Tests](#4properly-name-your-tests-)
    * 5.[Use The Appropriate assertions & expectations](#5-use-the-appropriate-assertions--expectations-)
    * 6.[Create tests for each validation rule](#6-create-tests-for-each-validation-rule-)
    * 7.[Don’t create a full instance of the Laravel application unless you have to](#7-dont-create-a-full-instance-of-the-laravel-application-unless-you-have-to-)
    * 8.[Use the new Refresh Database trait](#8-use-the-new-refresh-database-trait-)
    * 9.[Mock what you can’t control](#9-mock-what-you-cant-control-)
    * 10.[Use this instead of digging deeper in code](#10-use-this-instead-of-digging-deeper-in-code--)

---
## Overview 

When creating an application or new functionality, tests can be written in parallel during the development proccess or only at the very end. But when it comes to TDD, Test Driven Developement, the opposite becomes true. First, we write a test for a non-existent functionality, and then we write a code that will make our test pass.


notice that each functionality was a result of a test , and each test represents a single expected use case . so the more you understood your client's needs the more you wrote a useful test . so you won't be worrying about why you have added some unuseful features ..

---
## Proccess 
Here's what the TDD cycle looks like at a glance:

1. We write a test for a non-existent functionality.
2. We run the test, which of course ends up with an error. Based on the generated error, we create the minimum code that allows us to pass the test successfly.
3. After passing the test, we develop our code and run the tests again.

---
## Why TDD ?
if you're not very familiar with that term here's some advantages that you will be having whenever you're doing TDD : 

1. TDD helps avoid errors and bugs when introducing new features in an application
2. Project processes are made easier to track in a test-driven environment
3. TDD helps ease the process of documenting software
4. TDD inspires confidence when writing code
5. Bugs are easier to spot and fix in an environment where tests are written.

---
## Important Note 
In This repo we're not going to talk about how to do TDD in Laravel with phpunit . but instead , we're going to list a curated tips & tricks to do it in a better way ! 

---
## Guideline

#### 1. Use SQLite as a Database Connection . 
as it's not highly recommended to use the same Database that your application is using as a Default database connection .

Utilizing an in-memory SQLite database is another way to increase the speed of your tests that hit the database. You can get started with this quickly by adding these two environment keys to your phpunit.xml configuration.
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

#### 3.Use the DatabaseMigrations trait:
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




#### 7. Don’t create a full instance of the Laravel application unless you have to :
Not every test requires that you instantiate the full Laravel application, and doing so slows your tests down. If you don’t absolutely need the full application instantiated in the test, consider having your test inherit from the below simple test case class instead:

```php 
<?php
namespace Tests;

use Mockery\Adapter\Phpunit\MockeryPHPUnitIntegration;
use PHPUnit\Framework\TestCase as BaseTestCase;

class SimpleTestCase extends BaseTestCase
{
    use MockeryPHPUnitIntegration;
}
```

#### 8. Use the new Refresh Database trait :
This testing trait is generally more efficient than migrating down and up, because it empties the database afterwards rather than stepping through the changes of each migration. If you have a non-trivial number of migrations, it will almost certainly be quicker than migrating down, then back up for the next test.

```php 
use RefreshDatabase;
```

#### 9. Mock what you can’t control .
You should never, ever be making calls to external APIs in your test suite, because you can’t control whether those external API’s work - if a third-party API goes down, you may get a failed test run even if your application is working perfectly, not to mention it will add the time taken to send the request and receive a response to the test time. Instead, mock the calls to the third-party API.

#### 10. Use this instead of digging deeper in code  .
``` $this->withoutExceptionHandling() ``` tells PHPUnit not to handle exceptions that we may get. This is to disable Laravel’s exception handling to prevent Laravel from handling exceptions that occur instead of throwing it, we do this so we can get a more detailed error reporting in our test output.

```php 
public function a_test_that_went_wrong()
{
    $this->withoutExceptionHandling();


    // some testing code 
}
```

