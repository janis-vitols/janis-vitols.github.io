---
layout: post
title:  "Gemfile inside application"
date:   2017-03-03 21:19:00 +0200
categories: ruby gemfile bundler
comments: true
disqus_identifier: 0FCD2511-5C0F-4BB7-8561-98EB0DCA89EE
---

Quick tip here - you can write your Gemfile dependencies within your application file!

As a ruby/rails developer I'm used to Gemfile's where I can specify which gems and versions I'm using within
my project. But there can be cases when your application is so small that it would be cool if all the code
could be placed in one file, not having two separate files: application & Gemfile.

All you need to do, is to require `bundler/inline` and use [#gemfile][gemfile-method]{:target="blank"} method with block.
Let's use [rainbow][rainbow-gem]{:target="blank"} gem and create small application `coloriz.rb` example here.

{% highlight ruby linenos %}
#!/usr/bin/env ruby

require 'bundler/inline'

gemfile(true) do
  source 'https://rubygems.org'
  gem 'rainbow', '~> 2.2.1'
end

puts Rainbow("hola!").red.bright.underline
{% endhighlight %}

After that we can run our application with `ruby coloriz.rb`. But probably it would be smarter to add execution permissions
and run it like normal unix application.

{% highlight bash linenos %}
# set execute permissions
chmod +x coloriz.rb

# run our example application
./coloriz.rb
Fetching gem metadata from https://rubygems.org/..
Fetching version metadata from https://rubygems.org/.
Resolving dependencies...
Installing rainbow 2.2.1 with native extensions
Using bundler 1.14.5
hola!
{% endhighlight %}

And that's it! Now your Gemfile is inline with your application code. When user will try to run application it will automatically
check where all dependencies are in place and in case if not - install them.

At this momen I see only two caveats:

* every time when user will run application it will take some time to check dependencies
* no `Gemfile.lock` file will be created - my suggestion would be to specify version number pretty close as with updates it could break your application

In case if there is no need for gem auto-install you can set `install` parameter to `false` or remove it from `#gemfile` method call.
In such case it will only check for necessary gems in your system and if there will not be neede version you will receive an error, but overall
application start will be much more faster.

`Could not find gem 'rainbow (~> 2.2.1)' in any of the gem sources listed in your Gemfile or available on this machine. (Bundler::GemNotFound)`

* * *
Get source code: `git clone git@github.com:janis-vitols/examples.git --branch ruby/bundler/gemfile-inside-application --single-branch coloriz`

[gemfile-method]: https://github.com/bundler/bundler/blob/master/lib/bundler/inline.rb#L31
[rainbow-gem]:    https://github.com/sickill/rainbow
