---
layout: post
title: "Introducing Tracco: the Trello Effort Tracker gem"
date: 2013-03-18 23:27
comments: true
sharing: true
published: true
categories: [Ruby, Trello, Gem]
---
## Trello meets tracking
[Trello](https://trello.com) is a very good surrogate for a physical team board: it's simple and effective, and it can really help when you have a distributed team.
That said, Trello (still) doesn't offer a way to track time estimated and actually spent on cards, though many people are [asking for that feature](https://trello.com/card/time-tracking/4d5ea62fd76aa1136000000c/1054) on Trello's development board.

I had such precise need while working with one of our teams, so I came up with the following idea: using Trello's mention system to send tracking notifications to a predefined board member (I call him the 'tracking user'), collecting along this way all tracking events such as estimates and efforts.

![A tracking example](https://raw.github.com/xpepper/tracco/master/images/tracking_example.png)

Then I built a simple tool to persist and aggregate this data, so that it would have been possible to show interesting metrics such as estimate errors. I called this tool [Tracco](https://github.com/xpepper/tracco): a gem to help track estimates and efforts from Trello.

[Tracco](https://github.com/xpepper/tracco) is a gem, and you can use it as is, but it's intended use is inside an app which displays collected data.
To give an idea of what I mean, I developed a bare minimum Rails app to properly present card estimates and efforts: it's called [Trello Effort App](https://github.com/xpepper/trello_effort_app). It's really simple, but I wish it could be improved with the help of other committers :O)

## More details
To start using [Tracco](https://github.com/xpepper/tracco) you should have a Trello account, a Trello board and a board member to use as 'tracking user'.
You'll also need to know your Trello developer key and generate a proper auth token to have access to the trackinguser's notifications.
To see how to have these two keys, read [the following section](#api_key).

The Trello API is used behind the scenes to read data from the team board. [Tracco](https://github.com/xpepper/tracco) uses the awesome [Trello API Ruby wrapper](https://github.com/jeremytregunna/ruby-trello) for this purpose.

### An example
Here I show an example of how you could use [Tracco](https://github.com/xpepper/tracco). For more info please refer to the [official README](https://github.com/xpepper/tracco/blob/master/README.md).

{% codeblock %}
git clone git://github.com/xpepper/tracco.git
cd tracco
bundle install
{% endcodeblock %}

Then run the initializer
{% codeblock %}
tracco --initialize
{% endcodeblock %}
to create the two configuration files, which you'll need to edit properly (see _"Where do I get an API key and API secret?"_ section).

To fill the correct values for the mongodb environments ([see here](http://mongoid.org/en/mongoid/docs/installation.html#configuration) to have more details).

### <a id="api_key"></a>Where do I get an API key?
Log in to Trello with your account and visit [https://trello.com/1/appKey/generate](https://trello.com/1/appKey/generate) to get your developer\_public\_key.

### Where do I get an API Access Token Key?
To generate a proper access token key, log in to Trello with the 'tracking user' account. Then go to this URL:

    https://trello.com/1/connect?key=<YOUR_DEVELOPER_PUBLIC_KEY>&name=Tracco&response_type=token&scope=read&expiration=never

At the end of this process, you'll receive a valid access\_token\_key, which is needed by Tracco to have the proper rights to fetch all the tracking notifications sent as comments to the 'tracking user'.

## Collecting data from Trello
{% codeblock %}
tracco collect today --environment test # will extract today's tracked data and store on the test db

tracco collect today  # will extract today's tracked data and store on the default (that is development) db

tracco collect 2012-11-1 --environment production  # will extract tracked data starting from November the 1st, 2012 and store them into the production db
{% endcodeblock %}

### Console
You can open a irb console with the ruby-trello gem and this gem loaded, so that you can query the db or the Trello API and play with them

{% codeblock %}
tracco console
{% endcodeblock %}

The default env is development. To load a console in the (e.g.) production db env, execute:

{% codeblock %}
tracco console -e production
{% endcodeblock %}

## Estimate format convention
To set an estimate on a card, a Trello user should send a notification from that card to the tracker username, e.g.

    @trackinguser [15p]
    @trackinguser [1.5d]
    @trackinguser [12h]

estimates can be given in hours (h), days (d/g) or pomodori (p).

    @trackinguser 22.11.2012 [4h]

will add the estimate (4 hours) in date 22.11.2012.

## Effort format convention
To set an effort in the current day on a card, a Trello user should send a notification from that card to the tracker username, e.g.

    @trackinguser +6p
    @trackinguser +4h
    @trackinguser +0.5g

efforts can be given in hours (h), days (d/g) or pomodori (p).

### Tracking an effort in a specific date
To set an effort in a date different from the notification date, just add a date in the message

    @trackinguser 23.10.2012 +6p

There's even a shortcut for efforts spent yesterday:

    @trackinguser yesterday +6p
    @trackinguser +6p yesterday

### Tracking an effort on more members
By default, the effort is tracked on the member which sends the tracking notification.

To set an effort for more than a Trello user (e.g. pair programming), just add the other user in the message, e.g.

    @trackinguser +3p @alessandrodescovi

To set an effort just for other Trello users (excluding the current user), just include the users in round brackets, e.g.

    @trackinguser +3p (@alessandrodescovi @michelevincenzi)

### Tracking a card as finished (aka DONE)
Sending a tracking notification with the word DONE

    @trackinguser DONE

will mark the card as closed.

Moreover, a card moved into a DONE column (the name of the Trello list contains the word "Done") is automatically marked as done.

## Google Docs exporter
To export all your tracked cards on a google docs named 'my_sheet' in the 'tracking' worksheet, run

{% codeblock %}
tracco export_google_docs my_sheet tracking -e production
{% endcodeblock %}

The default env is development.

If you provide no name for the spreadsheet, a default name will be used.
If the spreadsheet name you provide does not exists, it will be created in you google drive account.

So, running simply

{% codeblock lang:ruby %}
tracco export_google_docs
{% endcodeblock %}
will create (or update) a spreadsheet named "trello effort tracking" using the development db env.

## Requirements
* MRI version 1.9.3+
* [mongoDB](http://www.mongodb.org/) - macosx users with homebrew will just run 'brew install mongodb' to have mongoDB installed on their machine.
* (optional) [rvm](https://rvm.io/rvm/install/) is useful (but optional) for development

## Roadmap and improvements
I develop [Tracco](https://github.com/xpepper/tracco) using [Trello itself](https://trello.com/board/trello-effort-tracker-roadmap/509c3228dcb1ac3f1c018791).

## Contributing
If you'd like to hack on [Tracco](https://github.com/xpepper/tracco), start by forking the repo on GitHub:

https://github.com/xpepper/tracco
