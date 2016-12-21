---
layout: post
title:  "Elixir Doctests"
date:   2016-12-21 20:09:00 +0200
categories: elixir doctests documentation testing
comments: true
disqus_identifier: 9862690C-83C7-4A90-915E-817489396911
---

Hi!

Recently my gut feeling started to point me to the [Elixir][elixir]{:target="blank"} programming language. I don't know why, but it just somehow
felt right to learn it. And when I had opportunity to make small "pet" project, I jumped stright into coding.
Honestly - I was amazed with the experience! I managed to develop this project within few hours without knowing a lot about language itself.

For these who don't know, Elixir is dynamic functional programming language, so that means [OOP][oop]{:target="blank"} skills will not be
very useful here. Also you will need to "break" your brain a little bit to adapt to Elixir paradigms if you were coding in OOP style for a long time.

As I enjoy applications covered with tests I was curious what kind of testing possibilites there are in Elixir. By studying more about Elixir I
found out that language makes documentation a first-class citizen in the language. You may be wondering now - why I'm talking about documentation now.
Well, I'm talking about documentation because you can write [Doctests][lang-doctests]{:target="blank"} inside the documentation! This way you can
show some examples to someone who is using your application/library and also be sure that your examples are working correctly (are covered with tests).

Let's just jump into example itself (you will need to have Elixir installed on your machine if you will want to repeat these examples. More
information about that you can find [here][install-elixir]{:target="blank"}):

{% highlight bash linenos %}
# create new project `dtest`
$ mix new dtest
# change directory and run tests
$ cd dtest
$ mix test
Compiling 1 file (.ex)
Generated dtest app
.

Finished in 0.03 seconds
1 test, 0 failures
{% endhighlight %}

After these few commands you have successfuly created new `dtest` project and run your first Elixir tests :+1:

If you will open `test/dtest_test.exs` file now, you will see elementar test example there counting `1 + 1` and assert it to be `2` (this is the one which was shown in output results as
successful and is added by every new project):

{% highlight elixir linenos %}
defmodule DtestTest do
  use ExUnit.Case
  doctest Dtest

  test "the truth" do
    assert 1 + 1 == 2
  end
end
{% endhighlight %}

Now check what's on **LN#3** `doctest Dtest`. This is the magic line which I was telling you earlier - it runs test cases from documentation if there
are any. To prove this, let's open `lib/dtest.ex` and implement small function with documentation which contains examples.

{% highlight elixir linenos %}
defmodule Dtest do
  @doc """
    Approves that DocTests are awesome!

  ## Examples

      iex> Dtest.awesome?
      true

  """
  def awesome? do
    true
  end
end
{% endhighlight %}

And now run `mix test` to check the results:

{% highlight bash linenos %}
$ mix test                                                                                                                                                         ⏎
Compiling 1 file (.ex)
..

Finished in 0.06 seconds
2 tests, 0 failures
{% endhighlight %}

You can see that now we have `2` successful tests, but we didn't added test in the `test/dtest_test.exs` right? How awesome is that?
To prove that Doctests are run as tests, try changing `true` to `false` inside `Examples` section of `awesome/0` function documentation.
Your code should look like this now:

{% highlight elixir linenos %}
defmodule Dtest do
  @doc """
    Approves that DocTests are awesome!

  ## Examples

      iex> Dtest.awesome?
      false

  """
  def awesome? do
    true
  end
end
{% endhighlight %}

And rune one more time tests with `mix test` command:

{% highlight bash linenos %}
$ mix test
Compiling 1 file (.ex)


  1) test doc at Dtest.awesome?/0 (1) (DtestTest)
     test/dtest_test.exs:3
     Doctest failed
     code: Dtest.awesome? === false
     lhs:  true
     stacktrace:
       lib/dtest.ex:7: Dtest (module)

.

Finished in 0.07 seconds
2 tests, 1 failure
{% endhighlight %}

And you will get failing test here :boom:. Basically failed test is telling us that `Dtest.awesome? === false` doesn't assert.
And on next line `lhs:  true` it's telling us that `left hand side` we have `true` value (function call returned `true`).

Don't know how about you, but I was astonished with Doctests. It probably means that there will be less old/wrong documentations
for Elixir functions because of this functionality as tests will just start to break on changes. I really enjoy it and I think I will
be using such testing approach in my next projects coding in Elixir! :)

By the way, I highly suggest to check out more about [ExUnit.DocTest on hexdocs.pm][hexdocs-doctest]{:target="blank"}. You must know few
things about aligning to make them work and will find out other different ways how to use DocTest. Happy testing :smirk:!

[elixir]:          http://elixir-lang.org/
[install-elixir]:  http://elixir-lang.org/install.html
[oop]:             https://en.wikipedia.org/wiki/Object-oriented_programming
[lang-doctests]:   http://elixir-lang.org/getting-started/mix-otp/docs-tests-and-with.html#doctests
[hexdocs-doctest]: https://hexdocs.pm/ex_unit/ExUnit.DocTest.html
