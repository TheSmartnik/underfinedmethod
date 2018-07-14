---

title: Doing the right thing
date: 2018-06-14 14:11 UTC
desc: And now it's time for a post on virtues of programming
tags:

---

Once in a while, I remember that I should be a professional and not try to hack every problem I face, rather actually try to solve it. So this post is about exactly that, how I properly fixed a problem.

So, looking through [an official ruby documentation](https://ruby-doc.org/core-2.5.1/Array.html#method-i-concat), I've noticed that links to `Array#+` are broken.
With a thought, that soon I be able to tell, that I've contributed to Ruby, I opened up a source code and... Nothing.
[Documentation to a method](https://github.com/ruby/ruby/blob/trunk/array.c#L3706) was fine. Okay, maybe it's not possible to link to symbol methods at all, thought I. No, documentation's [sidebar](https://ruby-doc.org/core-2.5.1/Array.html#method-list-section) links them perfectly. Wait... Did you guess what my first idea was?

Right, I thought I'd just you copy-paste html id's from sidebar and change documentation to something like:

```C
/*
 * ...
 *  See also {Array#+}[#method-i-2B].
 */
```

I had no idea what this id means, how it's generated or may it change, but it did work. Just need to make sure this id won't get changed by some unforeseen events. Well, it seems that [it should be fin](https://github.com/ruby/rdoc/blob/b7449e4b2d71c16e58f5a5db859867e8c90246ac/lib/rdoc/method_attr.rb#L291-L295). So I can totally hard code this methood url to fix documentation, right?

---

Unfortunately, I can't really tell what stopped me from doing this quick fix and instead read for six hours rdoc's [very](https://github.com/ruby/rdoc/blob/b7449e4b2d71c16e58f5a5db859867e8c90246ac/lib/rdoc/markup/to_html_crossref.rb#L147-L154) [interesting](https://github.com/TheSmartnik/rdoc/blob/0e655546b4d3f9d00982f12f87f968dd5b96103a/lib/rdoc/cross_reference.rb#L31-L73) source code to make a [proper one-line fix](https://github.com/ruby/rdoc/pull/632). But I glad I did, feels really good to do the right thing.

_Wait, but how do you know what is the right thing to do?_
