---
layout: post
title:  "How Jarme Is Built"
date:   2016-08-18 17:34:25
author: foysal_ahamed
categories: tech
tags: [featured, tech, frameworks, jarme]
image: /assets/article_images/2016-08-18-how_jarme_is_built/picture_of_meaningless_code.jpg
---

Made with :heart: (lol).... But seriously though, we truly loved working on this app and looking at the user stats for first 2 months, we probably have like 3 users including us 2 and even after that, we are still as motivated as the first day to make this app the best.

I use this app on a **daily basis** and every single feature has been very, very useful.

The initial design process took [Iraa](https://twitter.com/iraa_ahmed) approximately *a month*. Once we had enough skeleton to start with, I dived in and coded up a very rough beta within *ten days*.

## tldr; For the lazy ones like me

Here's a quick list of all the tools that runs behind the screen to make jarme run as smoothly as it does -

Client:

- angular 2
- ionic 2
- cordova
- sass

Server:

- VPS on [digitalocean](https://digitalocean.com) for production
- [heroku](https://heroku.com) free tier for staging
- puma with nginx for serving the api
- [lets encrypt](https://letsencrypt.org) for ssl
- ruby on rails
- devise for authentication
- paperclip for image upload/manipulation
- postgresql
- [mailgun](https://mailgun.com) for sending emails

## Languages & Frameworks

Initially I was thinking of using [meteor.js](https://meteor.com) just for the ease of running one codebase everywhere but soon abandoned the idea since we really didn't have any major realtime feature in mind. My next choice was [ionic](http://ionicframework.com). I was comfortable with ionic 1 and ionic 2 was in very early beta, I was itching to build something with it and this was the perfect opportunity. So I started with ionic2 beta.3 and picked the **es6** version instead of typescript. I was somewhat familiar with es6 but completely new to angular2. I ended up learning ***a lot*** of both es6 and typescript while building this app.

On the server side, I wanted node.js but having learned to code in ruby very recently, I wanted to make this a bit more exciting and use rails instead.

The features we had in mind really didn't need a noSql database and actually fits very well in an sql structure so I decided to go with postgresql.

## Client

Ionic 2 was really easy to dive in even though the syntax and everything is very different from v1. I really have to give it to the ionic team for their astonishing work on the documentation. There wasn't a thing that I couldn't find in the docs. They also have a [great slack team](https://ionic-worldwide.slack.com) if you want more hands on help from the community members.

Initially, I picked es6 but soon, the ionic team abandoned support for es6 so I had to move the whole app over to typescript, which was a bit annoyingly repetitive task but a lot easier than I thought it would be. Typescript is not that bad to work with either. I kind of miss the flexibility of typeless plain ol' .js but having a type system also made it easier to debug and maintain.

Also, there is a really nice angular1 token auth package built for devise token auth. However, it's not compatible with angular2 at the time of writing this post. So I had to built an auth module from scratch as angular2 service. It was really fun and learned a lot about authentication and security in general.

## Server

I always liked having more control over my server environment. Which is why I decided to go with a VPS provider instead of PaaS like heroku or openshift. I do, however, use those platforms for staging purposes and they are pretty handy to be honest.

Being a long time user of digitalocean, I really didn't have to look elsewhere. I use `git push` for deploy with custom git hooks. The api is served over nginx and puma. Digitalocean has a great article on how to set it all up [right here](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-rails-app-with-git-hooks-on-ubuntu-14-04).

### Thanks for reading

I really believe in ***using the right tool for the job*** and I'm never too religious about any programming language or framework. Which is why, this has been a really enjoyable ride for me so far. Going out of my comfort zone and using languages, frameworks, tools that I am not very familiar with is a lot more rewarding when you're building something that you will use everyday.

### Related Links
[Stack Share](http://stackshare.io/foysalit/jarme)