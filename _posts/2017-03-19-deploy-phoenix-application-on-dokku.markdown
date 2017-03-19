---
layout: post
title:  "Deploy Phoenix application on Dokku"
date:   2017-03-19 19:09:00 +0200
categories: deploy dokku phoenix
comments: true
disqus_identifier: 57520735-4D38-49D9-BBCC-5DDF0C3795B0
---

By studying [Elixir][elixir-lang]{:target="blank"} I also found out [Phoenix][phoenix-framework]{:target="blank"} web framework written in this language.
As currently mostly I have worked with [Ruby on Rails][rubyonrails-framework]{:target="blank"} framework it was very interesting for me to create something in
functional programming style and deploy it to get some experience. Here in this post I'm describing process how I deployed my very first Phoenix application on
[Dokku][dokku-github]{:target="blank"}.

In this post I'm going to imagine that you have access to Dokku server or know how to setup one - you can `ssh` into server and run `dokku` command. In our days
it's very easy to setup one - on DigitalOcean there is even [droplet][digital-ocean-dokku]{:target="blank"} for Dokku.

#### Suggestion

First suggestion would be to create small bash script to run dokku commands on remote server from your local machine. It could be placed in your projects folder
`bin/dokku` file.

{% highlight bash linenos %}
#!/bin/bash
ssh -t <user>@<server> -- "$@"
{% endhighlight %}

And don't forget to set execute rights to this file `chmod +x bin/dokku` so you can run it from terminal `bin/dokku <arguments>`. Here you can also see that we are
using `-t` option for `ssh`.

From manual: `-t Force pseudo-terminal allocation.  This can be used to execute arbitrary screen-based programs on a remote machine, which can be very useful, e.g.
when implementing menu services.  Multiple -t options force tty allocation, even if ssh has no local tty.`

Let's check our dokku version with our newly created file:
{% highlight bash linenos %}
bin/dokku version
0.8.0
Connection to <server> closed.
{% endhighlight %}

In case if you got similar output, you are ready for application setup with Dokku :) Let's go!

#### Buildpacks

When we will configure our application, we will provide `BUILDPACK_URL` to `https://github.com/heroku/heroku-buildpack-multi`. As far as I know, this will allow us to use
some buildpacks which we will need. `BUILDPACK_URL` configuration will be provided later, but for now we need to create file `.buildpacks` and add there:

* [heroku-buildpack-elixir][heroku-buildpack-elixir]{:target="blank"} - for erlang/elixir configuration with `elixir_buildpack.config` file
* [heroku-buildpack-phoenix-static][heroku-buildpack-phoenix-static]{:target="blank"} - for static assets compilation and `npm` configurations

{% highlight bash linenos %}
touch .buildpacks
echo "https://github.com/HashNuke/heroku-buildpack-elixir.git" >> .buildpacks
echo "https://github.com/gjaldon/heroku-buildpack-phoenix-static.git" >> .buildpacks
{% endhighlight %}

#### Lock Erlang/Elixir versions

I'm used to lock Ruby version in projects where I work so in my Phoenix project I decided to lock Erlang and Elixir versions also to match my local machines.
This is done with `heroku-buildpack-elixir` buildpack, we will need to create new file `elixir_buildpack.config` and add few lines there.

{% highlight bash linenos %}
touch elixir_buildpack.config
echo "erlang_version=19.2" >> elixir_buildpack.config
echo "elixir_version=1.4.0" >> elixir_buildpack.config
{% endhighlight %}

#### Procfile

With [Procfile][procfiles]{:target="blank"} you can specify your workers and how exactly to start them. For my Phoenix application I need only one worker, which I call `web` and basically
it starts server.

{% highlight bash linenos %}
echo "web: mix phoenix.server" > Procfile
{% endhighlight %}

#### Run migrations before deploy

In Dokku you can run predeply/postdeploy scripts, usually in web applications I use predeploy to migrate database. For this you will need to add [app.json][appjson]{:target="blank"}
file in root folder of your project. Put this content within `app.json` file to run migrations before deployment.

{% highlight bash linenos %}
{
  "scripts": {
    "dokku": {
      "predeploy": "mix ecto.migrate"
    }
  }
}
{% endhighlight %}

#### Create Dokku app

