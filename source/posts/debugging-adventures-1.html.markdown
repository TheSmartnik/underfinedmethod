---
title: "Debugging Adventures: #1"
date: "2018-10-24 13:21 UTC"
desc: "I love debugging. You know, the one that makes you dig into the source code of a library or a language. When everything should work just fine, but it doesn't. Well it's a small story just about that."
tags:

---

I love debugging. You know, the one that makes you dig into the source code of a library or a language. When everything should work just fine, but it doesn't. When you have no idea what to do and desperately executing same commands hoping for a different result. And it is an adventure really: you kinda know the path, but have no idea where it will lead you.

Anyway, a few months ago I've been looking through [an official ruby documentation](https://ruby-doc.org/core-2.5.1/Array.html#method-i-concat) and I had noticed that links to `Array#+` are broken. Immediately after that, encouraged by the thought, that soon I'll be a ruby contributor _(no one need to know it's been just documentation, right?)_ I opened up a source code and... Nothing. Documentation to a method looked just fine.

And that point, I knew it won't be as easy as I initially thought.

I wasn't really sure what to do at first. Then, a small detail caught my attention: a sidebar. A sidebar on the left of the page, where a link to `#+` worked just fine. 

Well, that looks great, I thought. I'll just copy the link from sidebar and change documentation to something like.

```C
/*
 * ...
 *  See also {Array#+}[#method-i-2B].
 */
```

I had my doubts, of course. I wasn't sure how `i-2B` is generated and can get be changed by some unforeseen events. I did a bit of digging into rdoc's source code and [it seemed to be fine](https://github.com/ruby/rdoc/blob/b7449e4b2d71c16e58f5a5db859867e8c90246ac/lib/rdoc/method_attr.rb#L291-L295). So I've decided that'll do.

Reading this, you probably saying to yourself: "Wait a minute, this just looks like a hack. Why do you have to hard code a link? Why can't it be generated automatically?".

Well, that was my thought exactly. I tried to convince myself that the solution above is good enough and working link better, than not a working one etc. But I couldn't really do it, so I decided to take a quick look. Maybe it would be a quick fix, who knows.

So, a few moments after, I was staring at Rdoc's source code. And I continue doing so for six straight hours. I don't know if you aware, but Rdoc code isn't [the most readable](https://github.com/ruby/rdoc/blob/b7449e4b2d71c16e58f5a5db859867e8c90246ac/lib/rdoc/markup/to_html_crossref.rb#L147-L154). I mean it does [raise questions](https://github.com/TheSmartnik/rdoc/blob/0e655546b4d3f9d00982f12f87f968dd5b96103a/lib/rdoc/cross_reference.rb#L31-L73).

And in the end? Well the whole fix, just took [one line of code](https://github.com/ruby/rdoc/pull/632).

Why am I telling this? Well, I feel like debugging is often daunting and  frustrating: nothing is working and you don't know why. But if you look at this from another perspective, it also can be fun and adventurous. Your princes will probably be in another caster most of the time, but it should never stop you from trying to save her 😁
