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

**Coming Soon** ...
