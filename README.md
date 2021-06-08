When creating an application or new functionality, tests can be written in parallel during its development or only at the very end. But In the case of TDD, Test Driven Developement, the opposite is true. First, we write a test for non-existent functionality, and then we write a code that will make our test pass.
Here's what the TDD cycle looks like at a glance:

1. We write a test for non-existent functionality.
2. We run the test, which of course ends with an error. Based on the generated error, we create the minimum code that allows you to pass the test.
3. After passing the test, we develop our code and run the tests again.

so here is a guidline to better do TDD with Laravel : 
### 1. configure & prepare the configuration file as Bellow : 
```php 
<env name="DB_CONNECTION" value="sqlite"/>
<env name="DB_DATABASE" value=":memory:"/>
<env name="API_DEBUG" value="false"/>
<ini name="memory_limit" value="512M" />
```

### 2.Make sure your base TestCase if prep for it:
- You need to add the DatabaseMigrations trait, so in every run of the test, the migration files are also being run. You may also notice that we have the setUp() and tearDown() methods that are needed to complete that cycle of the application during the test.

### 3.Create An Actual Test : 
- you can use the artisan command ```php artisan make:test TestName``` to create new Feature tests in laravel .

- I like to follow the ResourceThing Naming convention like : ItemTest , PackageTest ,ProductTest and so on .. 

- A Test in PHPUNIT is known only if you put ```/** @test */``` annotation on the docblock or ```test_``` as a prefix. examples : 

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

### 4. Use The Appropriate assertions & expectations :
For Example , in our previous example . We excpect the code to work following a certain procces . 

so the code behaviour will be much understandable if the application gave us a status of 200 after receiving the sent data. as we may assert that a new row was inserted in DB using ```assertCount```. 

### 5. Start Testing ! 
At That point , you can start the TDD proccess . as you can run the command : 
```
phpunit.xml
```

or 

```
vendor/bin/phpunit.xml
```

you may start noticing some failures like: 

```
was expecting 200 but received 404 
```

and That's because you have not define any routes or controllers.
After You create them an error may start appearing : 

```
was expecting 200 but received 500 
```
as this means you need to debug your controller and so on ..

Notice That Now you're driven by the Tests results to develop and write functionnality . you wrote a test for non-existent functionality, and then you wrote a code that will make your test pass. And that's the point when it comes to TDD ! 

This may sound silly at some point , but at a deeper level it learns you how to understand the client's needs & understand a better Use Cases for your software 



 