---
layout: basic
title: "Angular From Scratch: Brownfield Everywhere....."
---


At my current job, [Reachmail](http://www.reachmail.com), we've got a disgusting brownfield app. We've been working our way through a plethora of 
different technologies, methodoligies and theologies. We decided awhile ago that a one page app makes alot of sense in our case. 

<!--more-->

-   People spend alot of time in the app per sitting, so js load times shouldn't be an issue overall.
-   Its an app that isn't public, meaning that SEO isn't an issue
-   The complexity of alot of the interaction is a royal pain with postbacks  

We initially started with backbone and it was amazing. It truly changed the way I think of development. I told my boss, There is no way 
I'm going back to developing in web forms or even exclusively MVC. As we started to work with backbone more and more we ran into more and 
more issues that far from being show stopper just seemed ... stupid. Seriously wtf do I have to do all this _shit_ all the time?

**Enter AngularJS**  

I decided to read the tutorial on my bus trip home one day. Since then I've been drinking the kool-aid. When I finally got my head around 
expressions I just about shit myself. It was such an elegant way to remove so much _crap_. Since then I've been bashing my skull against 
Angular on a regular basis, and I've finally decided its time to get my thumb out and get this party started. This blog post and the following 
ones are going to be an absolute mess. I'm using it as a scratch pad/rant/log of my journey in setting up and actual large scale Angular app from 
scratch. I've played with Yomen a little and I've decidded that I'm going it alone on this one. Alot of the decisions made in Yoman for angular apps 
I don't think will work for my organization. Eventually I'll probably end up with something that Yomen could have given me faster, but to be honest if 
I don't set up a system then I've got a good chance of having no clue wtf is going on within that system. So here goes.

**Meet the new site, same as the old site**  

We recently went through a 'redesign', and I say that with qoutes because we had a graphic designer, [Adam Fox](http://adamfox.com/) design/cut/code about three pages for us. 
My co-worker [Brian Kothe](http://www.linkedin.com/in/briankothe) took those three pages and extrapalated/applied Adam's design to the rest of the site. 

At this point I should mention that the site is a hodpodge of classic ASP, ASP.net WebForms, ASP.net MVC3 and FUBUMVC. Classic ASP is by far the leader for shear vollume. 
BK went through and touched _every single page_ on the site and basically made it mostly sane. The amount of work he did and quality of the work was awe inspiring. It helps that 
he's pretty OCD about his code :) He used bootstrap and sanitized the vast majority of the html css on the site. So the site is now relatively sane when it comes to html/css. 
The majority of the code however still can't take advantage of all the work me and my co-workers have put into crafting a new backend in .net.

So now that we've got a site that visually, usually doesn't make me want to slit my wrists its time to bring it into the new world.   

I'm create a directory called ```/app``` and it'll be the new sites home. Visually its gonna look identical to the current site for the most part. The goal is for 
the users to be able to move between old crap and new hotness without them realizing it. So we can move chunks of the site over to Angular as we go and touch different parts. 
I considered just using angular embeded into the current site, and have in a couple of places, but I want a clean break.   
  
Ok so now we've got app lets get ```index.html``` rolling.  

So pulling in the base html from our app we end up with  
  
![Reachmail Base](/images/reachmail1.png)

Where to go from here:  
-   Menu: Reachmail has a menu that sticks around on all of the pages...
-   populate ? placeholders on this page. Its an html page and has no concept of who is looking at it.
-   Library decison time.
-   Testing: Already, bet your bipper. I've gotten 2 or three pages into this in the past and _every_ time it gets out of hand _so_ fast.

**Test early, Test often**  
Ok so testing wins. Since I like to keep up with the lastest and greatest (esp when its better then old and busted) I've been looking at Angular [Protractor](https://github.com/angular/protractor). My understanding is that its 
a wrapper for WebdriverJS that is Angular aware and is displacing the angular scenario runner. So a wonderful first test in my mind is just making sure the damn index.html comes up. How you do that?  

Shit, time for a an excursion. How to get protractor working well and how to make it easy to use and setup for my compatriates.   

-   Protractor itself
-   Running protractor tests
-   What testing framwork and assertion library

Alright protractor can be gotten easy enough.... ```npm install protractor``` Now where do I put it....
Ok so I'm putting this all in app. Just guessing at this point. New package.json with ```npm init```

```npm install --save-dev protractor```  
```npm install --save-dev mocha```  
```npm install --save-dev expect.js```  
```npm install selenium-webdriver --save-dev```

So I'm gonna steal the ```node_modules/protractor/examples/onMocha.js``` to get started. Then mod it to from ```lib/protractor.js``` to just ```protractor``` so we use the npm module.  

Now comes the pita. I need a selenium server for protractor to connect to. How can I 'get' that and make easily 'gettable' for my comrades.... Please wait, franticaly googling....  

Ok gonan half ass it. I'm using the install script provided by protractor at ```/node_modules/protractor/bin/instal_selenium_standalone``` and gonna commit it to git. Not ideal but it works.  
Ok we'll fire up the selenium server. and try and run the tests in onMocha.js to make sure this is working....  
Yay! Test run against agnular site. So now our dir struct is looking like the following.  
![Basic Dir So far](/images/dir1.PNG)  

Now time to mode onMocha.js to something more civalized. Hmmm.... Basic.spec.js

```javascript
var util = require('util');
var expect = require('expect.js');
var webdriver = require('selenium-webdriver');
var protractor = require('protractor');

describe('RM Angular App', function() {
    this.timeout(80000);

    var driver, ptor;

    before(function() {
        driver = new webdriver.Builder().
        usingServer('http://localhost:4444/wd/hub').
        withCapabilities(webdriver.Capabilities.chrome()).build();

        driver.manage().timeouts().setScriptTimeout(10000);
        ptor = protractor.wrapDriver(driver);
    });

    after(function(done) {
        driver.quit().then(function() {done()});
    })

    it('Should Get Main page', function(done) {
        //use driver to get that page. Protractor waits for angular, and we ain't got it yet.
        ptor.driver.get('http://go.dev.reachmail.net/app');

        ptor.driver.findElement(protractor.By.className('rmlogo')).
        then(function(){done();});
    });
});
```

Ok so now we've got a basic test to see that the app page loads. Outsanding!!! Shit still got an hour and 45 minutes left to work. Alright fine, but I'm at least 
gonna tackle something fun...

**Grunt**  
So I've seen grunt all over and I've worked with when contributing to OSS, but never set it up or tried to figure it out myself. 
Guess its time to go down that rabbit whole. What do we need it to do right now?  

-   install all npm dependencies
-   start selenium server
-   run mocha tests
-   watch for file changes and rerun mocha tests

Ugh that list looks unfun.

Reading [Grunt docs](http://gruntjs.com/getting-started).....


