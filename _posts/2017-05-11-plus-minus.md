---
layout: post
title: "Plus Minus"
description: "Have a little fun with so called metrics"
tags: [python, selenium]
---

# PlusPlus bots
So a recent move at my current employer introduced the concept of a rating system. It's called PlusPlus. It consists of 2 possible commands `@user --` or `@user ++`. These stats are then tallied in a leaderboard for all the company to see and review. Now though I have nothing against the person who wanted it added, I'm not a huge fan of being tracked and "rated" based on slack messages... So in a classic troll move I thought:

> If everyone is stuck at 0 then there's no point for keeping track of points.

So with this concept in mind I had a few different ideas to test.

# What's a slack bot?
So off the top of my head I have a couple attack vectors.

- interact directly with the PlusPlus bot
- use another bot to message it directly
- replicate a client and send it messages as myself
- connect to it with a browser and simply interact with it the old fashion way

So with a clear goal of rigging the elections, we can proceed to evaluate what exactly we're dealing with. At it's core slack operates exactly like an IRC network. A central server receives messages and then propagates them to the clients when they connect for updates. Simple. So what does a slack bot look like then? Well if I were making this, the bot would simply just be another client that checks for updates periodically. I'm just gonna assume that this is the case, since this'd be easiest to do from a developer standpoint. However this means that option #1 is no longer viable since there's no way for me to interact with the bot by pretending to be slack.

~~interact directly with the PlusPlus bot~~

# Crossing out my options
Well lets try using my own bot. If the team behind PlusPlus did a poor job they simply parse all the messages and then do matching against that. Maybe they're silly enough to parse other bots??

Well lets try it!
```python
#!/usr/bin/env python3
"""
test posting to slack
"""

import requests

URL_TEST = "https://hooks.slack.com/services/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t"
TEST = "@braam++"


print(requests.post(URL_TEST, json={"text": TEST}).text)
```

So using this snippet against my pre-existing slack web hook, I can quickly see that the lovely folks behind PlusPlus did in fact do a bit of a better job than I could've hopped for. Maybe there's a way to get it to accept my messages directly, but I'm not interested in trying to figure it out.

~~use another bot to message it directly~~

Next up we have to figure out if I can simply emulate a client and message the bot as myself from a script. A quick google + github search suggests that my options are limited. I could try implementing a RTM interface bot, but that seems like it'll be a lot of fiddling with unnecessary features.

Honestly I just want the simplest thing. I'm not interested in configuring some bot using oath and then reading event hooks via restful interface. Is there nothing simple?? Well it turns out no...

~~replicate a client and send it messages as myself~~

# When all you have is a hammer, everything's a nail...
So I'm down to my last option. Good news is that I know it'll work no matter what slack does. Bad news is that it's somewhat overkill considering what I'm trying to do here is just spam some bot. Also it's pretty fucking slow.

Welp... better get started

```python
!/usr/bin/env python3

"""
Slack PlusPlus score manipulator bot
https://plusplus.chat/
"""

import os
import time
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from bs4 import BeautifulSoup
```

Here's all the libs that I'm going to use. Pretty standard stuff. Next up lets open a browser and login.

```python
def init_slack_env(browser):
    """
    login browser to proper web page with ENV variables
    """
    signin_button_id = "signin_btn"
    email_element_id = "email"
    password_element_id = "password"

    # open slack
    browser.get("https://granify.slack.com/messages/C2DPUG6JK")
    time.sleep(SLACK_LOAD_DELAY)
    if "Sign in to Granify" not in browser.page_source:
        return None

    # authenticate slack
    email_form = browser.find_element_by_id(email_element_id)
    password_form = browser.find_element_by_id(password_element_id)
    submit_form = browser.find_element_by_id(signin_button_id)
    email_form.send_keys(os.environ["SLACK_NAME"])
    password_form.send_keys(os.environ["SLACK_PASSWORD"])
    submit_form.click()

    time.sleep(SLACK_LOAD_DELAY)
    return None
```

Now that I'm logged in, lets get the leaderboards so we have some actions to do.

Now one of the nice (lucky) things about doing it this way, is that I can simply parse the page as it's shown to me. When I look at the first request and the html that I received I find out that it only sends a stub of the page, and then dynamically loads the content using JS. This means that if I had used a different method to interact with slack, I would've also had to figure out how to interact with this API.

So lets look at the html that I'm giving when the page is done loading and rendering.

