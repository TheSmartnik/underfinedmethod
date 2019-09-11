---
title: "Breaking apart inheritable mattr_accessors in HTTParty"
date: "2019-07-04 20:42 UTC"
desc: "A weird magic feature I uncovered while debugging HTTParty"
tags:

---

# A land of sharp of knives

We all know that Ruby provides you with [sharp knives](https://m.signalvnoise.com/provide-sharp-knives/). Used with caution they give you great power, but misuse them and you will be remembered at your job long after.

I like some of those, from outside they look like magic, but inside they often are incomprehensible and hard to debug metaprogramming mess.

And in this post, I want to dig into one of those: inheritable attr_accessors.

# A little background

A conventional use of `HTTParty` is to `include` it in your `class`. It allows to create nice and simple wrappers around almost any API and perfect for little API gems. Something like.

```ruby
class ApiClient
  include HTTParty

  base_uri "example.com"

  headers(content_type: 'application/json')

  def clients
    get :clients
  end
end
``` 

But what if wanted to inherit from `ApiClient`? The obvious behavior would be for a child class to inherit parent's class-defined options.

As you might have already guessed, there is no straightforward way to do this.

# Here comes the dragon

```ruby
    module ModuleInheritableAttributes
    def self.included(base)
      base.extend(ClassMethods)
    end

    module ClassMethods
      def mattr_inheritable(*args)
        @mattr_inheritable_attrs ||= [:mattr_inheritable_attrs]
        @mattr_inheritable_attrs += args

        args.each do |arg|
          module_eval %(class << self; attr_accessor :#{arg} end)
        end
      end

      def inherited(subclass)
        super
        @mattr_inheritable_attrs.each do |inheritable_attribute|
          ivar = "@#{inheritable_attribute}"
          parent_value = instance_variable_get(ivar).clone
          subclass.instance_variable_set(ivar, parent_value)

          if parent_value.respond_to?(:merge)
            method = <<-EOM
              def self.#{inheritable_attribute}
                 #{ivar} = superclass.#{inheritable_attribute}.merge(Marshal.load(Marshal.dump(#{ivar})
                 )
              end
            EOM
            subclass.class_eval method
          end
        end
      end
    end
```
I modified a code a bit. Instead of `Marshal.load(Marshal.dump()` we now use a `hash_deep_dup` borrowed from `ActiveSupport`. It makes code even more complex and not relevant to our discussion here, so I replaced it.

It's then used like so

```ruby
module HTTParty
  def self.included(base)
    base.include ModuleInheritableAttributes
    base.mattr_inheritable(default_options)
    base.instance_variable_set("@default_options", {})
  end
end
```

As you can see, use of the code is simple and easy. Code that achieves that, however... Well it's complex, to say the least.


#Let's take a closer look to see how it works

## mattr_inheritable

```ruby
def mattr_inheritable(*args)
  @mattr_inheritable_attrs ||= [:mattr_inheritable_attrs]
  @mattr_inheritable_attrs += args

  args.each do |arg|
    module_eval %(class << self; attr_accessor :#{arg} end)
  end
end
```

1. Initializing a _class instance variable_ to hold our inheritable attributes including the variable itself.
2. Adding our new attribute to the array
3. Adding accesor to our **module**. Notice `module_eval` here, we evaluate our code in the context of the module. So we are adding accessor not to an instance, but to a module

## inherited

```ruby
def inherited(subclass)
  super
  @mattr_inheritable_attrs.each do |inheritable_attribute|
    ivar = "@#{inheritable_attribute}"
    parent_value = instance_variable_get(ivar).clone
    subclass.instance_variable_set(ivar, parent_value)

    if parent_value.respond_to?(:merge)
      method = <<-EOM
        def self.#{inheritable_attribute}
           #{ivar} = superclass.#{inheritable_attribute}.merge(Marshal.load(Marshal.dump(#{ivar})
           )
        end
      EOM
      subclass.class_eval method
    end
  end
end
```

1. Starting to iterate over our inheritable attributes
2. Getting variable value from parent
3. Setting the value to child 
3. Checking the value respond to `#merge`.
3. Here, with `class_eval` we redefine our attr reader to always get parent values and then merge with them with values of a subclass. It's needed for cases when parent options were changed *after* our child class was evaluated.

That's it 😊

# A land of magic

While reading this you might have thought: "God, what a mess!". And that's true, but it also allows for some clean code on the user's side.

Whether "magic" is always bad is a controversial topic. Rails have been criticized for years because of it. However, it is also the reason why Rails became so popular in the first place. It's the reason why many of us fell in love with Ruby.

So, as with every other decision in programming, there is always a trade-off one has to make. When you create a library your objective is to create an easy to use tool and for that, a little magic is sometimes necessary.


# Update 11.09.2019

[Janko Marohnić](https://github.com/janko) kindly [suggested](https://www.reddit.com/r/ruby/comments/chiz4j/breaking_apart_inheritable_mattr_accessors_in/) a refactored, simplified version of the code using [module builder pattern](https://dejimata.com/2017/5/20/the-ruby-module-builder-pattern)

```ruby
  class MattrInheritable < Module #:nodoc:
    def initialize(*names)
      attr_accessor *names

      define_method :inherited do |subclass|
        names.each do |name|
          super(subclass)
          subclass.send(:"#{name}=", send(name))

          subclass.class_eval <<-RUBY
            def self.#{name}
              @#{name} = superclass.#{name}.merge(MattrInheritable.hash_deep_dup(@#{name}))
            end
          RUBY
        end
      end
    end
  ```

That can be used later used like so

```ruby
base.extend MattrInheritable.new(:default_options)
```
