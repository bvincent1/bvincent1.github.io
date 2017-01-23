---
layout: post
title: eClass for lazy people
---
## Intro
Welcome to my first post and foray into the world of online blogging. Witness my attempt to both document my projects and teach others about the wonderful world of hack and slash scripting that I have come to love.

## Explanation
Because I'm lazy and I don't like having to keep checking eClass (the online class resource for U of A) I decided that I should invest the time required to build a scrapper for the eclass course pages. I decided that I wanted to use wget and python to achieve this. Python happens to be my go to for small scripting projects as it is my mother tongue and the easiest for me to remember when starting anew. My choice for wget over curl, and even python based web clients (EG: requests, or even the dreaded urllibs) was due to both my desire to learn more about command line web interaction, as well as the discovery of the clobber flag for wget. Having previously tried to download an entire classes worth of pdfs in one go, I quickly discovered that eclass starts to throttle and even halt users consuming large amounts of bandwidth. This revelation inspired me to be considerate of only downloading new files, and keeping unchanged files untouched on my local machine.

## Meat & Potatoes
Moving forwards we find that there are 4 problems to face. In order they are:


1. Masking the client
2. Authenticating the user
3. Extracting the pdf URLs
4. Putting it all together

### 1. Masking the client
As it turns out masking a get request takes place by defining the user agent of the requesting service. In our case we need to pass in our agent to the wget command with the *\-\-user-agent* flag.

A quick google for "masking user-agents" turns up plenty of results and I eventually decide to use

{% highlight bash %}Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2272.118 Safari/537.36{% endhighlight %}

This will make sure our requests look like a normal user browsing the site, and therefore will enable us to proceed with less hassle.

### 2. Authenticating the user
This part is quite easy to accomplish since wget supports post requests. All we need to do is supply our password and user name, and make the correct request. Then all we have to do is save our cookie for our next few requests. The first of these functionalities will be accomplished through the *\-\-post-data* flag. This will tell wget to post the provided key-value data to the site. We will also use the *\-\-save-cookies* flag to keep our session cookie for when we retrieve our pdfs. We will save our cookie to a file and load it later.

Up to this point our command looks like this:
{% highlight bash %} wget --save-cookies cookie.txt --user-agent "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2272.118 Safari/537.36" --post-data "user=foo&password=bar" {% endhighlight %}
*I leave it up to you to figure out where to fill in your actual credentials*

### 3. Extracting the pdf URLs
Now that we've acquired all the info we need in order to access the pdfs, we can go about extracting the links from the class info page. To do this we will make a request for the main class info page and use python's re library to extract them.

Using the ever usefull dev tools in google chrome we can see that the files are linked inside href tags for the file icon images. These hrefs link to an id that is associated with the file. This means that we need to get all the ids and then call the links instead of just calling a set link with the filename as the URL key.

{% include image.html path="eclass_source.png" path-detail="eclass_source.png" alt="Source for Eclass" %}

This actually makes our job easier since we can now simply look for a string of numbers. We then need to create a regular expression to target these ids. To do this I recommend using my favorite tool [RegExr](http://www.regexr.com/). This is well a put together tool for building and testing regular expressions. I highly recommend it, and I encourage anyone interested to browse the favorites for any usefull insights and re's that they might need down the road.

Using RegEx we come up with */resource\/view.php\?id=([0-9]{7,})/* as our regular expression.

Now that we have the expression we need to start writing the code. As previously mentioned I'll be using python. The code will run on either version, but mine will default to 2.7.

First we make our shebang and our imports
{% highlight python %}
#!/usr/env python

import re
import getpass
import subprocess as sub
from sys import argv
{% endhighlight %}

We will use the subprocess module to call wget from the python. We will also take the argument for the target class from argv.
Next up we create our two functions to implement our wget calls using the python subprocess module. We also defined our target sites that we want to get the cookie from.

{% highlight python %}
def getCookie():
	# def local vars
	auth_site = "https://weblogin.srv.ualberta.ca/"
	auth_cookie = "cookie.txt"


	# get credentials from user and build cmd
	auth_usr = 'foo'  # could also use str(raw_input("User Name:"))
	auth_pass = 'bar' # could also use getpass.getpass("Password:")
	cmd = "wget --save-cookies " + auth_cookie + " --post-data 'user=" + auth_usr + "&pass=" + auth_pass + "' " + auth_site

	# call wget with contructed cmd
	#print(cmd)
	sub.call(cmd)

	return auth_cookie

def getPage(site, cookie):
	# def local vars
	page_output = "site.html"

	# build cmd and get eclass page
	cmd = "wget --load-cookies " + cookie + " --output-document " + page_output + " " + site
	#print(cmd)
	sub.call(cmd)

	return page_output
{% endhighlight %}

### 4. Putting it all together

And Finally we have the function in charge of taking our downloaded site and extracting the URLs and then using the ids to contruct our wget command. Notice that in this code I have used the shortened *-nc* command simply out of preference.

{% highlight python %}
def getFiles(page, cookie):
	# def local vars
	target_exp = r"resource\/view.php\?id=([0-9]{7,})"
	target_folder_exp = r"folder\/view.php\?id=([0-9]{7,})"
	target_main = "https://eclass.srv.ualberta.ca/mod/resource/view.php?id="

	# open site page
	txt = open(page, "r").read()

	# get target file ids
	target_id_list = re.findall(target_exp, txt)

	for i in target_id_list:
		cmd = "wget -nc --load-cookies " + cookie + " " + target_main + i
		#print(cmd)
		sub.call(cmd)
{% endhighlight %}

I would also like to keep our code clean by putting all the actual function calls behind an *if \_\_name\_\_ == "\_\_main\_\_"*. If you are unfamiliar perhaps a [google](http://ibiblio.org/g2swap/byteofpython/read/module-name.html) is in order.

{% highlight python %}
# ### main ### #
if __name__ == "__main__":
	if argv[1].count("http") == 1:
		# get cookie and access page
		c = getCookie()
		p = getPage(argv[1], c)
		# get files
		getFiles(p, c)
	else:
		print("Error, missing arguments")
		print("SITE OUTPUT_DIR")
{% endhighlight %}

## Possible Improvements
A list of things that could certainly be implemented or even improved on:

+ Scrapes all enrolled course pages at once
+ Optional predefined file layout, to allow for auto organization of downloaded files
+ Setting up the script as a cron job
+ Adding (email) notifications of new files
+ Any other reasonable idea

If you do come up with any improvements send me an email with them as I'd love to see. I might even use it myself.