```html
<div class="columns" data-reactid=".0.1.1.$=1$static-t-granify/leaderboard.0.1.1">
  ...
  <span data-reactid=".0.1.1.$=1$static-t-granify/leaderboard.0.1.1.0.1">
    <ul class="leaderboard-list" data-reactid=".0.1.1.$=1$static-t-granify/leaderboard.0.1.1.0.1.0">
      <li class="leaderboard-header" data-reactid=".0.1.1.$=1$static-t-granify/leaderboard.0.1.1.0.1.0.0">
        ...
      </li>
      <li data-reactid=".0.1.1.$=1$static-t-granify/leaderboard.0.1.1.0.1.0.1">
        <div class="leaderboard-rank" data-reactid=".0.1.1.$=1$static-t-granify/leaderboard.0.1.1.0.1.0.1.0">01.</div>
        <div class="leaderboard-trophy" data-reactid=".0.1.1.$=1$static-t-granify/leaderboard.0.1.1.0.1.0.1.1">...</div>
        <div class="leaderboard-score" data-reactid=".0.1.1.$=1$static-t-granify/leaderboard.0.1.1.0.1.0.1.2">50</div>
        <div class="leaderboard-name" title="@braam" data-reactid=".0.1.1.$=1$static-t-granify/leaderboard.0.1.1.0.1.0.1.3">
          <div class="text-ellipse" data-reactid=".0.1.1.$=1$static-t-granify/leaderboard.0.1.1.0.1.0.1.3.0">
            ...
            <span data-reactid=".0.1.1.$=1$static-t-granify/leaderboard.0.1.1.0.1.0.1.3.0.1:0:0">braam</span>
          </div>
        </div>
      </li>
      ...
```

So it looks like the results are tallied in list elements and then divided up by divs. Great! All we need to look out for is the first list entry since it's a header.

```python
def get_slack_leaderboards(browser):
    """
    returns array of dicts, eg: [ {name: "ben", score: "-999"}, ...]
    """
    values = []
    browser.get("https://plusplus.chat/t/granify/leaderboard")
    soup = BeautifulSoup(browser.page_source, HTML_PARSER)
    leaderboard_rows = soup("li")[1:] # skip first one since its a <hr>
    for row in leaderboard_rows:
        score = row("div")[2].text
        name = row("div")[3].text
        values.append({"name": name, "score": score})

    browser.back()
    time.sleep(SLACK_LOAD_DELAY)
    return values
```

With that out of the way we now get to focus on the main routine of our project.

# Function and functional

So now that we've got a our information and our state figured out, lets get this functional. Now I've been trying to progress into functional programming so I'm going to start at the top, and work my way down.

```python
def main():
    """
    main event loop
    iterate through leaderboards and continously adjust scores to ~ -999
    """

    try:
      while True:
        browser = webdriver.Firefox()
        init_slack_env(browser)

        res = get_slack_leaderboards(browser)
        res.map(lambda entry: adjust_score(browser, entry["name"], -(int(entry["score"])) - 999))

        time.sleep(200)
    except Exception as e:
      raise e
    return None
```

Now the weird thing about functional programming is that for-loops are bad. This is fine to me I guess, but in this case I'd almost prefer using a for-loop just because I'm currently abusing the map function and throwing away the result. Eh, whatever. Under the current setup, I've planned for getting the env set up and then having a method that I can call with the user that I want to adjust, and the score that I want them to be at! This seems pretty functional, though it's not like I'd notice if it wasn't.

```python
def adjust_score(browser, user, adjustment, rate_limit=0.5):
    """
    adjusts user's score by amount with rate limiting
    """
    input_lement_id = "msg_input"

    if adjustment == 0:
        return None
    elif adjustment > -1:
        payload = "%s ++" % user
    else:
        payload = "%s --" % user

    msg = browser.find_element_by_id(input_lement_id)
    repeat(abs(adjustment), send_message, msg, payload, rate_limit)
    return None
```

So working down the chain, here's what the `adjust_score` method looks like. We find the messaging element on slack, and send a message on it `x` number of times. Pretty straight forwards, except we get to try and be functional, so we avoid a for loop with the `repeat` method. This is basically the same thing as the `map` except we use an interval to decide how many times the method should be repeated.

Ignoring the repeat method for a moment the send message function looks something (exactly) like this.

```python
def send_message(message_element, payload, rate_limit=0.5):
    """
    send slack message
    """
    message_element.send_keys(payload)
    message_element.send_keys(Keys.RETURN)
    time.sleep(rate_limit)
    return None
```

As it turns out the web client for slack can have some lag when sending large numbers of messages. This means we have to rate limit them so allow for the client to catch up to us. Pretty easy, but it's important since otherwise we overload the one entry with the same message until we're "done".

```python
def repeat(iterations, function, *args):
    """
    repeat method with arguments for # of iterations
    """
    for _ in range(iterations):
        function(*args)
```

This is probably the only "functional" part of my whole program, and as such is the part that I'm most proud of! It's actually the only thing that I was had to think through rather than just code away. It's a pretty simple method, and it's really not hard to make or use, but I'd like to think that it'll help me progress along the functional methodology.

# Conclusion

Most of this mini-project is pretty straight forwards and not really challenging from a technical standpoint. Trying to make this functional is the real challenge since I'm constantly evaluating my code and trying to work through all the concepts that I can remember in head.

>- Is this function pure?
- Did I mutate this input?
- Why am I using this for-loop?
- Are my methods too long?
- Is there too much state involved?
- Is this even better?

I'm sure that this isn't a great example of anything actually functional, but I'm still happy with how easy it was to do.

Moving forwards, it'd be nice if I could somehow test this, or maybe move to a client spoofing version that way I could do some cool stuff like async tasks and threading (true sign of functional code). If anyone knows of a lib that I could use, please msg me and I'll look into it.

Here's a link to the full source (some minor differences aside)

### [Source](https://gist.github.com/bvincent1/85b7dd82ccfb4e70cc20cecacd15c6bd)
