<style type="text/css">
  .brewer-note {
    background: rgba(0, 0, 0, 0.05);
    border-radius: 3px;
    margin: 0 -10px 20px;
    padding: 10px;
    text-align: center;
  }

  .brewer-note p:last-of-type {
    margin-bottom: 0;
  }
</style>

# Learning Django and AngularJS
In this tutorial you will build a simplified Google+ clone called “Not Google Plus” with Django and AngularJS.

Before we hit the proverbial books and learn to build a rich, modern web application with Django and Angular, let's take a moment to explore the motivations behind this tutorial and how you can get the most out of it.

## What is the goal of this tutorial?
Here at Thinkster, we strive to create high value, in depth content while maintaining a low barrier to entry. We release this content for free with the hope that you find it both exciting as well as informative.

Each tutorial we release has a specific goal. In this tutorial, that goal is to give you a brief overview of how Django and AngularJS play together and how these technologies can be combined to build amazing web applications. Furthermore, we place a heavy emphasis on building good engineering habits. This includes everything from considering the tradeoffs that come from making architectural decisions, to maintaining high quality code throughout your project. While these things may not sound like fun, they are key to becoming a well-rounded software developer.

## Who is this tutorial for?
Every author must answer this difficult question. Our goal is to make this tutorial useful  for novices as well as experienced developers.

For those of your who are in the early days of your software development careers, we have tried to be thorough and logical in our explanations as possible, while still making the text flow fluidly; we try to avoid making intuitive leaps where doing so makes sense.

For those of you who have been around the block a few times, and perhaps are just interested in learning more about Django or AngularJS, we know you don't need the basics explained to you. One of our goals when writing this tutorial was to make it easy to skim. This allows you to use your existing knowledge to speed up the reading process and identify where unfamiliar concepts are presented so you can grok them quickly and move on.

We want to make this tutorial accessible to anyone with enough interest to take the time necessary to learn and understand the concepts presented.

## A brief interlude about formatting
Throughout this tutorial, we strive to maintain consistent formatting. This section details what that formatting looks like and what it means.

* When presenting a new code snippet, we will present the snippet in it's entirety and then walk through it line-by-line as necessary to cover new concepts.
* Variable names and file names appear in-line with special formatting: `thinkster_django_angular/settings.py`.
* Longer code snippets will appear on their own lines:

        def is_this_google_plus():
            return False

* Terminal commands also appear on their own line, prefixed by a `$`:

        $ python manage.py runserver

* Unless otherwise specified, you should assume that all terminal commands are run from the root directory of your project.
 
## A word on code style
Where possible, we opt to follow style guides created by the Django and Angular communities.

For Django, we follow [PEP8](http://legacy.python.org/dev/peps/pep-0008/) strictly and try our best to adhere to [Django Coding style](https://docs.djangoproject.com/en/1.7/internals/contributing/writing-code/coding-style/).

For AngularJS, we have adopted John Papa's [AngularJS Style Guide](https://github.com/johnpapa/angularjs-styleguide). We also adhere to [Google's JavaScript Style Guide](https://google-styleguide.googlecode.com/svn/trunk/javascriptguide.xml) where it makes sense to do so.

## A humble request for feedback
At the risk of sounding cliche, we would not have a reason to make this tutorial if not for you. Because we believe that your success is our success, we invite you to contact us with any thoughts you have about the tutorial. You can reach us via the Olark box in the bottom-right corner of the screen, via Twitter at [@jamesbrwr](http://twitter.com/jamesbrwr) or [@GoThinkster](http://twitter.com/gothinkster), or by emailing [support@thinkster.io](mailto:support@thinkster.io).

We welcome criticism openly and accept praise if you believe it is warranted. We're interested in knowing what you like, what you don't like, what you want to know more about, and anything else you feel is relevant.

If you are too busy to reach out to us, that's OK. We know that learning takes a lot of work. If, on the other hand, you want to help us build something amazing, we await your mail.

## A final word before we begin
It is our experience that the developers who gain the most from our tutorials are the ones who take an active approach to their learning.

We **strongly** recommend you type out the code for yourself. When you copy and paste code, you don’t interact with it and that interaction is in turn what makes you a better developer.

In addition to typing the code yourself, do not be afraid to get your hands dirty; jump in and play around, break things and build missing features. If you encounter a bug, explore and figure out what is causing it. These are the obstacles we as engineers must tackle multiple times a day, and have thus learned to embrace these explorations as the best source of learning.

Let's build some software.

# Setting up your environment
The application we will be building requires a non-trivial amount of boilerplate. Instead of spending time setting up your environment, which is not the purpose of this tutorial, we have created a boilerplate project to get you started.

You can find the boilerplate project on Github at [brwr/thinkster-django-angular-boilerplate](https://github.com/brwr/thinkster-django-angular-boilerplate). The repository includes a list of commands you need to run to get everything running.

<div>
  <strong>Note</strong>
  <div class="brewer-note">
    <p>If you are interested in a detailed appendix on setting up your environemnt, reach out to [@jamesbrewer](http://twitter.com/jamesbrwr) on Twitter.</p>
  </div>
</div>

Go ahead and follow the setup instructions now.

{x: set_up_envrionment}
Follow the instructions to set up your environment

## Checkpoint
If all went well running the server with `python manage.py runserver` should allow you to visit `http://localhost:8000/` in your browser. The page will be blank except for the navigation bar at the top. The links in the navigation bar currently do nothing.

{x: checkpoint_environment}
Make sure your environment is running by visiting `http://localhost:8000/`
