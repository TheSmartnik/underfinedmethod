---

title: What protected actually does?
date: 2017-11-25 14:11 UTC
desc: Being famillar with C-family languages, you may be surprised to know how protected works in Ruby
tags:
  - ruby
  - middleman
  - static site generation

---

 `protected` visibility isn’t popular, nor should it be. Having surprising and little known behaviour, that will confuse future maintainers, there are only few possible and even fewer justified use cases for it.

## Behaviour
 `protected` methods can be explicitly called on objects, if caller is an **instance** of the same or descendant class .

Here is an example with comparison of `public`, `protected` and `private` visibility modifiers.

```ruby
class Foo

  def public_method
     puts 'public'
  end

  protected def protected_method
    puts 'protected'
  end

  private def private_method
    puts 'private'
  end
end

class FooBar < Foo
  def bar(foobar)
    foobar.public_method #> public
    foobar.protected_method  #> protected
    foobar.private_method  #> NoMethodError
  end
end

class Baz
  def bar(foobar)
    foobar.public_method #> public
    foobar.protected_method  #> NoMethodError
    foobar.private_method  #> NoMethodError
  end
end
```

As you can see from example above, `public` can be called from anywhere. `protected` from and instance of the same or descendant class .  And `private` can’t be called on instance  at all.

### respond_to?
One important thing  to note.
`respond_to?` doesn’t include protected and private methods in search. You should pass second boolean argument to enable this feature.

```ruby
class FooBar < Foo
  def bar(foobar)
    puts foobar.respond_to?(:protected_method) #> false
    puts foobar.respond_to?(:protected_method) #> true
    foobar.protected_method #> protected
  end
end
```

## In the wild
### Huginn. The good.
Other design choices aside. The following snippet is a simplified extract from [Huginn source code](https://github.com/huginn/huginn/blob/master/lib/location.rb). And `protected` here is a rare example of correct usage.

```ruby
class Location < Struct.new(:latitude, :longitude)
  protected :[]=

  def initialize(data = [])
    super()

    self.latitude, self. longitude = data if data.size == 2
  end

  def latitude=(value)
    self[:latitude] = value if value.abs <= 90
  end

  def longitude=(value)
    self[:longitude] = value if value.abs <= 180
  end
end
```
Struct descendant’s can assign variables both through `[]=` method and through explicit setters. Here, as you can see, `[]=` is protected to prevent assigning instance attributes with anything, but redefined setters.

```ruby
location = Location.new
=> #<struct Location latitude=nil, longitude=nil>

location.latitude = 53.1242
=> 53.1242

location[:longitude] = 42.2124
NoMethodError: protected method `[]=' called for #<struct Location latitude=53.1242, longitude=nil>
```

### Grape & Devise. The Bad
#### Grape
Let’s look at an example use of `protected`  in a simple coercion class. I’ll provide code, without comments, full source code [here](https://github.com/ruby-grape/grape/blob/220c345dff9602e431ac780abcb98dbb24293395/lib/grape/validations/types/json.rb#L33-L41)

```ruby
module Grape
  module Validations
    module Types
      class Json < Virtus::Attribute
        def coerce(input)
          return if input.nil? || input =~ /^\\s*$/
          JSON.parse(input, symbolize_names: true)
        end

        def value_coerced?(value)
          value.is_a?(::Hash) || coerced_collection?(value)
        end

        protected

        def coerced_collection?(value)
          value.is_a?(::Array) && value.all? { |i| i.is_a? ::Hash }
        end
      end

      class JsonArray < Json

        def coerce(input)
          json = super
          Array.wrap(json) unless json.nil?
        end

        def value_coerced?(value)
          coerced_collection? value
        end
      end
    end
  end
end
```

Here you can see an example of very common misconception that `private`  methods can’t be called inside child classes and  `protected`  methods should be called instead  _(As in Java_C)/. It is wrong.  You don’t need  `protected`  here, If we replace it with `private` the code will work just fine. And it would be better, because it will be less confusing for a reader.


#### Devise
Repeats the same mistake Grape does. It uses `protected`  in modules, that are intended be included in user generated models and controllers. So, `protected` in this case can be replaced with  `private` as those methods are never called on instance


### Rails. The Ugly
First, in `ActionDispatch::Routing::UrlFor`  module we define a protected method `optimize_routes_generation?`

```ruby
protected

def optimize_routes_generation?
  _routes.optimize_routes_generation? && default_url_options.empty?
end
```

And then later in code call it via `send`  from `ActionDispatch::Routing::RouteSet::NamedRouteCollection::UrlHelper::OptimizedUrlHelper`

```ruby
def optimize_routes_generation?(t)
  t.send(:optimize_routes_generation?)
end
```

The main point, of `protected` is to allow instances of the same and descendant classes to call interior methods, that no one else outside have no need to know about.  If some outside instance needs to call `protected` method, just leave it `public`



