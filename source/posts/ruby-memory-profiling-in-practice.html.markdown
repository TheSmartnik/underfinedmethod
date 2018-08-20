---

title: Ruby Memory Profiling in Practice
desc: A recent task at work inspired me to write a guide I was trying to find for a long time.
date: 2018-08-19 14:09 UTC
tags: 

---

When I only started programming, I loved tasks related to profiling and optimization.
However, my knowledge on this subject was very limited and I desperately searched for articles with some tips and tricks on how to profile properly.
I thought there were some secrets or techniques, that I should know. A few years forward, I can say there a none, really 🤷‍♂️.

But here are some tips to give you confidence.

## Basic steps

Profiling itself is very easy and consists of four basic steps

1. Profile

2. Find bottleneck

3. Fix bottleneck

4. Profile again 

## How to profile

Ruby has a rich set of decent profiling tools. Some would disagree, but in my opinion, they are good enough.

Depending on a type of problem you have, you'll need a different type of profiler. We are talking about memory here, so we'll take [memory_profiler](https://github.com/SamSaffron/memory_profiler).

### Splitting the work

Profiling itself requires a lot of memory. Therefore, if you start to profile the whole process, you'll probably run out of memory pretty quick.

There is a general solution to this. Divide process into few major parts and profile each of them one by one.

*It may seem obvious, but I feel like it's an important thing to note.*


## How to find and fix a bottleneck

1. Look at the report.

2. Find the code that takes the most memory.

3. Look at the code. Does it create any unnecessary objects? Can you rewrite it to allocate less memory?

4. Optimize it if it's possible. If not, go to the next piece of memory-heavy code in the report.

### Any unnecessary objects?

Here is a couple of examples to illustrate what I mean. I recently had a problem with middleman. It constantly used more memory than my 512 MB dyno allowed. So I had to find a solution. 

#### Middleman

Middleman created extra array each time I ignored the object.

```diff
- resources.map do |r|
+ resources.each do |r|
  if ignored?(r.normalized_path)
    r.ignore!
  elsif !r.is_a?(ProxyResource) && r.file_descriptor && ignored?(r.file_descriptor.normalized_relative_path)
    r.ignore!
  end
-  r
end
```

Seems like a very small thing. However, I had thousands of ignored objects. And those little arrays accounted for 10% of memory used during initialization.
Here is [a link to pr](https://github.com/middleman/middleman/pull/2183).

Usually, this is a small win and it doesn't help much, what you're looking for is a big win, like the one below.

#### Middleman S3 Sync

Gem `middleman_s3_sync` first created sync objects and then ignored ones that don't need to be synced. The strategy is ok most of the time, but not in my case of hundreds or even thousands ignored resources. It's very unwise use of resources.

**Before**

```ruby
def manipulate_resource_list(mm_resources)
  ::Middleman::S3Sync.mm_resources = mm_resources
end
```

**After**

```ruby
def manipulate_resource_list(resources)
  ::Middleman::S3Sync.mm_resources = resources.each_with_object([]) do |resource, list|
    next if resource.ignored?
    list << resource
    list << resource.target_resource if resource.respond_to?(:target_resource)
  end
  resources
end
```

This 8 lines of code freed up 200MB of memory. Here is [a link to this little pr](https://github.com/fredjean/middleman-s3_sync/pull/155)

### In a report everything is fine. What should I do?

These a couple of tips that really helped me during profiling.

1. If the report is fine. Double check that data you profile with is the same used in production.

2. Try to disable parts of the code in staging and check if used memory dropped significantly.

---

## No bottlenecks. My code is perfect. Third-party code is perfect. But it takes SO MUCH MEMORY.

Well, if you can't find a room for optimization and everything is fine. The only solution is to rethink the whole approach. Find a different solution to the same problem and rewrite this functionality altogether.

Many people jump to above method without attempting to find an actual problem in their code. I don't really like to do so. Profiling isn't hard and once you find a problem it's usually easy to fix it.

Rewrites are usually taking so much more time. And when you don't try to find problems in your code, you're bound to make the same mistakes again. Optimization tasks are a great way to learn and grow as an engineer.



