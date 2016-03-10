---
layout: post
title:  "Testing ActiveRecord associations with RSpec"
date:   2016-03-02 20:05:11 +0200
categories: testing rspec activerecord associations
comments: true
disqus_identifier: B269C223-AA9A-4E47-9EA9-6CECE731CFE9
---

Hi!

One day I was doing my usual everyday development with Ruby on Rails. My task was to add some kind of ActiveRecord associations and some options for them,
a few new methods, which were using new associations and a coulpe more things.

When these associations and options were implemented, I jumped to write unit tests in [RSpec][rspec]{:target="blank"}
(Yes, usually we don't do [TDD][tdd]{:target="blank"} in our project, just cover all functionality soon after code is written and some demos are
provided. But in some cases, if a bug pops out, we do use [BDD][bdd]{:target="blank"} or TDD approach when testing manually would take too much time and effort).

So I started thinking: How can I test these newly created associations? Do I even test these associations?
I begun digging in project files in hopes to find some earlier approaches of testing associations. And I did find some. Those examples were checking that
the corresponding models' class method: [.reflect_on_association][reflect-on-association]{:target="blank"} doesn't return `nil`.

By inspecting this class method `.reflect_on_association` and also searching on Google, I confirmed, that this could be the way to do it. Playing around with it and
cheking it's API revealed that this method returns `AssociationReflection` object. It is also possible to check what kind of `macro` is used (`:has_many`,
`:belongs_to`, etc...), and also discover association `options`, like, `:dependent`, `:through` and others. So after a while I ended up using the same approach for testing my newly created associations.

Here are some simple examples of this approach (by the way, it took me a while to set up ActiveRecord for use without the Rails framework).
These examples contain simple `Post` and `Comment` associations and their option check (disclaimer: *this is not code from the project, just a dummy example*).

{% highlight ruby %}
describe Post do
  describe "association" do
    context "with Comments" do
      let(:association) { described_class.reflect_on_association(:comments) }

      describe "macro" do
        subject { association.macro }

        it { is_expected.to eq(:has_many) }
      end

      describe "options" do
        subject(:options) { association.options }

        it { expect(options).to include(:dependent) }

        describe ":dependent" do
          subject { options[:dependent] }

          it { is_expected.to eq(:destroy) }
        end
      end
    end
  end
end
{% endhighlight %}

{% highlight ruby %}
describe Comment do
  describe "association" do
    context "with Post" do
      let(:association) { described_class.reflect_on_association(:post) }

      describe "macro" do
        subject { association.macro }

        it { is_expected.to eq(:belongs_to) }
      end
    end
  end
end
{% endhighlight %}

You can find the source code on [my GitHub page][example-link]{:target="blank"}.

I happened to stumble upon different opinions on this topic, while trying to figure out if it's actually a thing - association testing.
Should we do these association tests or should we not? While some prefer not to do them, by saying that ActiveRecord associations are tested well,
I don't mind testing them, as Im not testing ActiveRecord associations as such, just checking for associations between models in my project.

As I'm not really experienced in testing at the moment of writing this post, I will just stick to that and check on it later judging by my experience.
Till now, this approach has helped to discover a mistake once, so it's definitely working for me.

This takes some additional lines of test code, but there's a shorter way to do that, which I will probably tell you about in one of my later posts.

[rspec]:                  https://github.com/rspec/rspec
[tdd]:                    https://en.wikipedia.org/wiki/Test-driven_development
[bdd]:                    https://en.wikipedia.org/wiki/Behavior-driven_development
[reflect-on-association]: http://api.rubyonrails.org/classes/ActiveRecord/Reflection/ClassMethods.html#method-i-reflect_on_association
[example-link]:           https://github.com/janis-vitols/examples/tree/rspec/activerecord/associations
