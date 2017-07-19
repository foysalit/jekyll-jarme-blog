---
layout: post
title:  "Send push notification from rails to an ionic app"
date:   2017-07-19 14:34:25
author: foysal_ahamed
categories: tech
tags: [ionic, rails, push, notifications, rpush, jarme]
image: /assets/article_images/2016-09-14-display_ionic_datetime_picker_programmatically/cover.png
---

Why?
====
Setting up push notification is hard. Especially when it comes to hybrid apps. It seems easy at first glance but the more you try to do something specific the more it impossible it seems. I myself have a hard time everytime I have to implement push notification for an app doesn't matter what it's built on. Which why I have decided to document it. I have just finished implementing push notification on [Jarme App](https://jarmemori.es) and this post will reflect that process and use the same tools and implement a similar feature that we have in jarme app.

***Ready? Let's go.....***

Tools
=====
We will use [rpush gem](https://github.com/rpush/rpush) to send notifications from server and [sidekiq gem](https://github.com/mperham/sidekiq) to periodically run notification job along with rails on the server. On the client, we will have a shiny ionic app that uses [phonegap-push-plugin](https://github.com/phonegap/phonegap-plugin-push). For now, we will only focus on android and use google's FCM platform but I do intend to write up another post for iOS setup asap. I wrote this post on a linux machine but I'll expect you to have a \*nix system since most of the commands in this post will probably require a \*nix environment.

I will walk you through each and every step starting from creating our app till receiving push notifications on device. However, I will assume you're somewhat *familiar* with Angular/Ionic2+ and Rails in general and of course, feel free to ask if anything doesn't make sense or seems confusing or doesn't work the way it should.

Ground Work
===========
For most of you, skip this step since you probably already have a rails api along with an ionic app up and running. However, it may be worth just skimming through the setup we are going to lay out so that you can make sure your apps also have compatible versions.

First of all, I like to keep my apps organized in folders so usually, I have one folder that contains all related tools: *api app, mobile app, web app* etc. So, let's do that first; let's creatively name our app ***pushme*** and give it a nice folder structure :
```
cd ~
mkdir pushme
cd pushme
mkdir api
mkdir mobile
```
Just to be clear, we're going for a structure that looks like :
```
pushme
    |- api
        |- pushme
            |- ..... all our rails code
    |- mobile
        |- pushme
            |- ..... all our ionic code
```    

Let's setup a fresh new rails app. I'm assuming you have ruby environment setup with rails gem installed on your machine so just run the following command and it will scaffold a brand new rails app (creatively named *pushme*) for you :
```
cd ~/pushme/api/
rails new pushme
```
Now simply navigate inside the newly created folder and fire up the rails app :
```
cd pushme
rails s
```
You should now be able to see a jolly little ***Welcome Aboard*** page if you go to this url `http://localhost:3000/` in your browser. So, this will be the home for our backend api.

Now, let's quickly setup our ionic app, again, assuming you already have nodejs, cordova and ionic cli setup on your machine (if not, please follow [this guide](https://ionicframework.com/docs/intro/installation/)) :
```
cd ~/pushme/mobile/
ionic start pushme tabs
```
We now have a newly created ionic project that has a `tabs` layout. To see what that means, navigate inside the project folder and run the app :
```
cd pushme
ionic serve
```
You should now be able to see a ***Welcome to ionic*** page on `http://localhost:8100/` url in your browser.

Great. With that out of the way, from here on, we're ready to work on actual feature development.
