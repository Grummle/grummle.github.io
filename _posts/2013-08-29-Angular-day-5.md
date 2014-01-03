---
layout: blog
title: "Angular From Scratch: Day 5" 
---

Ok so today we get start by putting everything I destroyed last night back into shape. Ugh, you'd think I'd have learned this lesson by now.  

<!--more-->

[_Housekeeping_](http://www.youtube.com/watch?v=3QCSrQEGvZA)


Ok, so I put everything back and it all works, but it has the issue where if the mocha tests fails then grunt blows up and you don't get a propery error report. 
So now time to experiment with a simple async test in mocha to see if it will throw the same error. Something like:  

```javascript
describe('blarg',function(){
    it('should blow all upish',function(done){
        throw new Error('kablewy');
    });
});
```

And grunt still folds like a house of cards so we know its a no go. Ok I read some good things about grunt-mocha-test. Lovely easy  

```bash
npm install --save-dev grunt-mocha-test
```

And we got it, ripping off the config from the example we get  

```javascript
   mochaTest: {
            e2e: {
                options: {
                    reporter: 'spec',
                    // tests are quite slow as thy spawn node processes
                    timeout: 10000
                },
                src: ['test/e2e/*.spec.js']
            }
        }
```


but we need to make some adjustments to the spec files as we aren't starting up the selenium server before we test anymore. So quick utils.js for our tests.

```javascript
var webdriver = require('selenium-webdriver');
var protractor = require('protractor');

var startServer = function () {
    var remote = require('selenium-webdriver/remote');

    var server = new remote.SeleniumServer('test/selenium/selenium-server-standalone-2.34.0.jar', {
        port: 4444,
        args: ['-Dwebdriver.chrome.driver=test/selenium/chromedriver.exe']
    });

    var driver, ptor;

    return server.start()
        .then(function () {
            driver = new webdriver.Builder().
        
                usingServer('https://localhost:4444/wd/hub').
                withCapabilities(webdriver.Capabilities.chrome()).build();

            driver.manage().timeouts().setScriptTimeout(10000);
            return protractor.wrapDriver(driver);
        });
};

exports.startServer = startServer;
```

Ok and we'll use that in the tests like so...

```javascript
var util = require('util');
var utils = require('./utils');
var ex = require('expect.js');
var protractor = require('protractor');

describe('Login', function () {
    this.timeout(10000);

    var ptor;

    before(function (done) {
        utils.startServer().then(function (p) {
            ptor = p;
            done();
        });
    });

    after(function (done) {
        ptor.driver.quit().then(function () {
            done();
        });
    });


    it('should exit dialog on successful login', function (done) {
        ptor.get('https://go.dev.reachmail.net/app');

        ptor.findElement(protractor.By.className('login')).isDisplayed()
            .then(function (d) {
                ex(d).to.eql(true);
            });
        ptor.findElement(protractor.By.id('accountKey')).sendKeys('realaccount');
        ptor.findElement(protractor.By.id('username')).sendKeys('realuser');
        ptor.findElement(protractor.By.id('password')).sendKeys('realpassword');
        ptor.findElement(protractor.By.id('loginDialogSubmit')).click();

        ptor.wait(function () {
            return ptor.isElementPresent(protractor.By.className('login')).then(function (a) {
                return !a;
            });
        }, 10000).then(function () {
                ptor.driver.get('https://go.dev.reachmail.net/process_logout.asp');
                done();
            });
    });

    it('should show warning on failed login', function (done) {
        ptor.get('https://go.dev.reachmail.net/app');

        ptor.findElement(protractor.By.id('accountKey')).sendKeys('fark');
        ptor.findElement(protractor.By.id('username')).sendKeys('fark');
        ptor.findElement(protractor.By.id('password')).sendKeys('fark');
        ptor.findElement(protractor.By.id('loginDialogSubmit')).click();

        ptor.wait(function () {
            return ptor.isElementPresent(protractor.By.className('alert-error'));
        }, 5000).then(function () {
                done();
            });
    });
});
