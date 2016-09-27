---
layout: post
title:  "Background Jobs with ActiveJob | Ruby on Rails"
date:   2016-09-27 11:23:32
categories: code
---

Do you have a need for background jobs?
---------------------------------------
If you have tasks such as, **data processing, sending mail, processing files, connect to 3rd party APIs** - then you probably need a background job solution.

Options
-------

Fortunately, on Rails there are many solution to achieve background job processing, though we'll focus here on **[delayed_job]** and **[resque]**.


The main difference between the two is that they use different solution for storing jobs that need to be processed - with [delayed_job] using **SQL Table** and [resque] using **Redis** to store jobs that need to be processed.


Delayed Job
-----------
[delayed_job] is the easiest solution of the two. There is no need to setup another dependency (such as redis), and getting up an running is a breeze.

**Installation**

***Step 1:*** Start by adding this line to your gemfile:

{% highlight ruby %}
gem 'delayed_job_active_record'
{% endhighlight %}

Make sure you run `bundle install` afterwards.

***Step 2:*** Generate Migration
[delayed_job] offers a migration generator that will create the migration automatically for the table needed to store the jobs.

Run this command in your terminal:

`bundle exec rails generate delayed_job:active_record`

Now, let's run the migration we just created:

`bundle exec rails db:migrate`


***Step 3:*** Specify the ActiveRecord adapter on your config/application.rb

Add this line:

{% highlight ruby %}
config.active_job.queue_adapter = :delayed_job
{% endhighlight %}

Here is a snippet by [delayed_job] developers on how to start workers, that will process the jobs:
{% highlight ruby %}
RAILS_ENV=production script/delayed_job start
RAILS_ENV=production script/delayed_job stop

# Runs two workers in separate processes.
RAILS_ENV=production script/delayed_job -n 2 start
RAILS_ENV=production script/delayed_job stop

# Set the --queue or --queues option to work from a particular queue.
RAILS_ENV=production script/delayed_job --queue=tracking start
RAILS_ENV=production script/delayed_job --queues=mailers,tasks start

# Use the --pool option to specify a worker pool. You can use this option multiple times to start different numbers of workers for different queues.
# The following command will start 1 worker for the tracking queue,
# 2 workers for the mailers and tasks queues, and 2 workers for any jobs:
RAILS_ENV=production script/delayed_job --pool=tracking --pool=mailers,tasks:*2 --pool=:2 start

# Runs all available jobs and then exits
RAILS_ENV=production script/delayed_job start --exit-on-complete
# or to run in the foreground
RAILS_ENV=production script/delayed_job run --exit-on-complete
{% endhighlight %}

For more information on delayed_job visit their Github page at: [https://github.com/collectiveidea/delayed_job]

[https://github.com/collectiveidea/delayed_job]: https://github.com/collectiveidea/delayed_job
[delayed_job]: https://github.com/collectiveidea/delayed_job
[resque]: https://github.com/resque/resque


Resque
------

[resque] is a the most used background job processing solution for Ruby on Rails projects. As it's using Redis key-value store, it is also much faster in performing background jobs.

**Installation**

***Step 1:*** Start by adding this line to your gemfile:

{% highlight ruby %}
gem 'resque'
{% endhighlight %}

Don't forget to run `bundle install` after.

***Step 2:*** As mentioned, resque needs Redis, so let's install that by running `brew install redis`. If you don't have brew, you can install it by running:
`/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`


***Step 3:*** Here we don't need to run any migration, so the next step is just specifying the adapter in config/appicaton.rb

{% highlight ruby %}
config.active_job.queue_adapter = :resque
{% endhighlight %}


Examples (Applicable to both solutions)
---------------------------------------

Now that we've chosen either [delayed_job] or [resque], it's time to create our first job/task. Jobs in a Rails >= 4.x.x version app, should be stored under app/jobs directory. We don't need to create these manually, rails offers this generator command:
`bundle exec rails g job example_task`

After running the command, we should now have a new file under app/jobs directory.

We have full access to our full stack of our application, all of it's models, methods. One thing to keep in mind, is that ActiveJob serializes the arguments that are passed to these jobs, so keep in mind, to never pass a full object, make sure you only pass IDs for objects, and fetch them in the job.

Here is a simple job:

{% highlight ruby %}
class SendEmail < ActiveJob::Base
  queue_as :default # We set the queue here, as we can have different workers dedicated to different queues

  def perform(order_id, options = {}) # Note we're only passing an object ID, not the full object.
    order = Order.find(order_id) # Fetching the object now from the DB
    if order.processed? && order.completed? # We have full access to model methods
      order.send_email_to_customers
    end
  end
end
{% endhighlight %}
