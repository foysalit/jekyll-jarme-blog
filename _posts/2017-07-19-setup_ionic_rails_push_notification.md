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

Now, let's quickly setup our ionic app, again, assuming you already have nodejs, android sdk, cordova and ionic cli setup on your machine (if not, please follow [this guide](https://ionicframework.com/docs/intro/installation/)) :
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

Setup Google FCM project
========================
For this part, you'll need a google account. Follow these steps to get your firebase project setup and get the required data for our next steps :

- Go to [firebase console](https://console.firebase.google.com) and login with your google account.
- Add a new project with your preferred name and server region. I have named my project `pushme`. It looks something like the image below at the time of writing this post.

![Create firebase project screenshot](/assets/article_images/2017-07-19-setup_ionic_rails_push_notification/create_firebase_project.png)

- Now from your project page, go to the project settings page.

![Go to firebase settings page](/assets/article_images/2017-07-19-setup_ionic_rails_push_notification/firebase_settings_page.png)

- On the settings page, go to *Cloud Messaging* tab. You will find a *Server Key* and a *Sender ID*. We will need these two keys in a later stage.

![Firebase FCM credentials](/assets/article_images/2017-07-19-setup_ionic_rails_push_notification/firebase_fcm_credentials.png)

*I will be removing my project once I'm done writing this post so I'll just show sensitive keys on the post but make sure you have your keys safe*.

Setup Push and Device Plugins
===============================
In order to send push notification to a device, we need a unique key from each device called `registrationId`. the phonegap-plugin-push plugin allows us to retrieve that with a nice api. Once retrieve, we will send and store it on our api server for later use.

To make things easy, ionic has a bunch of packages under `@ionic-native` namespace that bridges ionic with various phonegap/cordova plugins and provides us a intuitive typescript compatible interface. Let's install the following cordova plugins and ionic native npm packages :
```
ionic cordova plugin add cordova-plugin-device
ionic cordova plugin add phonegap-plugin-push --variable SENDER_ID=710571851300
npm i @ionic-native/push @ionic-native/device --save
```
In the second command `SENDER_ID` variable make sure you replace the *710571851300* with your project's `SENDER_ID`.

Now we need to register the ionic-native services in our `src/app/app.module.ts` file in order to use them within our app so let's adjust the file like this :
```
....other imports.....
import { Push } from '@ionic-native/push';
import { Device } from '@ionic-native/device';
.....
    providers: [
        ...other providers,
        Device,
        Push,
```
To keep things modular we will extract the entire notifications feature into it's own service provider. The following command will generate a service provider placeholder for us and register that in the `app.module.ts` file as well.
```
ionic g provider notification
```
The above command will generate a new file in `src/providers/notification.ts`. Let's open it up and inject our new push and device plugins into the provider. These are the changes :
```
import { Device } from '@ionic-native/device';
import { Push, PushObject, PushOptions } from '@ionic-native/push';
import { Injectable } from '@angular/core';
import { Http, Headers, RequestOptions } from '@angular/http';
......
....
export class NotificationProvider {
    constructor (
        public http: Http,
        public push: Push,
        public device: Device
    ) { }

    init () {
        const options: PushOptions = {
            android: {
                senderID: XXXXXXXXX
            }
        };

        const pushObject: PushObject = this.push.init(options);

        pushObject.on('registration').subscribe((registration: any) => {
            console.log(registration);
        });
    }
```
Notice we're importing `{ Http, Headers, RequestOptions }` and injecting `Http` in the constructor but never using it. Don't worry, we will use these in the later steps.

Now we're gonna inject this new provider into our `app.component.ts` file and initialize it :
```
constructor(
    platform: Platform,
    statusBar: StatusBar,
    splashScreen: SplashScreen,
    noti: NotificationProvider
) {
    platform.ready().then(() => {
        // Okay, so the platform is ready and our plugins are available.
        // Here you can do any higher level native things you might need.
        statusBar.styleDefault();
        splashScreen.hide();
        noti.init();
    });
}
```
If you've followed along the above steps, you'll notice that we're running out app in the browser and cordova `Device` and `Push` features are only available on an actual device. So in order to move forward, we need to run our app on an emulator or an actual device. I've turned on *developer mode* on my phone and connected to my machine. The following commands will install and run the app on the connected device :
```
ionic cordova platform add android
ionic cordova run android --device
```  
If all goes well, you should now see the ionic app running on your phone. Our next step is to take the device registrationId from our device and store it on our rails server. So let's equip our rails api with that feature.

Device Registration On Rails
=================================
Rails can be lightning fast and super intuitive when setting up trivial features like basic CRUD and luckily that's what we need for the first part of this section.

We will setup a device resource that can :
- save a device identified by it's uuid and store the push registrationId of that device
- update the registrationId of a device identified by it's uuid.

Quite straight forward, so let's get to it. The following command will scaffold the route, model, controller, migrations and the tests for our device resource.
```
rails g scaffold devices model:string uuid:string token:text platform:string
# migrate your database to add the device table
rake db:migrate
```
Now we can simply make post request to `localhost:3000/devices.json` with `uuid`, `model`, `platform` and `token` parameter and rails will save it in the database and return a json response to our call.

Send Device Data to Rails
=========================
Let's go back to our `src/providers/notification/notification.ts` file and add the following code :
```
...........
    pushObject.on('registration').subscribe((registration: any) => {
        this.saveToken(token);
    });
}

saveToken (token) {
    // build the headers for our api call to make sure we send json data type to our api
    const headers = new Headers();
    headers.append("Accept", 'application/json');
    headers.append('Content-Type', 'application/json' );
    const options = new RequestOptions({ headers: headers });

    // this is our payload for the POST request to server
    const data = {
        platform: this.device.platform,
        model: this.device.model,
        uuid: this.device.uuid,
        token
    };
    const url = "http://localhost:3000/devices.json";

    this.http.post(url, data, options)
        .subscribe(data => {
            console.log('token saved');
        }, error => {
            console.log('error saving token', error);
        });
}
```
When we run the above code on our device, there will be slight bit of problem with the http call. In order to see what's broken here, open up chrome browser and go to `chrome://inspect`. You will see your device and app under the device on that page. @TODO add image here. Click the app name to open up developer's console and you'll see that the POST request is 404'ing. Since the app is running on an actual device, `localhost:3000` is trying to find that server within your phone and obviously there is no server running on port 3000 on our phone.

To work around this, we need to use our computer's local network ip address as our api endpoint. If you run `ifconfig` in your terminal and look at the output, you should find a line that looks like the content in the red square in the image below. `192.168.107` is the local ip of my machine here.

![ifconfig output](/assets/article_images/2017-07-19-setup_ionic_rails_push_notification/ifconfig_output.png)

So let's replace our api url `const url = "http://localhost:3000/devices.json";` with this `const url = "http://192.168.107:3000/devices.json";`. Again, this is my local ip so when you're following along this, make sure you use your local ip that you found from the `ifconfig` command.

However, this will still fail to make the call since our rails app is bound to localhost and now our local ip of our host machine which is why any calls made to our local ip never really reaches our rails app. To fix that, let's stop our rails using `Ctrl+c` and restart it with this command `rails s -b 192.168.1.107` which binds our server with the `192.168.1.107` ip.

Finally, we have a proper communication setup between our ionic app and rails api and if you reboot the app on device, you'll see on network tab of the developer's console that it's making the api call successfully.

Send Notification Using Rpush
=============================