Now to application creating part. I will list all these commads within one code block and add few comments before each line. We will use our earliery clearted
`bin/dokku` shell script for this.

{% highlight bash linenos %}
# With `apps:create` command we can create our dokku application
# we need to specify applications name
bin/dokku apps:create <your-app-name>
# Specificly for my project, I'm using PostgreSQL database so I need to use
# Postgres plugin to manage services. I would suggest to use the same name
# as for application itself when you create postgres database.
bin/dokku postgres:create <your-app-name>
# Then we need to link our newly created Postgre database with our application
bin/dokku postgres:link <your-app-name> <your-app-name>
# Using `domains` argument we can configure applications domain, here I'm
# setting subdomain for application.
bin/dokku domains:add <your-app-name> <name>.<server>
# Configure environment variables, like secrets, app specific configurations etc...
# here we are also setting `BUILDPACK_URL` which was mentioned earlier.
# My application uses Github Oauth, so I need to specify some app specific environment
# variables
bin/dokku config:set <your-app-name> $(cat << EOF
# Get rid of a locale-related warning ("warning: the VM is running with native name encoding of latin1 which may cause Elixir to malfunction as it expects utf8.
# Please ensure your locale is set to UTF-8 (which can be verified by running "locale" in your shell)")
LC_ALL=en_US.utf8
BUILDPACK_URL=https://github.com/heroku/heroku-buildpack-multi
SECRET_KEY_BASE=`openssl rand -hex 64`
HOSTNAME=<name>.<server>
MIX_ENV=prod
GITHUB_CLIENT_ID=<client_id_from_github>
GITHUB_CLIENT_SECRET=<client_id_from_github>
EOF
)
{% endhighlight %}

#### Update your Project code

For security reasons we are storing sensitive information in environment variables and we are setting them with `bin/dokku config:set`.
After that we should edit our Phoenix application to use these environment variables.

I would like to suggest to remove file `config/prod.secret.exs` and move this configuration to `config/prod.exs` as we will be using environment
variables. After that don't forget to remove line `import_config "prod.secret.exs"` within `config/prod.exs`.

Move `secret_key_base` option to your Endpoint configuration and use `System.get_env` and get configuration from environment variable:

{% highlight elixir linenos %}
config :your_app, YourApp.Endpoint,
  secret_key_base: System.get_env("SECRET_KEY_BASE"),
  (... all other options here ...)
{% endhighlight %}

And configure your database Repository using `url` option which also get from environment variables (it is automatically set
when you were runing dokku postgres related plugin commands):

{% highlight elixir linenos %}
config :your_app, YourApp.Repo,
  adapter: Ecto.Adapters.Postgres,
  url: System.get_env("DATABASE_URL"),
  pool_size: 20
{% endhighlight %}

#### Deployment

Now to the deployment process. You will need to initialize Git project within your project folder if it is not already done. You
can do it by running `git init`. After that you will need to add your dokku applications git repository as remote.

{% highlight bash linenos %}
git remote add dokku <user>@<server>:<your-app-name>
{% endhighlight %}

And when you want to deploy your application, you push your master branch to `dokku` remote you have just added.

{% highlight bash linenos %}
git push dokku master
{% endhighlight %}

After running `git push` command you should see that deployment process triggers. When deployment will be finished, you will be able
to open your project in web browser. And that's all magic to deploy Phoenix application on Dokku.

What do you think about Dokku deployment process? Is it hard, easy? I really enjoy deploying apps with Dokku, it makes development
much more easier for me! :wink:

[elixir-lang]:                     http://elixir-lang.org/
[phoenix-framework]:               http://www.phoenixframework.org/
[rubyonrails-framework]:           https://rubyonrails.org/
[dokku-github]:                    https://github.com/dokku/dokku
[digital-ocean-dokku]:             https://www.digitalocean.com/products/one-click-apps/dokku/
[heroku-buildpack-elixir]:         https://github.com/HashNuke/heroku-buildpack-elixir
[heroku-buildpack-phoenix-static]: https://github.com/gjaldon/heroku-buildpack-phoenix-static
[procfiles]:                       http://dokku.viewdocs.io/dokku/deployment/methods/dockerfiles/#procfiles-and-multiple-processes
[appjson]:                         http://dokku.viewdocs.io/dokku/advanced-usage/deployment-tasks/#appjson-and-scriptsdokku
