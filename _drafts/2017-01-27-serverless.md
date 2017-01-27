---
layout: post
title: "Python Serverless"
description: "Finally a place to run all my code."
og_image: "documentation/sample-image.jpg"
tags: [serverless, aws, python]
---

## Lambda, AKA  FaaS
If you're like me, one of the biggest problems you've always had was where to run your code. Sure most think that you can just scramble together a home server and run it all locally. But then how to you remotely do changes? Is it only secured to the local network? How do you know if it breaks? I can't connect, did the server shutdown? Why didn't my cron job work?

These questions are constantly at the back of mind when I work with my crappy out of date server running on decode old hardware plugged into the corner of my living room where it won't keep me or my roommates awake at night.

I try to keep all these problems in mind whenever I find myself bored at work since it'll give me something to research and keep up to date with. Most recently I came across [Serverless]() and AWS [Lambda](). When I first looked into it, it seemed like a strange and very niech concept.

"Who'd wanna run only 1 function at a time? How could you even do anything with that?"

These were the plebian questions that floated in my head when I first heard of this. Little did I know that this was not even remotely what FaaS is even directed towards. As it turns out I was completely missing the point of this whole service.

- your code runs on a server
- you don't have to manage anything
- it's not just a function

#### Revisiting my concerns
~~But then how to you remotely do changes?~~  
~~Is it only secured to the local network?~~  
~~How do you know if it breaks?~~  
~~I can't connect, did the server shutdown?~~  
~~Why didn't my cron job work?~~  

So I'm an idiot.

## Serverless
"Welcome to AWS the most convoluted and feature bloated provider I can imagine"

Now I don't like amazon as a software company because of their crazy stupid software practices (Yeah take that Jeff Bezos, I'm calling you out!!), but that doesn't mean that they don't have a lot of cool stuff. Unfortunately navigating their cool stuff requires a lot of digging around, and even reading blogs just to figure out what all these names even mean.

[ElasticBeanstalk]()? Da fuck is that?
At least [S3]() makes sense.
