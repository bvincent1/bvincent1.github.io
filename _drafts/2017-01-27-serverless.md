---
layout: post
title: "Going Serverless"
description: "Yet another lambda appraisal blog. I know there's tons. But it's cool and hip. And Easy."
og_image: "documentation/sample-image.jpg"
tags: [serverless, aws]
---

## Lambda
"AKA FaaS"

If you're like me, one of the biggest problems you've always had was where to run your code. Sure most think that you can just scramble together a home server and run it all locally. But then how to you remotely do changes? Is it only secured to the local network? How do you know if it breaks? I can't connect, did the server shutdown? Why didn't my cron job work?

These questions are constantly at the back of mind when I work with my crappy out of date server running on decade old hardware plugged into the corner of my living room where it won't keep me or my roommates awake at night.

I try to keep all these problems in mind whenever I find myself bored at work since it'll give me something to research and keep up to date with. Most recently I came across [Serverless](https://serverless.com/) and AWS [Lambda](http://docs.aws.amazon.com/lambda/latest/dg/welcome.html). When I first looked into it, it seemed like a strange and very niche concept.

"Who'd wanna run only 1 function at a time? How could you even do anything with that?"

These were the plebeian questions that floated in my head when I first heard of this. Little did I know that this was not even remotely what FaaS is even directed towards. As it turns out I was completely missing the point of this whole service.

- Your code runs on a server
- You don't have to manage anything, so someone else is worrying about the uptime
- It's just a function
- It can be triggered by a variety of events

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

Set environment variables to your aws admin keys:
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

Look at how easy that was! We just deployed code to a server without any manual configuration, or even tinkering to get it working. Of course if we wanted to tinker with it we can absolutely look into `serverless.yaml` which houses all our configurations, names, and `env` variables that we need. So let's do that.

```
service: aws-nodejs # name of our app

provider:
  name: aws # which FaaS provider we're going to use
  runtime: nodejs4.3

functions:
  hello: # name of our function at the FaaS level
    handler: handler.hello # file_name.function_name
```

Now this is a basic app that doesn't do much. But the point here is to show just how each it is to start and create an app from a basic template. From here we could add libs with `npm install newest-crazy-thing` and it'd automatically get installed at the FaaS level. Now all we have to figure out is the code, not the configuration.

## AWS Lambda framework
"Nano services with a smile!"

Let's take a look at the function call and see what the arguments and responses look like.

```js
'use strict';

// our FaaS function needs to match the handler file_name.function_name template
module.exports.hello = (event, context, callback) => {
  const response = {
    statusCode: 200,
    body: JSON.stringify({
      message: 'Go Serverless v1.0! Your function executed successfully!',
      input: event,
    }),
  };

  // send our response to the http event
  callback(null, response);
};
```

Here's a quick overview of the arguments passed to us:
- `context` is an object for looking up runtime data about our function (time left, function name, [etc](http://docs.aws.amazon.com/lambda/latest/dg/nodejs-prog-model-context.html#nodejs-prog-model-context-methods))
- `event` is the event that triggered our function. In our case this will be the body of the request

*If we weren't using http events, the event would be one of [these](http://docs.aws.amazon.com/lambda/latest/dg/eventsources.html)*

The callback arguments looks like
```js
callback(Error error, Object result);
```

## Conclusion

Now I don't like amazon as a software company because of their crazy stupid software practices (Yeah take that Jeff Bezos, I'm calling you out!!), but that doesn't mean that they don't have a lot of cool stuff. Unfortunately navigating their cool stuff requires a lot of digging around, and even reading blogs just to figure out what all these names even mean.

All in all, I'm super excited by this stuff since it's very much a service that handles my use cases and offer a lot of space for interesting development.

Long term my only concern is with scalability and long-running functions. It will be interesting to see how the eco-scape grows around this service and to see how it will adapt to the restrictions that it presents.
