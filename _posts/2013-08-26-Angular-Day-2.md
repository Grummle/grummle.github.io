---
layout: blog
title: "Angular From Scratch: Day 2" 
---

So, we left off looking for a way to make the E2E tests run with Protractor easier to run. This means we need something to start up the Selenium server, 
then run mocha with the correct files.  

<!--more-->

_Googling and futzing montage_   
  
Ok, so it appears that there is a plethora of stuff out there for this. Doesn't look like I'll be able to make heads or tails enough of it to use it however. 
For instance there is a grunt task 'grunt-mocha-selenium' that looked promising, but doesn't appear to be using WebdriverJS so the instance of selenium it gives back 
can't be use by protractor. On the bright side it appears that the webdriver-selenium package has a remote function to start a selenium server from node. Looks like it 
comes with the events to kill it after the program exits as well, which is nice. So I'm thinking I can just use that to start the server, then find a way to run the mocha 
tests.   

Ok well this turned out to be easier then I'd have thought. I hate/love it when I work on something and go through all kinds of controtions and download all these files, 
setup all these things and then it ends up being a half dozen lines of code in just the right place.   

So I ended up adding a grunt file like the following.  

```javascript
module.exports = function(grunt) {
    grunt.initConfig({});

    grunt.registerTask('mocha_protractor','Run e2e with mocha/protractor',function(grunt){
        var remote = require('selenium-webdriver/remote');
        var Mocha = require('mocha');
        var fs = require('fs');
        var path = require('path');

        var mocha = new Mocha();
        var gruntDone = this.async();

        server = new remote.SeleniumServer('test/selenium/selenium-server-standalone-2.34.0.jar',{
            port:4444,
            args:['-Dwebdriver.chrome.driver=test/selenium/chromedriver.exe']
        });

        server.start().then(function(){
            fs.readdirSync('test/e2e').forEach(function(file){
                mocha.addFile(path.join('test/e2e',file));
            });

            mocha.run(function(failures){
                gruntDone();
            });
        });
    });

    // Default task(s).
    grunt.registerTask('default', ['mocha_protractor']);
};
```

Alrighty then, we now get the following  
![Grunt Output 1](/images/grunt1.png)  
Ok, and time for a side how with the last half hour of the work day. Pretty sad this is all I got done today....

I'd love to have jshint/lint avaialble so that I can bitch/be bitched at/by my coworkers. This is totally a solved 
thing so we'll return after another google montage....  

Ok that easy enought, just get the package and put a small entry into the gruntfile. Of course then I also had to fix my files so they'd lint properly :)

:w

