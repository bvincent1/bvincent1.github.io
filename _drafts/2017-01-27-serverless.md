---
layout: post
title: "Going Serverless"
description: "Yet another lambda appraisal blog. I know there's tons. But its cool and hip. And Easy."
og_image: "documentation/sample-image.jpg"
tags: [serverless, aws]
---

## Lambda, AKA  FaaS
If you're like me, one of the biggest problems you've always had was where to run your code. Sure most think that you can just scramble together a home server and run it all locally. But then how to you remotely do changes? Is it only secured to the local network? How do you know if it breaks? I can't connect, did the server shutdown? Why didn't my cron job work?

These questions are constantly at the back of mind when I work with my crappy out of date server running on decode old hardware plugged into the corner of my living room where it won't keep me or my roommates awake at night.

I try to keep all these problems in mind whenever I find myself bored at work since it'll give me something to research and keep up to date with. Most recently I came across [Serverless]() and AWS [Lambda](). When I first looked into it, it seemed like a strange and very niche concept.

"Who'd wanna run only 1 function at a time? How could you even do anything with that?"

These were the plebeian questions that floated in my head when I first heard of this. Little did I know that this was not even remotely what FaaS is even directed towards. As it turns out I was completely missing the point of this whole service.

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

Without server ado I present to you yet another AWS Lambda quick start tutorial!!
###### Cept I actually have no clue how to do it manually, and honestly I don't really want to learn, so instead I present:

## Serverless
"Welcome to AWS: the most convoluted and feature bloated web services provider I can imagine"

Install serverless:
```bash
npm install -g serverless
```

Set environment variables to your aws keys:
```bash
# add to .bashrc or whatever file you use
export AWS_ACCESS_KEY_ID=GETUROWNACCESSKEY
export AWS_SECRET_ACCESS_KEY=ITSASECRET
```

Create a starter project:
```bash
serverless create --template aws-nodejs
cd aws-nodejs
```

Deploy the project:
```bash
serverless deploy
```

Look at how easy that was! We just deployed code to a server without any manual configuration, or even tikering to get it working. Of course if we wanted to tinker with it we can absolutely look into `serverless.yaml` which houses all our configurations, names, and `env` variables that we need. So lets do that.

```yaml
service: aws-nodejs

provider:
  name: aws
  runtime: nodejs4.3

functions:
  hello:
    handler: handler.hello
```


## Conclusion

Now I don't like amazon as a software company because of their crazy stupid software practices (Yeah take that Jeff Bezos, I'm calling you out!!), but that doesn't mean that they don't have a lot of cool stuff. Unfortunately navigating their cool stuff requires a lot of digging around, and even reading blogs just to figure out what all these names even mean.

[ElasticBeanstalk]()? Da fuck is that?
At least [S3]() makes sense.