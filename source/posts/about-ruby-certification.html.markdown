---
title: "About Ruby Certification"
date: "2019-02-28 13:21 UTC"
desc: "A few months ago, I've decided to take the ruby cerfication. The following article is a summary of my experience."
tags:

---

# Intro

Ruby has an official certification provided by Ruby Association, chairman of which is Yukihiro Matsumoto.
I can't say for Japan, but in the rest of the world, it's not very popular. Either it's because Ruby is a community-driven language,
where the knowledge and achievements of an individual programmer are characterized by his or her contributions to Open Source and work experience. Maybe it's because Ruby Association doesn't do anything to promote it. Most of the pages on the official website are in Japanese and you can hardly find any resources on the web on how or why you should take it.

A few months ago, inspired by Egor Bugayenko's [article](https://www.yegor256.com/2014/10/29/how-much-do-you-cost.html#certifications) and [talk](https://www.youtube.com/watch?v=SXd_Ccta1c8) about programmers worth _(a provoking theme and title for sure, but it's nevertheless very useful)_, I've decided to take the ruby certification. The following article is a summary of my experience.

# About the Certification

Currently, Ruby certification consists of two tests: silver and gold. Another one, platinum is still in development. You'll have 90 minutes to answer 50 multiple choice questions. A passing score is 75% and wrong answers don't affect it, which means you have to answer correctly at least 38 questions.

Tests cost $100 each and are taken in Prometric test centers. They are located all around the world, so there shouldn't be any problem to find one in your area. Right after the completion, you’ll get the results and in a week a digital certificate will be emailed to you.

Here is how certificates look: [Silver](https://www.credential.net/k6zuqudp)/[Gold](https://www.credential.net/s0bd99gu)

# Hey, why do I need to take the certification, again?

Before taking the certification I was quite suspicious about it and had a few questions:

### Can it really tell if I know Ruby well?

I'd say that both tests are good at what they aim to do. As you'll see later, questions aren't simple. They test variety of different aspects of ruby both in wide and depth.

### Aren't it's just an enterprise thing to make your CV look shiny?

Well, your CV will look shinier. And you'll be also able to say that you're certified ruby developer.

### But will anyone actually care?

I don't know. Probably not, but I learned a lot during preparation. So, I'd say it was worthwhile.

# Preparation

## Silver

Silver test covers the basics of ruby. Some basic edge cases of syntax. And not so well known methods of most popular classes. They may not appear in your day to day work, but you'll definitely encounter them sooner or later.

I can't really write the exact questions, but here is an example to give you an idea:

> Let's suppose you've defined a constant, what will happen if you try to redefine it?

In real life, you won't ever need to change constant _(I mean, that what constant means, right?)_. In ruby, however, you **can** change it. So, even though it's not very useful, it's a good thing to know how your tools work and behave.

[Ruby certification site](https://www.ruby.or.jp/en/certification/examination/) has a list of topics that tests cover. In my opinion, It's rather broad and unclear. So for actual questions, you need to look at a prep test.

There are two prep tests you can use. One can be found on [Ruby Association website](https://www.ruby.or.jp/assets/images/en/certification/exam_prep_en.pdf) _(it's an old version with only 45 questions, but is still helpful)_ and one on [github](https://github.com/ruby-association/prep-test/blob/master/silver.md). These tests are really good, they provide an explanation to answers and show you **exactly** what would you expect during an actual test.

I suggest you to take prep tests and if you find out that you scored 85-90%, you are all set and ready for the test. During the certification, my score was just a little below the prep one _(5-7%)_.

### What if you don't want to take the test?

If you don't believe in certifications or just don't want to spend $100, I still encourage you to take a prep test. It's a great learning material. It will show you the gaps in your knowledge and generally a really fun thing to do.

One more thing. If you conduct an interview and not really sure what questions to ask. Pick a few from the tests. They are good 👌

## Gold

That's a trickier one. It has lot's of questions on completely different topics: [Fibers](/why-on-earth-do-fibers-exist.html), meta programming, [method visibility modifiers](/what-protected-actually-does.html), exceptions, exception ancestry chain, catch/throw, use of Enumerable and Comparable, Refinements.

A few books I've read previously that were of great help during the test are [Exceptional Ruby](http://exceptionalruby.com/), [Confident Ruby](http://www.confidentruby.com/), [Metaprogramming Ruby](https://pragprog.com/book/ppmetr2/metaprogramming-ruby-2). If you haven't read these, I encourage you to. Even if you don't plan to take the test. These are generally really good Ruby books that will boost your understanding and knowledge of the language.

As a silver test, gold one also has [a prep test](https://github.com/ruby-association/prep-test/blob/master/gold.md), but it doesn't contain the full range of questions you'll face during an official examination. So take prep test, see where your gaps are and study them carefully.

Overall, Gold Certification is a lot more fun, challenging and rewarding experience.

# Registration

When you're ready to take a test. Time for registration. Just go to a [Prometric website](https://www.prometric.com/en-us/clients/ruby/Pages/landing.aspx), choose the certification you want, and book your time and place.

And that's it. All you'll need now is to arrive at the test center at a chosen date and take the test.

# Conclusion

In the end, I’m glad I took the certification. I learned a lot along the way and it’s so fun to discover new things in tools you’ve known for years. Some things I’ve learned I’ll only need in job interviews, but some are very helpful and I started to use in my code right away.
