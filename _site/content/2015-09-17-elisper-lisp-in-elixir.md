
I recently read a really interesting and great blog post titled [Lisp in your Language](http://danthedev.com/2015/09/09/lisp-in-your-language/) by Dan Prince. The article walked through an implementation of a subset of Lisp. After reading the post, I thought it was a great project to not only have a deeper understanding of Lisp but also of the language that the implementation is in. In fact, I was so inspired by the post, I decided that maybe I'd give it a shot to write it in [Elixir](https://elixir-lang.org).

I've been reading and messing around with Elixir for the past month or so. Elixir is a dynamically typed functional programming language built on top of the Erlang VM BEAM. The language is showing a lot of promise and even more interesting it's built on top of a very scalable platform.

Funny enough, besides syntax, I find Elixir very similar to Clojure (a Lisp dialect implemented on the JVM). If you're not familiar with Lisps by now, they have a very simple syntax that follows the convention

{{< highlight clojure >}}
(fn-name arg1 arg2 ... argn )
{{< / highlight >}}

With introductions out of the way, I'll try to write this as incrementally as possible. Unfortunately, I didn't make initial git commits when I was starting out, so some of the initial code is from memory.

#### Starting Out ####
In the blog post, Prince originally starts out by evaluating a predefined function using Javascript's `fn.apply` method. I thought this was a good way to start so I began by doing the same.

{{< highlight elixir >}}
defmodule Elisper do
  def eval([expr | args]) do
    apply(expr, args)
  end
end
{{< / highlight >}}

Similarly to Javascipt, Elixir has an `apply/2` function which allows you to provide a function name and a list of arguments to pass to that function. The nice thing about function parameters in Elixir is that we can pattern match in them immediately, allowing us to easily pull apart the values in a list and alias them. For the parameter `[expr | args]`, we're saying bind the first value (or head) of the list to the name `expr` and the rest of the list (or tail) to the name `args` (note that `args` is also a list).

We can test this out through unit tests:

{{< highlight elixir >}}
defmodule ElisperTest do
  use ExUnit.Case, async: true

  test "sum function" do
    sum = fn (a, b) -> a + b end
    assert (Elisper.eval([sum, 1, 2])) == 3
  end
end
{{< / highlight >}}

In our first unit test, we're creating an anonymous function `sum` which we're passing to our `eval` function with the arguments 1 and 2. Note that for pattern matching our `expr` value would be the function `sum` and the `args` parameter would take the value `[1,2]`. Let's run our test and see what we get...

    mix test test/elisper_test.exs:8                                             0|10:47:19
    Compiled lib/elisper.ex
    Generated elisper app
    Including tags: [line: "8"]
    Excluding tags: [:test]

    .

    Finished in 0.07 seconds (0.07s on load, 0.00s on tests)
    1 tests, 0 failures

#### Mathematical and Native Operations ####
The next step in Dan's article is the ability to pass arithmetic functions such as plus, minus, division, subtraction, etc. Dan accomplishes this by creating a Javascript object which he uses the passed in string as a key in the object's properties. We can follow a similar example in Elixir using Maps and atoms for cleaner syntax. Originally I was hoping to be able to pass in the already defined arithmetic functions in Elixir such as `Kernal.+/2` but I was unsuccessful in passing the function without it being evaluated. Our syntax for evaluating our "native" functions will be using atoms. Let's start by writing a test to show how this would look.

{{< highlight elixir >}}
defmodule ElisperTest do
  use ExUnit.Case, async: true

  test "sum function" do
    sum = fn (a, b) -> a + b end
    assert (Elisper.eval([sum, 1, 2])) == 3
  end

  test "native addition function" do
    assert (Elisper.eval([:+, 1, 2])) == 3
  end
end
{{< / highlight >}}

You can see that from the new test the arguments to our eval function is now a list containing an atom `:+` to represent our native addition function with the same arguments 1 and 2. Let's update our implementation to support these native functions.

We'll start by defining a new eval function using a guard statement. A guard statement allows us to define a function that will only be called when a set of conditions are true. In this case, we want to create an `eval` that will be called when we pass an `atom` as the head of the list. In this function we'll define our `native_ops` map which will be a map containing a list of our native arithmetic functions.

{{< highlight elixir >}}
defmodule Elisper do
  def eval([expr | args] = expression) when is_atom(expr) do
    native_ops = %{
      +: fn (a,b) -> a + b end,
      -: fn (a,b) -> a - b end,
      *: fn (a,b) -> a * b end,
      /: fn (a,b) -> a / b end,
      =: fn (a,b) -> a == b end
    }
    case Map.get(native_ops, expr) do
      nil -> eval(expression)
      func -> eval([func] ++ args)
    end
  end

  def eval([expr | args]) do
    apply(expr, args)
  end
end
{{< / highlight >}}

Let's run our tests to verify that what we've done works correctly:

    $ mix test test/elisper_test.exs                                              0|13:54:00
    ..

    Finished in 0.03 seconds (0.03s on load, 0.00s on tests)
    2 tests, 0 failures

    Randomized with seed 398982

#### Handling Nested Expressions ####

The next thing we want to do is handle nested arguments. Lisps can evaluate nested expressions in the form:

{{< highlight clojure >}}
(+ (+ 1 1) (+ 1 1))
{{< / highlight >}}

We want to be able to handle the same expressions. Since we already have an idea of what we want to test, let's start by again writing our tests first.

{{< highlight elixir >}}
defmodule ElisperTest do
  use ExUnit.Case, async: true

  test "sum function" do
    sum = fn (a, b) -> a + b end
    assert (Elisper.eval([sum, 1, 2])) == 3
  end

  test "native addition function" do
    assert (Elisper.eval([:+, 1, 2])) == 3
  end

  test "two recursive arguments" do
    assert (Elisper.eval([:+, [:+, 1, 1], [:+, 1, 1]])) == 4
 end

end
{{< / highlight >}}

Before attempting the implementation for evaluating nested arguments. I'd like to explain the solution that I took. Before evaluating the function and its arguments in our `eval` function that calls `apply/2`, we could iterate through the list of values and determine if a list exists. If one of the values is a list, we can recursively call eval, which will format and evaluate the nested expression. Let's see what that looks like:

{{< highlight elixir >}}
defmodule Elisper do
  def eval([expr | args] = expression) when is_atom(expr) do
    native_ops = %{
      +: fn (a,b) -> a + b end,
      -: fn (a,b) -> a - b end,
      *: fn (a,b) -> a * b end,
      /: fn (a,b) -> a / b end,
      =: fn (a,b) -> a == b end
    }
    case Map.get(native_ops, expr) do
      nil -> eval(expression)
      func -> eval([func] ++ args)
    end
  end

  def eval([expr | args]) do
    sub_exprs = Enum.map(args, fn
      ([_h | _t] = arg) -> eval(arg)
      arg -> arg
    end)
    apply(expr, sub_exprs)
  end
end
{{< / highlight >}}

Lets break this down piece by piece. First we start out by enumerating over all of our arguments using `Enum.map`. As people are familiar with `map` functions in other languages, they accept a collection to iterate through and a function to apply to each argument. In this case we've created an anonymous function that has two bodies. An awesome feature about Elixir is that anonymous functions can pattern match and have multiple bodies allowing us to handle different input. In this case, we want to specifically handle two inputs, one where the value is a list and when the supplied argument is just a single value. By pattern matching on an item with a head and tail, we can match against lists and as we've stated before, recursively call eval on our list. If the supplied argument is just a value, we simply want to return it. Once we've completed enumerating through and evaluating all of the arguments, we can finally call `apply/2` with our original `expr` and the newly evaluated `sub_exprs`.

#### Conditionals ####
One of the many features we use while programming is conditionals. To make some interesting expressions, let's add support for conditionals into our implementation. Conditionals in the Lisps take the form:
{{< highlight clojure >}}
(if (conditional)
  (true)
  (false)
)
{{< / highlight >}}

We can define a simple conditional using the test:

{{< highlight elixir >}}
test "if clause" do
  assert (Elisper.eval([:if, [:=, 1, 1], [:+, 1, 1], [:+, 2, 2]])) == 2
end
{{< / highlight >}}

We can handle the case for `if` within our `native_ops` map. Within `native_ops`  we can simply define a function for the if key that accepts 3 arguements, a `conditional`, the `first` (or true) clause, and the `second` (or false) clause. From there we can simply evaluate our `conditional` that is passed to our function. As you may have guessed, if `conditional` evaluates to true, we simply return `first`, else we return `second`.

{{< highlight elixir >}}
defmodule Elisper do
  def eval([expr | args] = expression) when is_atom(expr) do
    native_ops = %{
      +: fn (a,b) -> a + b end,
      -: fn (a,b) -> a - b end,
      *: fn (a,b) -> a * b end,
      /: fn (a,b) -> a / b end,
      =: fn (a,b) -> a == b end,
      if: fn(conditional, first, second) -> if(conditional) do first else second end end
    }
    case Map.get(native_ops, expr) do
      nil -> eval(expression)
      func -> eval([func] ++ args)
    end
  end

  def eval([expr | args]) do
    sub_exprs = Enum.map(args, fn
      ([_h | _t] = arg) -> eval(arg)
      arg -> arg
    end)
    apply(expr, sub_exprs)
  end
end
{{< / highlight >}}


#### Binding Values ####
One of the harder things that I came across was the ability to define values. Values in Lisps are bound using the `def` expression. These bound values are immutable and cannot be changed. Let's take a look at what an expression would look like:

{{< highlight clojure >}}
(def a 5)
{{< / highlight >}}

Doing so allows us to use these values in other contexts like:
{{< highlight clojure >}}
(do
  (def a 5)
  (+ a a)
)
{{< / highlight >}}

At this point, we have an idea for our next test and we also know what we want to implement, `do` and `def` expressions.
{{< highlight elixir >}}
defmodule ElisperTest do
  use ExUnit.Case, async: true

  test "sum function" do
    sum = fn (a, b) -> a + b end
    assert (Elisper.eval([sum, 1, 2])) == 3
  end

  test "native addition function" do
    assert (Elisper.eval([:+, 1, 2])) == 3
  end

  test "two recursive arguments" do
    assert (Elisper.eval([:+, [:+, 1, 1], [:+, 1, 1]])) == 4
 end

 test "def" do
   assert (Elisper.eval([:do, [:def, :a, 5], [:+, :a, :a]])) == 10
 end

end
{{< / highlight >}}

Now that we have our tests defined, we need to review what our implementation may look like. After trying out a number of options, I decided the best way to handle `def` clauses was to treat them separately before evaluating any expressions. I would iterate through the passed in arguments and define a `scope` which could be carried through evaluation that would hold all of the bounded values from the `def` clauses. Some of our steps:

  * Update existing `eval` functions to accept a second `scope` parameter.
  * Implement a new `eval` function that accepts only one parameter (the expression to evaluate) where the default scope is an empty scope.
  * Before calling our `eval/2` functions, we'll update our scope with all the bounded variables to pass to our lower `eval/2` functions.
  * Handle our `def` and `do` functions.
  * Replace any instances of our variables with their bounded values.

**Step 1:** Update existing functions and define new `eval/1` function.

{{< highlight elixir >}}
defmodule Elisper do

  def eval([expr | args] = expression, scope) when is_atom(expr) do
    native_ops = %{
      +: fn (a,b) -> a + b end,
      -: fn (a,b) -> a - b end,
      *: fn (a,b) -> a * b end,
      /: fn (a,b) -> a / b end,
      =: fn (a,b) -> a == b end,
      if: fn(conditional, first, second) -> if(conditional) do first else second end end
    }
    case Map.get(native_ops, expr) do
      nil -> eval(expression, scope)
      func -> eval([func] ++ args, scope)
    end
  end

  def eval([expr | args], scope) do
    sub_exprs = Enum.map(args, fn
      ([_h | _t] = arg) -> eval(arg, scope)
      arg -> arg
    end)
    apply(expr, sub_exprs)
  end

end
{{< / highlight >}}

**Step 2:** Define new `eval/1` function.
{{< highlight elixir >}}
defmodule Elisper do

  def eval(expr) do
    eval(expr, %{})
  end

  def eval([expr | args] = expression, scope) when is_atom(expr) do
    native_ops = %{
      +: fn (a,b) -> a + b end,
      -: fn (a,b) -> a - b end,
      *: fn (a,b) -> a * b end,
      /: fn (a,b) -> a / b end,
      =: fn (a,b) -> a == b end,
      if: fn(conditional, first, second) -> if(conditional) do first else second end end
    }
    case Map.get(native_ops, expr) do
      nil -> eval(expression, scope)
      func -> eval([func] ++ args, scope)
    end
  end

  def eval([expr | args], scope) do
    sub_exprs = Enum.map(args, fn
      ([_h | _t] = arg) -> eval(arg, scope)
      arg -> arg
    end)
    apply(expr, sub_exprs)
  end

{{< / highlight >}}

**Step 3:** In step 3, we iterate through `expr` but this time we use the `reduce` function to output a map which contains the values bounded by the `def` expressions. We can once again use pattern matching to make our anonymous function more concise. Our first case, where we match the head of the list to `:def` we know to return a new map with our new value. In the case that we don't find a match, simply return the existing map.

{{< highlight elixir >}}
defmodule Elisper do
  def eval(expr) do
    scope = Enum.reduce(expr, %{}, fn
      ([:def | arg], acc) -> Map.put(acc, List.first(arg), List.last(arg))
      (arg, acc) -> acc
    end)
    eval(expr, scope)
  end

  def eval([expr | args] = expression, scope) when is_atom(expr) do
    native_ops = %{
      +: fn (a,b) -> a + b end,
      -: fn (a,b) -> a - b end,
      *: fn (a,b) -> a * b end,
      /: fn (a,b) -> a / b end,
      =: fn (a,b) -> a == b end,
      if: fn(conditional, first, second) -> if(conditional) do first else second end end
    }
    case Map.get(native_ops, expr) do
      nil -> eval(expression, scope)
      func -> eval([func] ++ args, scope)
    end
  end

  def eval([expr | args], scope) do
    sub_exprs = Enum.map(args, fn
      ([_h | _t] = arg) -> eval(arg, scope)
      arg -> arg
    end)
    apply(expr, sub_exprs)
  end

end
{{< / highlight >}}

**Step 4:** Handling the `def` and `do` expressions during evaluation.
Our `def` and `do` functions are a little weird and we'll have to handle them both a little differently than our arithmetic functions. In the case of `do`, we need to take a step back and think about what the function actually does. According to the [Clojure Docs](https://clojuredocs.org/clojure.core/do) on `do`:
> Evaluates the expressions in order and returns the value of the last. If no
expressions are supplied, returns nil.

Following the documentation, we can simply define a function for `do` which returns the last argument that has been passed to it, that's simple enough. In the case of `def`, since we've already handled it's purpose by binding the value to its alias within the scope, we can simply ignore it during evaluation.

{{< highlight elixir >}}
defmodule Elisper do
  def eval(expr) do
    scope = Enum.reduce(expr, %{}, fn
      ([:def | arg], acc) -> Map.put(acc, List.first(arg), List.last(arg))
      (arg, acc) -> acc
    end)
    eval(expr, scope)
  end

  def eval([expr | args] = expression, scope) when is_atom(expr) do
    native_ops = %{
      +: fn (a,b) -> a + b end,
      -: fn (a,b) -> a - b end,
      *: fn (a,b) -> a * b end,
      /: fn (a,b) -> a / b end,
      =: fn (a,b) -> a == b end,
      if: fn(conditional, first, second) -> if(conditional) do first else second end end,
      do: fn(_a, b) -> b end
    }
    case Map.get(native_ops, expr) do
      nil -> (
        if expr == :def do
          :ok
        else
          eval(expression, scope)
        end
      )
      func -> eval([func] ++ args, scope)
    end
  end

  def eval([expr | args], scope) do
    sub_exprs = Enum.map(args, fn
      ([_h | _t] = arg) -> eval(arg, scope)
      arg -> (
        case Map.get(scope, arg) do
          nil -> arg
          x -> x
        end
      )
    end)
    apply(expr, sub_exprs)
  end
end
{{< / highlight >}}

#### Defining Functions ####
For our last section, we'll cover how to define functions. We'll use the following syntax provided by Dan's article for defining a function:

{{< highlight clojure >}}
(def fn-name
  (fn [args] (body))
)
{{< / highlight >}}

Using this syntax, functions must be defined with `def`, this may give you a hint as to where we'll need to add it in the implementation. Similar to other `def` expressions, we'll want to be able to add our function information to the scope, so that it can be used throughout the evaluation of the rest of the expressions. Functions are a slightly different case of the `def` expression that we have, so we'll need to take that into account. Before jumping into the implementation, let's write another test.

{{< highlight elixir >}}
test "def fn" do
 assert (Elisper.eval(
    [:do,
     [:def, :multi,
       [:fn, [:x, :y],
         [:*, :x, :y]
       ]
     ],
     [:multi, 3, 4]
    ]
  )) == 12
end
{{< / highlight >}}

One more thing to think about before we begin the implementation is how we want to add function information to our scope. I chose to create a map containing a `params` key and a `body` key to store the params and body of the function, respectively. After updating our scope should look similar to:

{{< highlight elixir >}}
%{
  params: [:x, :y],
  body:   [:*, :x, :y]
}
{{< / highlight >}}

Now that we have an idea of what we want to implement, let's go back to our `Enum.reduce` function.

{{< highlight elixir >}}
scope = Enum.reduce(expr, %{}, fn
  ([:def | arg], acc) -> Map.put(acc, List.first(arg), List.last(arg))
  (arg, acc) -> acc
end)
{{< / highlight >}}

As we stated before, we'll want to add a case to match, we know that we still want to match for on `def` as the head of the list, but what about the tail? The tail of our list has the format `[fn-name, [:fn, [params], [body]]]`. The great thing is that we can pattern match to exactly that format. Since we've extracted the values from the list, we can easily add a new map to the scope.

{{< highlight elixir >}}
scope = Enum.reduce(expr, %{}, fn
  ([:def | [func_name, [:fn, params, body]]], acc) -> (
    Map.put(acc, func_name, %{params: params, body: body})
  )
  ([:def | arg], acc) -> Map.put(acc, List.first(arg), List.last(arg))
  (arg, acc) -> acc
end)
{{< / highlight >}}

The next thing we'll want to do, is handle what to do when we come across `:fn` and our user created functions during evaluation. This hapens in the `nil` case when we lookup function replacements from our `native_ops` map. We'll have to modify the `nil` case a bit to get our desired behavior. First, we'll want to update our conditional to also return `:ok` when we come across a `:fn` value in our list of expressions. The next thing we'll want to do, is see if the `expr` is defined in our scope and if it is, evaluate the function with the correct parameters. Let's see what this looks like:

{{< highlight elixir >}}
case Map.get(native_ops, expr) do
  nil -> (
    if expr == :def || expr == :fn do
      :ok
    else
      case Map.get(scope, expr) do
        %{body: body, params: params} -> (
          new_scope = Enum.zip(params, args)
            |> Enum.reduce(scope, fn({key, val}, acc) -> Map.put(acc, key, val) end)
          eval(body, new_scope)
        )
        nil -> eval(expression, scope)
      end
    end
  )
  func -> eval([func] ++ args, scope)
end
{{< / highlight >}}

Using the `Map.get` function we'll lookup the `expr` in our `scope`. If a case matches the pattern, `%{body: body, params: params}`, that we defined for our function entries, we need to evaluate the user defined function. Before simply passing the `body` to our `eval` function, we'll need to update our `scope` with the new values. To do so, we'll first use `Enum.zip` to create a list of tuples containing the parameter name and the value associated with it. In the case of our test, our zipped list would look something like: `[{:x, 3}, {:y, 4}]`. Using this list we can once again use `Enum.reduce` to create a new scope that adds the key/value pair of the tuple to our existing scope. Using our new scope, so cleverly titled `new_scope`, and our `body` to finally evaluate the expression.

We can now re-run all of our tests:

    mix test test/elisper_test.exs                                              0|10:06:43
    ......

    Finished in 0.04 seconds (0.04s on load, 0.00s on tests)
    6 tests, 0 failures

    Randomized with seed 969698

#### Conclusion ####

I hope that you enjoyed this blog post and maybe I've peaked your interest to take a look into Elixir. I've put the code on [Github](https://github.com/barakyo/elisper) to mess around with. Pull requests are welcomed if you want to mess around and add features or clean up the code for me to learn from. :)
