# First impressions with Flask and Elastic Beanstalk (EB)

All in all it was pretty simple to get up my flask app up and running on Elastic Beanstalk.  A ran into a couple of gotchas, and wanted to share them to save others some time:

I relied heavily on two tutorials to get going:

1. [Deploying a Flask Application to AWS Elastic Beanstalk](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_Python_flask.html)
2. [Customizing your python container](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_Python_custom_container.html)

Two main gotchas where (see below for how to get solve / work around):

1. [As of June 2013, **EB only supports python 2.6** (2.6.8 to be specific)](#1)
2. [Make sure your container is configured to point to your static files](#2)


Somethings that I am impressed with (again only been playing with this a day):

1. Fundamentally, pretty easy to get up and going
1. Autoscaling seems to be pretty slick
1. Besides the python for the web server, the instances are highly configurable ([docs](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/customize-containers-ec2.html))
	1. in particular I like the configurable services that you can attach to each instance



Things that I am currently struggling with (but again its only been a day) are:

1. Can not specify python version
2. Not much of a community (or at least I can't find the community) around EB.  And so …
	1. Forums and StackOverflow are not manned by Amazon employees, and in general does not seem to have much of an active community
	2. Documentation is pretty poor
	3. No documented patterns or best practices
4. Tailing logs is unintuitive
6. No console concept (eb run python --> and get the interactive console)


----

## Currently Beanstalk only supports python 2.6
<a id="1"></a>
* For now the only real option is to make sure your web app runs on 2.6.  I do not think thats critical, and I am sure in time they will upgrade to 2.7 and gasp maybe 3.0.  But if you need to run a higher version -- you will need to look to another solution (ec2 or heroku).

* To get your localenv to run 2.6, make a virtualenv like so ...

        virtualenv -p /usr/bin/python2.6 your_project_name

* And you will need to keep another session open with 2.7 or greater to work with the `eb` cli (this still makes be laugh)


## Configure your container to point to you static files:
<a id="2"></a>
If you do not do this, all of your css, javascript and all your other static goodness will bot be served.

1. Create a `.ebextensions` folder in your project root
2. Create a [project_name].config file in that .ebextensions folder
3. Mine looks like the following:

	    option_settings:
	      "aws:elasticbeanstalk:container:python:staticfiles":
	        "/static/": "static/"    
 
You can read more about configuration [here](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/customize-containers-ec2.html)   

## Other tips
### Using brew to install the eb cli
* If you are on a mac, you can use brew to install the elastic beanstalk cli (ironically you need to have python 2.7 or 3.0 installed on your machine)

        brew install aws-elasticbeanstalk
    
### To Check out the logs
You have a couple of options to check out the logs:

1. using the CLI, `eb logs`
2. going to the web console, navigate to logs, and hit 'Snapshot logs' ([docs](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.loggingS3.title.html))

I found the that formatting of `eb logs` in the console to be pretty unreadable.  The code is found [here](http://aws.amazon.com/code/6752709412171743) (couldn't find it on github).  If I find I rely on this a lot I will probably rewrite this.  I think the main issue is two fold:

1. They probably are giving us too much info.  They provide us separate logs from the following sources:

	1. /opt/python/log/httpd.out
	2. /var/log/httpd/error_log
	3. /var/log/cfn-hup.log
	4. /opt/python/log/supervisord.log
	5. /var/log/eb-tools.log
	6. /var/log/httpd/access_log
	7. /var/log/eb-cfn-init-call.log
	8. /var/log/eb-publish-logs.log
	9. /var/log/cfn-init.log
	10. /var/log/eb-cfn-init.log

2. And I think they are simply tailing each of these files. So you get the last n rows for each file.  What I _really_ want to know is what _just_ happened or even better -- what is happening right now.




    
