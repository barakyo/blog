Besides writing test cases just because "everyone else is", we should be writing test cases since they'll help create a stronger trust in our code that our functions are doing what they should be doing.


### Requirements ###
* PHP
* PHPUnit (you will most likely need a global install) 
* Test Cases (DUH)

### Helpers ###
Helpers are a set of classes that we create to take care of common mundane tasks. For example, the User Helper for testing provides functions for returning a user with a specified role or easily create a new user with the specified role if one does not exist or the user requests one.

Since we can define helpers for a number of things, we want to ensure that the helpers for our test case don't conflict with helpers that we use in other applications, for this we'll create our test helpers in the sub-namespace `Testing` within the `Helpers` name space.

PHP Namespaces are defined as the first line after the `<?php` opening. It **MUST** be the first line after the opening, or else PHP will throw an error.

{{< highlight php >}}
<?php

namespace Helpers\Testing;
{{< / highlight >}}


After defining the namespace, you will most likely want to include common exceptions that will be thrown when accessing models.
{{< highlight php >}}

use Illuminate\Database\Eloquent\ModelNotFoundException;

{{< / highlight >}}
From there, you're ready and free to define your helper class as you please. 

**PLEASE COMMENT FUNCTIONS**

### Test Cases ###
To write test cases you must extend from the Laravel's base `TestCase` class. When writing test cases please follow certain conventions:

* The name of the file and the class for the test case should follow the naming convention ControllerFunctionTest
	* Ex: Writing test cases for the Index function for the User Controller: UserControllerIndexTest.php
{{< highlight php >}}
<?php
class UserControllerIndexTest extends TestCase {

}
{{< / highlight >}}
* All functions that are test musted start with the word **test** (PHPUnit convention) or else PHPUnit will not find your function
* Functions should specify what role they're testing as, ex: `testIndexAsAdmin()`

### Tips for Writing Test Cases ###

#### Calling Routes #####
We want to guarantee our that not only our functions are correct, but that our routes/API are calling the correct functions. If you can, please test routes rather than functions. You can call routes using the `$this->call()` function. More information for calling routes can be found in [Laravel's Documentation](http://laravel.com/docs/testing#calling-routes-from-tests).

#### Acting as a User ####
To act as a user, you can use the `$this->be()` function, which can also be found in [Laravel's Documentation](http://laravel.com/docs/testing#helper-methods)

#### Specifying Exceptons ####
An exception being throw can be a success if a user should not have access to certain functionality. Since our API immediately throws a InsufficientPermissionsException when a user doesn't have the right permissions, we can easily have our test cases catch this and mark them as a passed test. To specify that an exception should be thrown, you can define it in the PHPDocs comments like so:
    {{< highlight php >}}
<?php
    class UserControllerIndexTest extends TestCase {
    	/**
         * Tests user index function as student, passes on thrown exception
         * @expectedException InsufficientPermissionsException
        */
        public function testIndexAsStudent() {
            $user = $this->userHelper->getStudentUser();
            $this->be($user);
            $this->call("GET", $this->base_route);
        }
    }
{{< / highlight >}}
#### Writing Functions ####
**Note:** Only one test per function, so you cannot have a function return users with all roles and test them as one test case. Hence the need for the naming convention of what you're testing as what role.

Test cases can become repetitive, to ease the amount of typing you can write a function (*GASP!*) which we can call in our <del>all</del> most  of our tests. 

### References ###
* [Laravel Documentation on Testing](http://laravel.com/docs/testing)
* [PHPUnit Documentation](http://phpunit.de/manual/current/en/index.html)

