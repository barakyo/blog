
While browsing [/r/javascript](http://www.reddit.com/r/javascript) Thursday night, I came across a post regrading a [mildly interesting interview question](http://www.reddit.com/r/javascript/comments/1qn7bo/mildly_interesting_question_i_got_while/) which had to do with flattening a Javascript object to a URI.

Two things immediately jumped into my mind:

1. I have to show one of my coworkers.
2. This would make a "midly interesting" Python challenge!

Friday morning, the first thing I did when I got into work was write the challenge on our white board and my solution in Python. Funny enough, once I came back from lunch, my coworker was writing his Javascript solution next to my Python one. It turns out that he saw the challenge Thursday night and even posted his solution to the thread.

I figured, I'd code up my solution and give it a shot. Both for recursive practice and writing test cases. So to start with some test driven development (TDD), I began by writing a simple test case.

First we'll define a class for our test cases:
{{< highlight python >}}
import unittest
class FlattenTests(unittest.TestCase):
{{< / highlight >}}

From there, we'll add a simple `setUp` method which creates a dictionary to use for testing:
{{< highlight python >}}
import unittest

class FlattenTests(unittest.TestCase):
    def setUp(self):
        self.example = {
                "foo": {
                    "bar": {
                        "team": True,
                        "company": ["Bill", "Ted"]
                        }
                    }
                }
{{< / highlight >}}

Now lets define a test case!
{{< highlight python >}}
import unittest

class FlattenTests(unittest.TestCase):
    def setUp(self):
        self.example = {
                "foo": {
                    "bar": {
                        "team": True,
                        "company": ["Bill", "Ted"]
                        }
                    }
                }

    def test_for_dict(self):
        d = flatten(self.example, "")
        self.assertTrue(type(d) is dict)


if __name__ == '__main__':
    unittest.main()
{{< / highlight >}}

The test case we've defined, `test_for_dict`, just determines if the return type of the `flatten` method (which we haven't written yet) is of type dictionary. Lets run our test case and see what we get!

{{< highlight bash >}}
ERROR: test_for_dict (__main__.FlattenTests)
----------------------------------------------------------------------
Traceback (most recent call last):
File "tests.py", line 16, in test_for_dict
d = flatten(self.example, "")
NameError: global name 'flatten' is not defined
----------------------------------------------------------------------
{{< / highlight >}}

Ahh what was that!? According to my understanding of TDD, this is a good thing! We can start writing code now. Lets define a new file `flatten.py` and create a flatten function that accepts a dictionary and string. Remember, according to TDD, we must right the most minimal amount of code that will pass our test.

{{< highlight python >}}
def flatten(d_flat, key_string):
    pass

{{< / highlight >}}

Well that was simple enough, lets run our test.
{{< highlight bash >}}
======================================================================
FAIL: test_for_dict (__main__.FlattenTests)
----------------------------------------------------------------------
Traceback (most recent call last):
File "tests.py", line 17, in test_for_dict
self.assertTrue(type(d) is dict)
AssertionError: False is not true
----------------------------------------------------------------------
{{< / highlight >}}

Hmm, seems like we didn't write enough code! Lets go back and update our `flatten` function.
{{< highlight python >}}
def flatten(d_flat, key_string):
   return {} 

{{< / highlight >}}

Now that we've updated our code, lets run our test again.

{{< highlight bash >}}
caster:flatten/ $ python tests.py
.
----------------------------------------------------------------------
Ran 1 test in 0.000s

OK
{{< / highlight >}}

Looks good to me! Now that one of our tests has passed, we can now write another (more interesting) test! I'm not very creative at writing these incremental tests, so lets just get to it.
{{< highlight python >}}
import unittest
from flatten import flatten

class FlattenTests(unittest.TestCase):
    def setUp(self):
        self.example = {
                "foo": {
                    "bar": {
                        "team": True,
                        "company": ["Bill", "Ted"]
                        }
                    }
                }

    def test_for_dict(self):
        d = flatten(self.example, "")
        self.assertTrue(type(d) is dict)

    def test_case_one(self):
        d = flatten(self.example, "")
        self.assertTrue(d["foo/bar/team"])
        self.assertTrue(d["foo/bar/company"] == ["Bill", "Ted"]) 

if __name__ == '__main__':
    import unittest
{{< / highlight >}}

We've defined a new test case, `test_case_one`, (like I said, I'm not very creative), that'll run our `flatten` function and determine if we've received the correct output. Lets try running our test case now (and cross our fingers) to see if it fails.

{{< highlight bash >}}
caster:flatten/ $ python tests.py
E.
======================================================================
ERROR: test_case_one (__main__.FlattenTests)
----------------------------------------------------------------------
Traceback (most recent call last):
File "tests.py", line 21, in test_case_one
self.assertTrue(d["foo/bar/team"])
KeyError: 'foo/bar/team'

----------------------------------------------------------------------
Ran 2 tests in 0.000s

FAILED (errors=1)
{{< / highlight >}}

