---

title: ActiveJob In Retrospect
date: 2018-07-12 14:11 UTC
desc: ActiveJob has been around for almost 4 years now. But was it a good idea in the first place?
tags:

---

Recently, I replaced `ActiveJob` with `Resque` _(why it's resque and not sidekiq is a whole other matter)_, which got me into thinking of ActiveJob usefulness as a whole.

Creating `ActiveRecord` was a very reasonable thing to do. It's enough to look at connection adapters for [postgres](https://github.com/rails/rails/blob/master/activerecord/lib/active_record/connection_adapters/postgresql/database_statements.rb) and [mysql](https://github.com/rails/rails/blob/master/activerecord/lib/active_record/connection_adapters/mysql/database_statements.rb) to see why.

1. Lots of methods

2. Quite a different implementation

3. Hard to remember

4. Hard to switch to projects, where a database is not the one you're used to.

5. And of course, it gives an ability to switch databases without changing code (unless you use db specific features as most of us do)

Now let's look at ActiveJob's adapters.

#### Sidekiq

```ruby
class SidekiqAdapter
  def enqueue(job) #:nodoc:
    job.provider_job_id = Sidekiq::Client.push(
      "class" => JobWrapper, "wrapped" => job.class.to_s,
      "queue" => job.queue_name, "args" => [ job.serialize ]
    )
  end

  def enqueue_at(job, timestamp) #:nodoc:
    job.provider_job_id = Sidekiq::Client.push(
      "class" => JobWrapper, "wrapped" => job.class.to_s, "queue" => job.queue_name,
      "args" => [ job.serialize ],"at" => timestamp
    )
  end
end
```

#### Resque

```ruby
    class ResqueAdapter
      def enqueue(job) #:nodoc:
        JobWrapper.instance_variable_set(:@queue, job.queue_name)
        Resque.enqueue_to job.queue_name, JobWrapper, job.serialize
      end

      def enqueue_at(job, timestamp) #:nodoc:
        unless Resque.respond_to?(:enqueue_at_with_queue)
          raise NotImplementedError, "To be able to schedule jobs with Resque you need the " \
            "resque-scheduler gem. Please add it to your Gemfile and run bundle install"
        end
        Resque.enqueue_at_with_queue job.queue_name, timestamp, JobWrapper, job.serialize
```

And that's it, just two methods. Do we really need an abstraction layer for just two methods?

Sometimes yes, but do we really need an abstraction layer that prevents us from using [features](https://github.com/lantins/resque-retry/issues/140) without monkey patching _(not to say it's an absolute evil, but it rather error prone)_ or adds unnecessary [performance overhead](https://github.com/mperham/sidekiq/wiki/Active-Job#performance)?

---

Rails is very good at tools that, provided out of the box, just work. Minimum if any configuration and you all set. However, setting up any background job framework is simple. Just five lines of code for sidekiq and a little more for resque.

Another feature that `ActiveJob` provides is an ability to change background processing backend  with one line of code. Although, I think it's a rare use case. I used it once to switch from resque to sidekiq to resque again. And it wasn't one line of code. With `ActiveJob` the only thing I avoided is a few global 'Find And Replace' over the codebase.

When something is a default and works out of the box, there is a lot of inner resistance in replacing it. `ActiveJob` brings little agility, gives another _default_ interface to learn how to work with and how to test properly. It also adds the possibility that some external gems won't work and you would have to somehow monkey patch this gem to make it work or get rid of `ActiveJob` altogether. All this and not much profit in return.
