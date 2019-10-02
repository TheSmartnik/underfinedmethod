---

title: Why on earth do fibers exist?
date: 2018-11-23 14:11 UTC
desc: Are you ready for the truth?
tags:

---

First, let's answer the question right away. Fibers were created so one could implement generator pattern which in ruby is incorporated in `Enumerator`. That's it.

To quote [one book:](https://www.amazon.com/dp/0596516177)
> Fibers are an advanced and relatively obscure control structure; the majority of Ruby programmers will never need to use the Fiber class directly

Now that you know the truth. Let's start from the beginning.

## <a name="a-bit-of-history"></a> A bit of history

Back in ruby 1.8 [generator was implemented through continuations](https://github.com/ruby/ruby/blob/ruby_1_8_7/lib/generator.rb), little-known control structure inspired by Lisp. Continuations main use is again to make generators and programs that needed [backtracking:](https://en.wikipedia.org/wiki/Backtracking) [ambiguous operator](http://www.randomhacks.net/2005/10/11/amb-operator/), for example. It has bugs,  unpredictable behavior and implemented only in CRuby. Very few understand what it is for and even fewer actually tried to use it.

Then, ruby 1.9 with YARV came along. Continuations became to be [harmful](http://www.atdot.net/~ko1/pub/ContinuationFest-ruby.pdf) and were moved to the standard library. `Enumerator` was rewritten in C using fibers.

## Fibers

In other languages, fiber is the name for a lightweight thread. In ruby, it is a coroutine. Why it's not named coroutine then, you might ask? Well, because fiber sounds better [apparently](http://www.atdot.net/~ko1/pub/ContinuationFest-ruby.pdf)(Page 20).

There are two types of coroutines: semicouroutune and coroutine. They only differ in the way they transfer control.

### Semicouroutune a.k.a Asymmetric Coroutines

These coroutines called asmymetric, because there is fundamental asymmetry between caller and coroutine. _Resumed_ by the caller, coroutine **can't** transfer control to any other coroutine, only to suspend itself and _yield_ control back to the caller.

This is the default mode for ruby fibers.

### Symmetric Coroutines

If you `require 'fiber'`, though. `Fiber` becomes symmetric coroutine, which basically means that now, fibers can transfer control between one another _(there are limitations in ruby, though. But here is the basic idea)_

Okay, now when we become familiar with fibers and coroutines. Let's look at the thing you can build with them.

## Generator

### Why would I need generators, again?

I'll be completely honest. You probably don't.

Having said that, they are pretty cool. They are generally useful for partial computations or laziness. In ruby, there are two options to make something lazy: through `Enumerator#next`(which uses fibers) and through `Enumerator::Lazy`(which doesn't)

### Partial computations

Here is the basic generator's algorithm:
  
  1. Compute a partial result
  2. Return the result back to the caller
  3. Save the state
  4. If needed, resume the generator to get the next result

Let's see an example with enumerator:

```ruby
enumerator = Enumerator.new do |yielder|
  i = 0
  loop do
    i += 1
    yielder << i
  end
end
# => #<Enumerator: #<Enumerator::Generator:0x00007fe6ac9d41e0>:each>

enumerator.next   # => 1
enumerator.next   # => 2
enumerator.next   # => 3
enumerator.rewind
enumerator.next   # => 1
```

Yeah, I know. That example probably doesn't raise your eyebrows, except for the syntax, of course. I mean use `<<` to return control, really?
By the way, there is an even more confusing alias for that -- `yield`. It corresponds to `Fiber.yield`, but has nothing to do with ruby keyword `yield`.

Anyway, let's see how it works.

1. You pass a block, where you do computation
2. On `#next` the block is called from the beginning or where it left of
3. You return from the function with a result using `<<`
4. Go back to step .2

`Enumerator` allows you to do two types of iterations: internal and external.
 In daily life, we usually see enumerators as internal iterators. It is returned everytime you call `Enumerable` methods without any arguments and it allows you to chain those methods together: `each.with_index` etc.

Additionally, you can use enumerator as an external iterator as shown in the example above. This construct might be useful for heavy computations, where saving the previous state would save a lot of time and doing so with `Enumerator` would also be more readable than saving the previous state yourself. Unfortunately, it's not a very popular method _(as laziness in ruby in general)_ and I couldn't find any good examples of it in the wild 😞


## The Fun Part

You've read a lot of theory, not it's time for _the fun part_. The best way to learn something is to build it, right? Also, it's probably the only time you'll ever actually use fibers, so let's get to it.

### Generator

Ok, let's look at the `Enumerator` example again. What we can say about it?
  
  1. Enumerator accepts block
  2. The block is being called with an argument
  3. The argument has a method `<<` that returns computation result

That's pretty much all we need to be able to construct a simple abstraction over fiber. We'll call it `Generator` because that's what it is.


```ruby
class Generator
  class Yielder
    def <<(x)                         # Argument that accepts <<
      Fiber.yield(x)                  # And returns computation result
    end
  end

  def initialize(&block)              # Accepts block
    @block = block           
    @fiber = create_fiber
  end

  def next
    @fiber.resume             
  end

  def rewind                  
    @fiber = create_fiber
  end

  private
  
  def create_fiber            
    Fiber.new do
      @block.call(Yielder.new)        # Block being call with an argument
    end
  end
end
```

That's it. Here is the tiniest possible version of a generator.

Let's test that it works as expected. We'll use `Enumerable` this time.

```ruby
generator =  Generator.new do |yielder|
  (1..Float::INFINITY).each do |i|
    yielder << i
  end
end

generator.next    # => 1
generator.next    # => 2
generator.next    # => 3
generator.rewind 
generator.next    # => 1
```

## Summary

For a long time, I couldn't understand why fibers exist. I thought there was something special about them, something that I just couldn't grasp. Turned out there wasn't. They are very specific things, created for a very specific task.

I hope that this article gave you a better understanding of what `Fiber` is, what it's not and how it's used.

## Update about concurrency

Remember that fibers in ruby have two mods? I haven't really talked about fibers as symmetric coroutines and one redditor [pointed that out](https://www.reddit.com/r/ruby/comments/a0ivny/why_on_earth_do_fibers_exist/eai2k67/).

Given an ability to transfer control between one another, fibers can be used to write asynchronous concurrent code. To do this you'll need a library that implements event loop such as [async](https://github.com/socketry/async) or [eventmachine](https://github.com/eventmachine/eventmachine) or you can even build your own simple reactor with ruby `io/nonblock`.

When it is useful? In tasks that require a lot of io such as web requests. There are two great examples of this: [goliath](https://github.com/postrank-labs/goliath) webserver that uses `eventmachine` and [falcon](https://github.com/socketry/falcon) build on top of the `async`.

I tried to give a brief explanation here, so nothing was left out and you had a complete picture. While it's not in depth look at fibers as means for concurrency, I hope you now better understand how they can be utilized.
