---
layout: post
title:  "Debugging in Elixir"
date:   2016-12-29 19:30:00 +0200
categories: elixir debugging
comments: true
disqus_identifier: A4B9F372-2BDA-4081-B14B-79DD7687644D
---

Hi there!

Debugging is very useful when you are searching for nasty bugs in code. In Ruby world I have used [byebug][byebug]{:target="blank"}
and [pry][pry]{:target="blank"} a lot for such situations. Landing into Elixir world is no different, you will have situations where
you need some good debugging tool by your hands to catch these bugs. :)

Let's check few of these debugging methods on this code sample:

{% highlight elixir linenos %}
defmodule Debugging do
  def main do
    sum_1 = sum(1, 1)
    sum_2 = sum(2, 3)

    multiply(sum_1, sum_2)
  end

  defp sum(a, b) do
    a + b
  end

  defp multiply(a, b) do
    a * b
  end
end
{% endhighlight %}

* Simple debugging by outputing into console using `IO.puts` LN#6-9

{% highlight elixir linenos %}
defmodule Debugging do
  def main do
    sum_1 = sum(1, 1)
    sum_2 = sum(2, 3)

    IO.puts "+++Debugging+++"
    IO.puts "Sum of 1 & 1 is #{sum_1}"
    IO.puts "Sum of 2 & 3 is #{sum_2}"
    IO.puts "+++End of debugging"

    multiply(sum_1, sum_2)
  end

  defp sum(a, b) do
    a + b
  end

  defp multiply(a, b) do
    a * b
  end
end
{% endhighlight %}

By starting Elixir’s [interactive shell][iex]{:target="blank"} `iex -S mix` and running `Debugging.main`
we should see our debugging output in the console (LN#6-9):

{% highlight bash linenos %}
iex -S mix
Erlang/OTP 19 [erts-8.2] [source] [64-bit] [smp:4:4] [async-threads:10] [hipe] [kernel-poll:false] [dtrace]

Interactive Elixir (1.3.4) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)> Debugging.main
+++Debugging+++
Sum of 1 & 1 is 2
Sum of 2 & 3 is 5
+++End of debugging
10
{% endhighlight %}

That's cool, but sometimes there could be a lot of variables which we would like to check (that means a lot of
`IO.puts`). Also maybe we would like to do some calculations on the fly or try out different approach.

* In case if we would like to stop caller process on specific line and access variable values,
experiment with some Elixir code on the fly we can use `IEx.pry` LN#1 (we need to require IEx in our app) and LN#21 (stop caller)

{% highlight elixir linenos %}
require IEx

defmodule Debugging do
  def main do
    sum_1 = sum(1, 1)
    sum_2 = sum(2, 3)

    IO.puts "+++Debugging+++"
    IO.puts "Sum of 1 & 1 is #{sum_1}"
    IO.puts "Sum of 2 & 3 is #{sum_2}"
    IO.puts "+++End of debugging"

    multiply(sum_1, sum_2)
  end

  defp sum(a, b) do
    a + b
  end

  defp multiply(a, b) do
    IEx.pry
    a * b
  end
end
{% endhighlight %}

We will need to close our interactive shell by pressing twice `CTRL + c` and boot it up back with `iex -S mix` to apply our changes.
After that let's run one more time `Debugging.main`.

{% highlight bash linenos %}
iex(1)> Debugging.main
+++Debugging+++
Sum of 1 & 1 is 2
Sum of 2 & 3 is 5
+++End of debugging
Request to pry #PID<0.106.0> at lib/debugging.ex:21


      defp multiply(a, b) do
        IEx.pry
        a * b
      end

Allow? [Yn]
{% endhighlight %}

We should see that we are prompted with `Request to pry`. Our caller has stopped where we wanted it to stop.
Now let's confirm it with `y` and press `enter`. Let's check what we have in our `a` and `b` params within our `multiply/2` function.
And also try some calculations within interactive shell to confirm that we can use Elixir. To continue our application running we
need to enter `respawn` command.

{% highlight bash linenos %}
iex(1)> Debugging.main
+++Debugging+++
Sum of 1 & 1 is 2
Sum of 2 & 3 is 5
+++End of debugging
Request to pry #PID<0.106.0> at lib/debugging.ex:21


      defp multiply(a, b) do
        IEx.pry
        a * b
      end

Allow? [Yn] y

pry(1)> a
2
pry(2)> b
5
pry(3)> a * b
10
pry(4)> respawn

Interactive Elixir (1.3.4) - press Ctrl+C to exit (type h() ENTER for help)
10
{% endhighlight %}

That's better than writing `IO.puts` all over, right? But there is still one caveat, we can't step forward
and set new break points debugging in this way.

* Here comes our third way of debugging. We can use `:debugger` from Erlang.

When we want to use Erlang `:debugger`, we don't need to require any models or insert any code for binding.
We just need to run few commands from interactive shell.

1) Type `:debugger.start` in our IEx session - an GUI `Monitor` app should open up

{% highlight bash linenos %}
iex(1)> :debugger.start
{:ok, #PID<0.108.0>}
{% endhighlight %}

![](/images/2016/debugging-in-elixir-monitor-1.png)

2) Now we can prepare our module for debugging by typing `:int.ni(Debugging)`

{% highlight bash linenos %}
iex(2)> :int.ni(Debugging)
{:module, Debugging}
{% endhighlight %}

We should see `Elixir.Debugging` in our `Monitor` window left side.
Try double clicking it an window `View Module Elixir.Debugging` should open up.

![](/images/2016/debugging-in-elixir-monitor-2.png)

3) Now we can either use GUI app and set some break points `Break -> Line Break...`

![](/images/2016/debugging-in-elixir-monitor-3.png)

Or we can set break points from our interactive shell `:int.break(Debugging, 17)`
(must specify module & line number).

{% highlight bash linenos %}
iex(3)> :int.break(Debugging, 17)
:ok
{% endhighlight %}

We should see red dots in our `View Module Elixir.Debugging` window. Let's run our `Debugging.main` function now
from interactive shell. It should look like our interactive shell is frozen, by switching back to `Monitor` window we should
see that there we have stopped on our first break point on LN#17. Now we can jump all around our application and do proper
debugging.

![](/images/2016/debugging-in-elixir-monitor-4.png)

Hope you enjoyed this post. Happy debugging in Elixir! :smirk:

* * *
Get source code: `git clone git@github.com:janis-vitols/examples.git --branch elixir/debugging/puts-pry-and-erland-debugger --single-branch debugging`

[byebug]: https://github.com/deivid-rodriguez/byebug
[pry]: https://github.com/pry/pry
[iex]: https://hexdocs.pm/iex/IEx.html
