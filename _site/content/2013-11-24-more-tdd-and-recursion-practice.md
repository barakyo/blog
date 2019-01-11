It seems that Javascript challenges are more frequent at work. Thursday afternoon when we got back from lunch, my coworker was testing himself in the [You Can't Javascript Under Pressure](http://toys.usvsth3m.com/javascript-under-pressure/) challenges. As great coworkers, we (by that I mean, me) decided to help out and put more pressure on our fellow coworker, without even being asked to! One of the challenges that came up during the test was a function `ArraySum()` which accepts a list (array) of values and you must total all the integers within the array. The array could contain strings, numbers, booleans, lists of strings, numbers, booleans, etc. Regardless of the input, you must total all the integers within the array. After seeing that challenge (and helping with the correct answer), I thought it might be good practice for another blog post.

### Practice makes Perfect ###
Before jumping into more testing and programming, I want to stop and think about my methodology. While re-reading [Chapter 3](http://chimera.labs.oreilly.com/books/1234000000754/ch03.html) of [Test-Driven Web Development with Python](http://chimera.labs.oreilly.com/books/1234000000754/index.html) on Friday with a coworker, I realized that almost every line of code that was written in the chapter to satisfy the test, had a test case as a reason to write that code. After seeing this and thinking about my previous blog post, I realized I didn't follow the TDD methodology as closely as I <del>could</del>should have. According to the author, Harry Percival, TDD is a discipline, it even says so in chapter 4.

> TDD is a discipline, and that means it's not something that comes naturally; because many of the payoffs aren't immediate but only come in the longer term, you have to force yourself to do it in the moment.

To cut myself some slack, TDD is something you have to practice. Yes, I'm talking about [practice](http://www.youtube.com/watch?v=eGDBR2L5kzI). I'm sure there are a bunch of young grasshoper learning quotes I could state right now, but lets get on to testing.

### Stricter Test Cases ###
Last time we started by importing our function and writing a basic test case to test the return type. Let's do that again.

{{< highlight python >}}
import unittest
from arraysum import ArraySum

class ArraySumTests(unittest.TestCase):
    def setUp(self):
        pass

    def test_for_int(self):
        result = ArraySum([1, 2, 3, 4, 5,])
        self.assertTrue(type(result) is int, "Result is not integer")

if __name__ == '__main__':
    unittest.main()
{{< / highlight >}}

Looks good to me, lets run it!
{{< highlight python >}}
Traceback (most recent call last):
File "tests.py", line 2, in <module>
  from arraysum import ArraySum
  ImportError: No module named arraysum
{{< / highlight >}}

Our first error! Lets correct it, but remember, minimal amount of code! In fact, we won't right any code at all.

{{< highlight bash >}}
caster:arraysum/ $ touch arraysum.py
{% endhighlight bash %} 

Let's re-run our test.

{{< highlight python >}}
Traceback (most recent call last):
File "tests.py", line 2, in <module>
  from arraysum import ArraySum
  ImportError: cannot import name ArraySum
{{< / highlight >}}

Yay new error! This time it can't find our function, `ArraySum`, lets (minimally) create that in our new `arraysum.py` file.

{{< highlight python >}}
def ArraySum(int_list):
    pass
{{< / highlight >}}

Okay, let's see what we get now.
{{< highlight python >}}
F
======================================================================
FAIL: test_for_int (__main__.ArraySumTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "tests.py", line 10, in test_for_int
  self.assertTrue(type(result) is int, "Result is not integer")
  AssertionError: Result is not integer

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (failures=1)
{{< / highlight >}}

<del>More</del>MOAR errors!! This time our test is failing because we're not returning an integer. Let's fix that by modifying our return statement.

{{< highlight python >}}
def ArraySum(int_list):
    return 0
{{< / highlight >}}

Alright, lets re-run our test.

{{< highlight python >}}
.
----------------------------------------------------------------------
Ran 1 test in 0.000s

OK

{{< / highlight >}}

Woohoo! Our first test passed. According to TDD, we can now write some more tests. Let's make this one a little more interesting.
{{< highlight python >}}
import unittest
from arraysum import ArraySum

class ArraySumTests(unittest.TestCase):
    def setUp(self):
        pass

    def test_for_int(self):
        result = ArraySum([1, 2, 3, 4, 5,])
        self.assertTrue(type(result) is int, "Result is not integer")

    def test_flat_list(self):
        result = ArraySum([1, 2, 3, 4, 5])
        self.assertTrue(result == 15, "Incorrect sum")

if __name__ == '__main__':
    unittest.main()

{{< / highlight >}}

We've now added a new test, `test_flat_list` to test a simple list of integers and ensure the sum is correct. Back to running tests...
{{< highlight python >}}
F.
======================================================================
FAIL: test_flat_list (__main__.ArraySumTests)
----------------------------------------------------------------------
Traceback (most recent call last):
File "tests.py", line 14, in test_flat_list
  self.assertTrue(result == 15, "Incorrect sum")
  AssertionError: Incorrect sum

----------------------------------------------------------------------
Ran 2 tests in 0.000s

FAILED (failures=1)
{{< / highlight >}}

The obvious problem here is that our function is returning 0 and not doing any sort of summation. According to TDD, we want to write the minimal amount of code, Python, being the beautiful language that it is, provides us with a simple `sum` function which operates on iterables. Allowing us to very easily pass our `test_with_flat_list` test.

{{< highlight python >}}
def ArraySum(int_list):
    return sum(int_list)
{{< / highlight >}}

Running our test again will show that we were able to successfully pass it.
{{< highlight python >}}
..
----------------------------------------------------------------------
Ran 2 tests in 0.000s

OK
{{< / highlight >}}

We can continue writing more test cases, so lets make it a little more interesting. We'll introduce some non-integer values into the list, which will create errors for our `sum` function in a test case called `test_complex_list`.
{{< highlight python >}}
import unittest
from arraysum import ArraySum

class ArraySumTests(unittest.TestCase):
    def setUp(self):
        pass

    def test_for_int(self):
        result = ArraySum([1, 2, 3, 4, 5,])
        self.assertTrue(type(result) is int, "Result is not integer")

    def test_flat_list(self):
        result = ArraySum([1, 2, 3, 4, 5])
        self.assertTrue(result == 15, "Incorrect sum")
    
    def test_complex_list(self):
        result = ArraySum([1, 2, 3, "hello"])
        self.assertTrue(result == 6, "Incorrect sum")


if __name__ == '__main__':
    unittest.main()

{{< / highlight >}}

Running our tests, returns the results:

{{< highlight python >}}
E..
======================================================================
ERROR: test_complex_list (__main__.ArraySumTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "tests.py", line 17, in test_complex_list
  result = ArraySum([1, 2, 3, "hello"])
    File "/home/caster/Development/arraysum/arraysum.py", line 2, in ArraySum
        return sum(int_list)
        TypeError: unsupported operand type(s) for +: 'int' and 'str'

----------------------------------------------------------------------
Ran 3 tests in 0.000s

FAILED (errors=1)
{{< / highlight >}}

Just as we expected! Let's make some corrections to our `ArraySum` function.
{{< highlight python >}}
def ArraySum(int_list):
    sum = 0
    for x in int_list:
        if type(x) is int:
            sum += x
    return sum
{{< / highlight >}}

After making our fix, we'll re-run our test and...
{{< highlight python >}}
 ...
----------------------------------------------------------------------
Ran 3 tests in 0.000s

OK

{{< / highlight >}}

Looks good! Once again, passed test cases means we can write more tests, so lets give it a go one last time. We'll create a new test case, `test_complex_nested_list`, where we'll introduce a nested list into our previous complex list. A nested list within our function also brings in the sense of recursion! We'll want to take in account what we have, where we want to go, and if we've done something like this before (which we have). Currently we're iterating through all the values and if they're integers we'll add them to our `sum` variable. We're throwing away that is anything but an `int`. We know this is wrong though since the function must also include the values within nested lists, hence the embarrassingly recursive algorithm.

We know that we're going to have to iterate through the values in our list, so we understand that, but lets look at our cases:

* Value is an `int`: Add to sum
* Value is a `list`: Add the total of the list to sum
* Value is neither: Ignore

Now that we have an idea of what we want to do, lets write our last test case.
{{< highlight python >}}
import unittest
from arraysum import ArraySum

class ArraySumTests(unittest.TestCase):
    def setUp(self):
        pass

    def test_for_int(self):
        result = ArraySum([1, 2, 3, 4, 5,])
        self.assertTrue(type(result) is int, "Result is not integer")

    def test_flat_list(self):
        result = ArraySum([1, 2, 3, 4, 5])
        self.assertTrue(result == 15, "Incorrect sum")
    
    def test_complex_list(self):
        result = ArraySum([1, 2, 3, "hello"])
        self.assertTrue(result == 6, "Incorrect sum")

    def test_complex_nested_list(self):
        result = ArraySum([1, 2, 3, "hello", [4, 5]])
        self.assertTrue(result == 15, "Incorrect sum")

if __name__ == '__main__':
    unittest.main()

{{< / highlight >}}

Before we make any changes to `ArraySum`, we have to run our test.
{{< highlight python >}}
.F..
======================================================================
FAIL: test_complex_nested_list (__main__.ArraySumTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "tests.py", line 22, in test_complex_nested_list
      self.assertTrue(result == 15, "Incorrect sum")
      AssertionError: Incorrect sum

----------------------------------------------------------------------
Ran 4 tests in 0.000s

FAILED (failures=1)

{{< / highlight >}}

Now that we have a failed test, we are given the okay by the testing goat to modify our code. Let's add our recursive call, so our function now looks like:
{{< highlight python >}}
def ArraySum(int_list):
    sum = 0
    for x in int_list:
        if type(x) is int:
            sum += x
        elif type(x) is list:
            sum += ArraySum(x)
    return sum
{{< / highlight >}}

Re-running our final test...
{{< highlight python >}}
....
----------------------------------------------------------------------
Ran 4 tests in 0.000s

OK
{{< / highlight >}}

Ahhhhh :)