Awesome! More good news! :) Before we jump into writing our function, lets think about what we want the function to do.

* First we'll need to iterate through all the key/value pairs in the dictionary passed to the function.
* We'll then need to determine if the value for the key we've iterated to is a dictionary or not.
    * If the value is a dictionary, we'll want to recursively call `flatten` and append our key to `key_string` so that we're keeping track of the key we want to eventually use.
* The easy (or base) case, if our value is **NOT** a dictionary, simply add it to the dictionary with it's key append to `key_string`
* Finally, once we've iterated through all the values, return the dictionary.

When we put it like that, it sounds easy! (Or so I hope...)

Lets take care of iterating through the dictionary, that sounds easy enough:
{{< highlight python >}}
def flatten(d_flat, key_string):
    d = {}
    for k,v in d_flat.items():
        pass
    return d
{{< / highlight >}}

Simple enough, we've declared an empty dictionary `d` and started iterating through the items in the dictionary. Lets take a look at our first case, if the value for a key is a dictionary:

{{< highlight python >}}
def flatten(d_flat, key_string):
    d = {}
    for k,v in d_flat.items():
        if(type(v) is dict):
            pass
        else:
            pass
    return d
{{< / highlight >}}

Okay we've added our if statement, now if it's true, we'll have to recursively call our function. Before we jump to make the recursive call, we need to ensure we're keeping track of our `key_string` value. To do so, we'll pass `key_string` with the current key value, `k`. We'll recursively call `flatten` with the inner dictionary stored in `v`.

{{< highlight python >}}
def flatten(d_flat, key_string):
    d = {}
    for k,v in d_flat.items():
        if(type(v) is dict):
            d = flatten(v, key_string + k + "/")
        else:
            pass
    return d
{{< / highlight >}}

Almost there! We now need to worry about our base case, so let's take care of that! Recall that we said if our value is not of type dictionary, we simple append it to our dictionary that we declared, `d`. Again, we have to keep in mind that we want to assign the correct key for our value, which would be our current `key_string` concatenated with the current key we're looking at, `k`. Once we create the right key, we'll just assign it to our value, `v` and return our dictionary, `d`.

{{< highlight python >}}
def flatten(d_flat, key_string):
    d = {}
    for k,v in d_flat.items():
        if(type(v) is dict):
            d = flatten(v, key_string + k + "/")
        else:
            d[key_string + k] = v
    return d

{{< / highlight >}}

So good so far, lets run our tests!
{{< highlight bash >}}
caster:flatten/ $ python tests.py
..
----------------------------------------------------------------------
Ran 2 tests in 0.000s

OK

{{< / highlight >}}

Perfect...or so we think. There's actually a problem in our algorithm, which we can expose by making our testing data little more robust. Lets update our `setUp` and our `test_case_one` functions so they look like this:

{{< highlight python >}}
import unittest
from flatten import flatten

class FlattenTests(unittest.TestCase):
    def setUp(self):
        self.example = {
                "foo": {
                    "bar": {
                        "team": True,
                        "company": ["Bill", "Ted"]
                        }
                    },
                    "fizz": {
                        "blue": "red"
                    }
                }

    def test_for_dict(self):
        d = flatten(self.example, "")
        self.assertTrue(type(d) is dict)

    def test_case_one(self):
        d = flatten(self.example, "")
        self.assertTrue(d["foo/bar/team"])
        self.assertTrue(d["foo/bar/company"] == ["Bill", "Ted"]) 
        self.assertTrue(d["foo/bar/fiz/blue"] is "red")

if __name__ == '__main__':
    unittest.main()
{{< / highlight >}}

We've added another key that points to another dictionary. Nothing unusual or out of the oridinary. We've also added a new assertion to ensure that the key `foo/bar/fiz/blue` contains the string `red`. Lets run our test case and see what happens.

{{< highlight bash >}}
caster:flatten/ $ python tests.py
E.
======================================================================
ERROR: test_case_one (__main__.FlattenTests)
----------------------------------------------------------------------
Traceback (most recent call last):
File "tests.py", line 21, in test_case_one
self.assertTrue(d["foo/bar/team"])
KeyError: 'foo/bar/team'

----------------------------------------------------------------------
Ran 2 tests in 0.000s

FAILED (errors=1)

{{< / highlight >}}

Failed! What's going on here? If you take a look at our algorithm, we're not **updating** our dictionary when we return from a recursive call, we're actually **overwriting** it. Let's make a small change and ensure we're updating our dictionary and see what happens.
{{< highlight python >}}
def flatten(d_flat, key_string):
    d = {}
    for k,v in d_flat.items():
        if(type(v) is dict):
            d.update(flatten(v, key_string + k + "/"))
        else:
            d[key_string + k] = v
    return d

{{< / highlight >}}

Now lets run our tests.

{{< highlight bash >}}
caster:flatten/ $ python tests.py
..
----------------------------------------------------------------------
Ran 2 tests in 0.000s

OK
{{< / highlight >}}

All is well and now our function works as expected!

