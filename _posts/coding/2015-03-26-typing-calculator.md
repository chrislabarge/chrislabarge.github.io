---
layout: post
title:  "Typing Calculator Tutorial"
date:   2015-03-26 12:01:27
permalink: /typing-calculator/
author: "Chris LaBarge"
photo-heading: "/images/WPM6.jpg"
description: "A tutorial on creating a typing calculator words per minute app 
using Ruby and the Volt frame-work."
keywords: "typing calculator, volt, words per minute, opal, ruby "
category: coding
---

##Overview

[Volt](http://voltframework.com/) is a new Ruby framework that allows your Ruby code to run on both the 
server and the client.  It uses the [opal](http://opalrb.org/) gem to compile 
the Ruby code to Javascript.

I suggest watching the [Intro to Volt](https://www.youtube.com/watch?v=Tg-EtRnMz7o)
video, that goes over how to set up a to-do list in under 20 minutes. It is very
comprehensible and showcases the magic of Volt.

In this tutorial you will learn how to create a working
that updates as the user types.([like this one](https://typing-calculator.herokuapp.com/))  The the tutorial will go over how to...

- Install the Volt Framework
- Generate new Volt application 
- Use reactive form input 
- Create backend code for the typing calculator
- Enable Bootstrap Animation



##Installation

First we need to install the Volt framework:

{% highlight bash %}
	gem install volt
{% endhighlight %}

Once installed, create a new Volt project:

{% highlight bash %}
volt new typing-calculator
cd typing-calculator
{% endhighlight %}


Note, Volt uses [mongoDB](https://www.mongodb.org/) for its database. You will 
need to download and install it if you do not have it already on your machine.

If you are using git for version control and plan on pushing your app up to 
github, you must get rid of your secret key. Just open up the app.rb in your 
applications config directory...

{% highlight bash %}
config/app.rb
{% endhighlight %}

And delete the string labeled config.app_secret


##SetUp
Fire up the Volt server which automatically pushes any changes you save to
anyone viewing the page.

{% highlight bash %}
	bundle exec volt server
{% endhighlight %}

Open your browser and put "http://localhost:3000/" in the address bar.  This 
will bring you to the homepage of your new app. The template already includes
a login link for in the navigation.  We won't be needing users to sign up or 
login for this project so go ahead open the project in your favorite editor and 
locate the main.html file.

{% highlight bash%}
	app/main/views/main/main.html
{% endhighlight %}

And comment out the user templates

{% highlight html%}
.
.
.
<:Body>
  <div class="container">
    <div class="header">
      <ul class="nav nav-pills pull-right">
        <:nav href="/" text="Home" />
        <:nav href="/about" text="About" />
    <!--    <:user-templates:menu /> COMMENT ME OUT -->
      </ul>
      <h3 class="text-muted">typing-calculator</h3>
    </div>
.
.
.

{%endhighlight%}

You can also comment out the routes for signing in and logging in as well.

{% highlight bash%}
	app/main/config/routes.rb
{% endhighlight %}

{% highlight Ruby %}
# See https://github.com/voltrb/volt#routes for more info on routes

get '/about', _action: 'about'

# Routes for login and signup, provided by user-templates component gem
#get '/signup', _controller: 'user-templates', _action: 'signup'
#get '/login', _controller: 'user-templates', _action: 'login'

# The main route, this should be last. It will match any params not
# previously matched.
get '/', {}

{% endhighlight %}



##Reactive Form Input

Locate the index.html file within views and add some content along with a form 
to get user input.  


{% highlight bash%}
	app/main/views/main/index.html
{% endhighlight %}


{% highlight html %}
<:Title>
  Typing Calculator

<:Body>
  <h1>Words Per Minute Typing Calculator</h1>
 
  <form e-submit="complete" role="form">
    <div class="form-group">
     <label><h3>Begin Typing</h3></label>
     <input class="form-control submit-field" type="text" 
            value=" {% raw html %}{{page._user_string}}{% endraw %}"> 
    </div>
  </form>
{% endhighlight %}


The input value for the form {% raw %}{{ page._user_string }}{% endraw %} is a piece 
of Ruby code.  Anything within the double curly brackets {% raw %}{{ ruby_function }}{% endraw %}
is executed as ruby code.  In this case we are creating a page collection. In Volt, 
there are several types of collections to store data.  Page collections are 
temporary, meaning they will lose their data once you refresh or visit a 
different page.  Which is fine because we only need one page.


This form/input field is where the users will begin typing. The data we get from
this form is how we will calculate the typing speed of the user.

![Reactive Form Input](/images/WPM1.jpg)

Now we need sample text that the user will type. This can be any chunk of text want.
We will start by defining a function in the app controller that returns the 
text.

{% highlight bash%}
	app/main/controllers/main_controller.rb
{% endhighlight %}

{% highlight ruby%}
class MainController < Volt::ModelController
  def index
    # Add code for when the index view is loaded
  end

  def about
    # Add code for when the about view is loaded
  end

  def sample_text
   "In Volt, to simplify managing application state, all application state is kept in models that can optionally be persisted in different locations. By centralizing the application state, we reduce the amount of complex code needed to update a page. We can then build our page's html declaratively."
  end

{% endhighlight %}

Now add the newly defined method to the home-page

{% highlight bash%}
	app/main/views/main/index.html
{% endhighlight %}

{% highlight html%}
<:Title>
  Typing Calculator

<:Body>
 <h1>Words Per Minute Typing Calculator</h1>
 
 <h3>Sample Text</h3>
 <p>{%raw ruby%}{{sample_text}}{%endraw%}</p>
.
.
.
{% endhighlight %}

![Sample Text](/images/WPM2.jpg)

##Words Per Minute(WPM) Calculations

Now for writing the back end code in order to compare the users input with the
sample text and calculate how much time has passed.

[How to Calculate Typing Speed (WPM) and Accuracy](http://www.speedtypingonline.com/typing-equations)
is what I used to come up with the back-end. Check it out if you would like a 
deeper understanding.  It's really not that complicated.

In order to compare the sample text with the user input we must split the strings
and put them into an array.

Create two functions in the controller that handle this task.

{% highlight bash%}
	app/main/controllers/main_controller.rb
{% endhighlight %}

{% highlight ruby%}
class MainController < Volt::ModelController
 .
 .
 .
 def sample_array 
   sample_text.split
 end
 
 def user_array 
   page._user_string.split
 end
{% endhighlight %}



 

You can do this right on the home page for now in order to help you visualize. 

{% highlight bash%}
	app/main/views/index.html
{% endhighlight %} 

{% highlight html %}

{% raw ruby%} 
<b>sample text array</b>
<p>{{sample_array}}</p>
	
<b>user text array</b>
<p>{{user_array}}</p>

 {% endraw %} 
{% endhighlight %} 


![Sample Text](/images/WPM3.jpg)

Now we have to create a function that compares the two arrays, and gives us 
an array of user mistakes. 


{% highlight bash%}
	app/main/controllers/main_controller.rb
{% endhighlight %} 

{% highlight ruby%}
.
.
.
def mistakes_array
  popped_array = user_array
  popped_array.pop
  mistakes = popped_array - sample_array
end
.
.
.
{% endhighlight %}

The reason we set a new variable to the user array and pop that new variable is 
because you do not want the word the user is in the process of typing to count as 
a mistake.  You only want completed words to be considered.  

Add the 'mistakes_array' function to the home page to get an idea of what I am 
talking about.

{% highlight bash%}
	app/main/views/index.html
{% endhighlight %} 


{% highlight html %}

{% raw ruby%} 

<b>mistakes array</b>
<p>{{mistakes_array}}</p>


 {% endraw %} 
{% endhighlight %} 

![Sample Text](/images/WPM4.jpg)

As you can see in the photo above, the second mistake will not count until the
user starts a new word following the mistake.  

We can use the mistakes_array to find the user's accuracy. But before writing 
the accuracy method, we must create one the finds the total number of characters
in an array.



{% highlight bash%}
	app/main/controllers/main_controller.rb
{% endhighlight %}

{% highlight ruby%}
def character_length(array)
  array.join.length
end

def accuracy
  correct_characters = character_length(user_array) - character_length(mistakes_array)
  fraction = correct_characters/ character_length(user_array)
  accuracy_percentage = (fraction * 100).round
end

{% endhighlight %}

Dividing the total character length of the user_array by only the correct_word_length
gives us a decimal/fraction that we multiply by 100 to get the accuracy percentage.

Per  [SpeedTypingOnline.com](http://www.speedtypingonline.com/typing-equations), 
when calculation a typing speed, a WORD is any "5 characters".  Lets create the
function to find the number of words, in the controller. 

{% highlight bash%}
	app/main/controllers/main_controller.rb
{% endhighlight %}

{% highlight ruby%}
def word_num
 character_length(user_array) / 5
end 

{% endhighlight %}

Feel free to add your new "word_num" and "accuracy" functions to the the home
page if you want to see them in action.


{% highlight bash%}
	app/main/views/index.html
{% endhighlight %} 


{% highlight html %}
{% raw ruby%}

<p>{{word_num}}</p>
<b>number of words</b>

<p>{{accuracy}}</p>
<b>accuracy</b>

{% endraw %}
{% endhighlight %} 


Now it is time to create the methods that will allow us to calculate how much
time has passed.

{% highlight bash%}
	app/main/controllers/main_controller.rb
{% endhighlight %}

{% highlight ruby%}

def time_elapsed
  if page._user_string.length == 1
    @start_time = Time.new
  elsif page._user_string.length == 0
    @start_time = 0	
  else
   nil
  end	
 
  minutes = (Time.now - @start_time).round / 60

end

{% endhighlight %}

The if-statement allows the timer to start only when the user starts typing.  The
function then returns the amount of time elapsed in minutes.  The if-statement
also allows the timer to start over once the user clears out the form.

By dividing the elapsed time in seconds by 60, the function returns the number
of elapsed time in minutes.

Now that we have the time, we can find the gross words per minute. 

{% highlight bash%}
	app/main/controllers/main_controller.rb
{% endhighlight %}

{% highlight ruby %}
def gross_wpm
  (word_num / time_elapsed).round
end
{% endhighlight %}

Using the gross_wpm function we can find the net words per minute, which is
what will give us our final words per minute calculation.

{% highlight ruby %}
def net_wpm
  errors_per_min = (mistakes_array.count / time_elapsed).round
  gross_wpm - errors_per_min
end
{% endhighlight %}


Now that we have the net words per minute typed we can use that function in 
a bootstap progress bar.  This is one of several ways you can use reactive form
input to control real time animation on your page.

{% highlight bash%}
	app/main/views/index.html
{% endhighlight %} 

{% highlight html %}
<div class="progress">
{% raw %}
  <div class="progress-bar progress-bar-{{ if net_wpm > 60 }}success
                          {{elsif net_wpm > 31 }}warning
                          {{else}}danger{{end}} 
              progress-bar-striped" role="progressbar" 
              aria-valuenow="{{net_wpm}}" 
              aria-valuemin="0" 
              aria-valuemax="100" 
              style="width: {{net_wpm}}%">
							
     <span class="sr-only">{{net_wpm}}%</span>
  </div>

</div>
.
.
.
<p>{{net_wpm}}<p>
<b>net wpm</b>

<p>{{gross_wpm}}<p>
<b>gross wpm</b>

{% endraw %}
{% endhighlight %}


![Progress Bar](/images/WPM5.jpg)

##DONE

Now we have a working application, you can style and animate it how ever you like.
You can change whatever you want about the app too.  Have it do something cool like
randomly access sample text from parts of the web. Make sure to let me know of
any cool features you come up with!

Here is an example of my design.  

![Finished Product](/images/WPM6.jpg)

The above site is hosted on Heroku @ [https://typing-calculator.herokuapp.com/](https://typing-calculator.herokuapp.com/)

You can view the code for this application 
on my [Github](https://github.com/chrislabarge/typing-calculator)






