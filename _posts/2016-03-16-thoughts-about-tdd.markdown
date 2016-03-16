---
layout: post
title:  "Thoughts about TDD"
date:   2016-03-16 22:13:00 +0200
categories: testing thoughts tdd
comments: true
disqus_identifier: BF1EBD2C-75DF-40A3-BDBE-93EE9A51D852
---

Hello!

In this short blog post I would like to share my thoughts about [TDD][tdd]{:target="blank"}. Maybe some of you can relate to this.

I would like to start doing TDD, but to be honest, somehow I just don't feel skilled enough or I'm just a little scared to try it.
So as I've also noted in the [about page][about-page] that's one of the reasons I'm writing this blog.
I hope this journey will eventually lead me to gaining confidence in testing, so I would feel more comfortable using TDD approach in development.

I often catch myself thinking:

> "John, this project is 6+ years up & running. Even more experienced developers don't do that.
> The systems' business logic is so complicated, that even after a year and a half you still don't know a lot about this system. You will not succeed with TDD here!
> You already see that this system is depending heavily on Database, you can't even run tests locally without an internet connection.
> Sometimes It's so complex, that you spend a hell of a lot of time mocking and stubbing before actually testing simple classes, objects and methods.
> Write your tests after your code is performing as expected."

Does anyone feel the same? Do you use TDD in your projects? What is you experience in starting with TDD?

Currently I have found two pretty good blog posts about TDD:

- [Quick Left - "Why We Use Test-Driven Development"][quickleft-tdd]{:target="blank"}
- [DHH - "TDD is dead. Long live testing."][dhh-tdd-dead]{:target="blank"}

In short, the first post is talking about benefits of TDD approach and how they use it in their projects. And on the other hand
there is the [DHH's][dhh-twitter]{:target="blank"} (creator of Ruby on Rails) blog post about TDD being dead...

[tdd]:           https://en.wikipedia.org/wiki/Test-driven_development
[dhh-tdd-dead]:  http://david.heinemeierhansson.com/2014/tdd-is-dead-long-live-testing.html
[dhh-twitter]:   https://twitter.com/dhh
[quickleft-tdd]: https://quickleft.com/blog/use-test-driven-development-tdd/
[about-page]:    {{ site.url }}/about
