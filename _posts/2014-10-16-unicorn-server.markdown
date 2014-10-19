---
layout: post
title:  "Switching Your Rails Application Web Server to 'Unicorn'"
date:   2014-10-16 12:01:27

---
###Introduction

I was listening to a [Ruby Rouges][richard-pod] podcast sometime back and they had guest [Richard Schneeman][richard] who worked at [Heroku][heroku]. Although I 
did not fully understand what he talking about for most of the show.. I remembered him specifically making a point to 
tell the audience to "PLEASE switch your WEBSERVERS from the default WEBRICK to UNICORN".  

Well in my experience Webrick worked just fine..at least for developing with.  So I wasn't too hasty with
switching right away.  But as I was finishing up my first contracted Website a couple weeks ago, I remembered what
Richard had said. So I started to google for tutorials on how to switch over to 'Unicorn'.

Easy right?  

Well maybe to an experienced web developer..but I am not that :(

I found most the tutorials especially from [Heroku-Docs][heroku-docs] were lacking the "Hold My Hand" aspect to them that I kind of need
at this point in my career.  So I decided to create a tutorial that does just that.  To help my future self and anyone
else who is confused by current documentation on the subject. 


###What You Will Learn

This tutorial will show you step by step how to switch your default 'Webrick' web server in a Ruby On Rails Application 
to the 'Unicorn' server. Which is apparently much faster for production purposes on hosting sites like [Heroku][heroku].

##First


You will want to add Unicorn to your app's 'Gemfile'.  Which is located in the 'root' directory of
your application.

`my_rails_app/Gemfile`

 {% highlight ruby %}
	gem 'unicorn' # It might already be in your Gemfile but commented out. If so just remove the '#' before it
{% endhighlight %}

Next, you have to install the gem with bundler.  Run `bundle install` 


##Second

Now you need to create a configuration file for Unicorn in your `my_rails_app/config` directory.
Name the file `unicorn.rb` and add the following to it:

{% highlight ruby %}
# config/unicorn.rb
worker_processes Integer(ENV["WEB_CONCURRENCY"] || 3)
timeout 15
preload_app true

before_fork do |server, worker|
  Signal.trap 'TERM' do
    puts 'Unicorn master intercepting TERM and sending myself QUIT instead'
    Process.kill 'QUIT', Process.pid
  end

  defined?(ActiveRecord::Base) and
    ActiveRecord::Base.connection.disconnect!
end

after_fork do |server, worker|
  Signal.trap 'TERM' do
    puts 'Unicorn worker intercepting TERM and doing nothing. Wait for master to send QUIT'
  end

  defined?(ActiveRecord::Base) and
    ActiveRecord::Base.establish_connection
end
{% endhighlight %}

If you would like to discover what all of this script does, go to the [Heroku Documentation][heroku-docs] page 
on installing Unicorn.  They give a detailed explanation for the basic configuration script above.  

##Third 

####a)

The last thing you have to do is create a 'Procfile' in your applications 'root' directory. `my_rails_app/Procfile`
Once you have created the file just add the following to it:

{% highlight ruby %}
	web: bundle exec unicorn -p $PORT  -c ./config/unicorn.rb
{% endhighlight %}

###or....

####b)
If you are feeling fancy, (and have not done the step directly above) 
you can create the Procfile including its contents with one command in the commmand line:

run `echo "web: bundle exec rails server thin -p \$PORT -e \$RACK_ENV" > Procfile`
and the 'Procfile' will automatically be created in the 'root' directory of your application and contain
the desired script.

##Fourth and Final
Now all you have to do is `push` the application to [Github][github] and then [Heroku][heroku]:

* Run `git add .`

* Next `git commmit -m "switched web server from default 'Webrick' to 'Unicorn'"`

* Then `git push`

* Lastly `git push heroku`

##YOU'RE DONE!

Boom now you are running Unicorn in your production environment. You and that Heroku employee [Richard][richard] from
the [Ruby Rouges][richard-pod] podcast can rest easy. 

[heroku]:      http://heroku.com
[heroku-docs]: https://devcenter.heroku.com/articles/rails-unicorn

[richard-pod]: 		 http://rubyrogues.com/140-rr-heroku-with-richard-schneeman/
[richard]: https://twitter.com/schneems
[github]:  		http://github.com	
[jekyll-help]: https://github.com/jekyll/jekyll-help


