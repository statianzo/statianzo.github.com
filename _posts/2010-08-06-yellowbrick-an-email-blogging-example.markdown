---
layout: post
title: "Yellowbrick: an email blogging example"
---

Yellowbrick is a small sinatra blog designed to run on Heroku. It combines
several of Heroku's addons such as MongoHQ and Sendgrid. Using the skeleton
Git repository, you can be up and rolling with your own blog similar to that of
Tumblr, but configurable down to the last snippet of code.

Why?
---

Simple: learning. Yellowbrick is an easy way to help get a grasp of
programatically dealing with incoming emails. Also, there's some definite cool
factor to running your own custom blog over taking the de facto. Finally, it's
extensible; there's no limit as to what you can create when the source is yours.

Prerequisites
---

First of all, Yellowbrick requires that you own a domain and have control over
its DNS settings. If you don't have a Heroku account, Git, and Ruby go grab
those now too.

Also, you need the *bundler*, *heroku*, and *thin* gems installed.

    $ sudo gem install bundler heroku thin

Getting Some Source
---

Pull down a copy of the skeleton from Github and enter the directory

    $ git clone git://github.com/statianzo/yellowbrick.git
    $ cd yellowbrick

Yellowbrick consist of a relatively simple structure. *yellowbrick.rb* is the
core of the app, handling incoming requests. *boot.rb* loads dependencies and
sets up the database. *config.ru* prepares the app to be run on Heroku.

The *models* and *views* directories contain the models and views for the app,
respectively.

Running Locally
---

To start up Yellowbrick locally, you first need to install the Gemfile's gems

    $ bundle install

Second, you need to have the environment variable *MONGOHQ_URL* pointed to an
instance of MongoDB. For a local instance you could use the following:

    $ export MONGOHQ_URL='mongodb://localhost/yellowbrick'

And finally, start the *thin* webserver

    $ thin start

If you browse to `localhost:3000` you should be met with an empty Yellowbrick
blog. To send a sample post, use *curl*

    $ curl -d "subject=MyPost&text=Body+Goes+Here" http://localhost:3000/receive

Refreshing will show your post

Preparing Heroku
--- 

Running locally and posting via command line is not too exciting. Blogs are all
about web presence. Let's get some.


Create a new Heroku app using the *heroku* command line tool. Set up the
Sendgrid and MongoHQ addons. Push to the heroku remote repository.

    $ heroku create <app_name_here>
    $ heroku addons:add mongohq:free
    $ heroku addons:add sendgrid:free
    $ git push heroku master

Your site should now be usable the same as your local instance. Heroku's MongoHQ
addon automatically sets up the *MONGOHQ_URL*, and will point to the correct
location.

Set Up Sendgrid
---

Yellowbrick is functional now. However, to have the ability to receive and
handle emails, Sendgrid must be configured. Sendgrid provides a feature to parse
an email sent to a doma and POST it to a specified url.

If your domain's DNS settings are controlled by your domain provider, go to
their site, and add an additional *mx record* that points to *mx.sendgrid.net*.

Retrieve your Sendgrid credentials from Heroku

    $ heroku config

Log into the Sendgrid website, enter the Developers - Parse Incoming section
where you can set up a hostname to receive emails at
(*watch.yourdomain.com*) and the url to hit when an email arrives,
*yourapp.heroku.com/receive*. It may take up to a half hour while your mx
record is getting set up.

Send an email with a subject and body to *anything@watch.yourdomain.com*, and in
moments, the contents should be displayed on your heroku site.

Extending Yellowbrick
---

As is, Yellowbrick is raw and not secure against anyone else getting their hands
on the email address to post to your blog. An initial enhancement could be to
add additional restrictions and preventing senders who are not your email using
the `params[:to]`.

Read more about what parameters are sent in Sendgrid's API documentation.









