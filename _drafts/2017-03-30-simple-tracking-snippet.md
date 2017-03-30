---
layout: post
title: "Simple Tracking Snippet"
description: "How to make a tracking snippet in under 20 minutes"
tags: [serverless, javascript]
---

# Tracking a page
Making a website is an awesome feeling. No matter who you're making it for or what purpose it has, it's always satisfying to know that there are people out there seeing your content and navigating your site. It's only after that first buzz that you start to question yourself and wonder who actually is visiting your site. Maybe you add a tag manager, and analytics (probably google analytics, but I don't judge). Then you patiently wait, hoping to see blips on the graph that show your traffic and imagine the steadily rising number. But maybe you want more. You want to be able to track specific people, or even track where your guests are coming from.

Welcome to snitcher, my custom tracking snippet. If your reading this now then congrats! You've been logged as a visitor and I have the following pieces of information about you and your visit.

- ip address
- time of visit
- url that you visited
- query values

Now this is some pretty sparse information, and really I could do a lot better using some user id generation, header analysis, and whatnot. But for my purposes this is fine.


My main reason for having this snippet is to know when someone in particular navigates my site. This is done via the query values. I generally leave special `ref=blah` key-values in all the links to my blog. These refs allow me to track where the user came from. For instance my resume has a link to my blog, and it includes the key value `ref=resume`. Now when I send in my resume I can roughly check to see if they even looked at it, or if they just ignored it and moved on. It's not perfect, but really... I'm not picky.

At it's core this project consists of 2 chunks, the frontend snippet that handles the data gathering, and the backend that logs this information. Let's dive into the frontend and see what we can scrape up!

# Snitcher Snippet
