# Sharing Code Across Applications


*Problem Statement*  We have libraries that we need to share across multiple applications.  Using Git submodules, makes me cry -- there has to be a better way (and there is)

I am going to break this up into three sections:

* Packaging Python
* Sharing Publicly
* Sharing Privately

## Packaging

Python (our beloved language with bells and whistles includes) has `setuputils` built in that we can use to package our utility.

I really do not having anything to say that hasn't been said before.  This tutorial was excellent: 

> http://www.scotttorborg.com/python-packaging/minimal.html

When in doubt do what Kenneth does:
	
> https://github.com/kennethreitz/requests/blob/master/setup.py

Another nice article (with impressive formatting)

> https://jamiecurle.co.uk/blog/my-first-experience-adding-package-pypi/


But the gist of it is:

1. setup.py (to define whats to be shared)
2. Gotsa to have README.rst (sharing code with documenting it is useless)
3. Test it locally
4. Use [Semantic Versioning](http://semver.org/) (or you are a bad person)


This stuff needs to happen regardless if you share in the public domain or privately

## Sharing in the public domain

Okay, you wrote an amazing utility and want to share it with the world.  I really was blown away how easy it was to do this with thanks to PyPi (Python Package Index) (aka the Cheeseshop)

For this example, I wrote a package called `human_dates`  That takes some the syntax sugar from the ruby and rails world and brings it over to python land.

You can find the code [here](https://github.com/jtushman/human_dates)


Once you have you package tested locally, its as simple as:

	$ python setup.py register
	$ python setup.py sdist upload


And now anyone can do the following:

	$ pip install human_dates
	
And then in a python console, you can:

	```python
	
	from human_dates import time_ago_in_words, beginning_of_day
	
	print time_ago_in_words()
	#prints "just now"
	
	print time_ago_in_words(beginning_of_day())
	# prints 8 hours ago
	```


AWESOME!  I really enjoyed this experience.  I found the overhead very light for creating reusable components.

Also -- if this still sounds like to much work for you -- the least you can do is just write and share a [gist](https://gist.github.com/) of your snippet.


## Sharing in the private domain (within your company)

Okay, now things get a bit more interesting.

Requirements:

* The sharing should be done via pip, and the requirements.txt file
* I do not want to set up my own pypi server
* In needs to be secure within our organization
* But, needs to be deployable to 3rd party PaaS providers like heroku or elastic beanstalk

<br/>

To accomplish this, I am using pip's ability to interact with git over ssh.  That and git tagging.  Here are the steps that I took ...


1. Create a private git repo (I am using github)	
2. Make you package, just like we did before.  For this example I will share piece of code that I use to inspect into dictionaries. ([here is the project](https://github.com/jtushman/dict_digger) (note: its public to share with you guys -- but it the real use case it MUST be private.))

		'''python
		# Core Imports
		from copy import deepcopy
		
		
		def dig(hash,*keys):
		    """
		    digs into an dict, if anything along the way is None, then simply return None
		    """
		    end_of_chain = deepcopy(hash)
		    for key in keys:
		        if isinstance(end_of_chain, dict) and key in end_of_chain:
		            end_of_chain = end_of_chain[key]
		        else:
		            return None
		
		    return end_of_chain
		'''


3. push to github.  It needs to have all the same trappings of a python package you would push to PyPi

3.  When working with git and pip, you need to go through an additional step of explicitly tagging your versions

		git tag -a v0.1.0 -m 'version 0.1.0 initial version'
		git push --tags


4. For YOU to install the most current version from HEAD, we can now do the following:

		pip install git+ssh://git@github.com/jtushman/dict_digger.git
		
4. For YOU to install the a **specific** version, we can now do the following:

		pip install git+ssh://git@github.com/jtushman/dict_digger.git@v0.1.0		
		
At this point, your colleagues and deployment machines WILL NOT be able to access this package

5. For your colleagues to access this library:

	1. First they will need to have access to your github repository.  So make sure you have the permissions set.  Best handled on the github website
	2. They will need to turn on ssh agent forwarding.  Following instructions ([here](https://help.github.com/articles/using-ssh-agent-forwarding))
	
6. For your deployment machines to access the library.  You will need to follow the following two steps:

	1. Create a [Machine User](https://help.github.com/articles/managing-deploy-keys#machine-users)
	2. Add the ssh keys to your deploy machine.  If you are using heroku.  Its as simple as [this](https://devcenter.heroku.com/articles/keys#adding-keys-to-heroku)



Cool -- this is working for me.  Love to hear others thoughts and successful approaches.


-----

Other interesting articles while researching this ...

- [http://lucumr.pocoo.org/2012/6/22/hate-hate-hate-everywhere/](http://lucumr.pocoo.org/2012/6/22/hate-hate-hate-everywhere/)
- [http://blog.mozilla.org/webdev/2013/01/11/switching-to-pip-for-python-deployments/](http://blog.mozilla.org/webdev/2013/01/11/switching-to-pip-for-python-deployments/)









