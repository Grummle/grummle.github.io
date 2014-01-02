---
layout: basic
title: "Angular: Nursing" 
---

My wife is currently nursing so we are going to sneak in a little 'work'.  

<!--more-->

We left off with a decent start to our angular app. One thing that was bothering me was that I had hard coded some of the data 
for things like urls and username/passwords in the tests. We keep most if not all of that stuff in our web/app.config. It'd be 
nice if that stuff was available to our node/grunt stuff.

_Google Montage_  

Ok so we can use xml2js to read the app.config file. Some something like this.  

```javascript
var getConfig = function () {
        var parser = new xml2js.Parser();
        var deferred = q.defer();
        fs.readFile('../../Reachmail.Tests/app.config', {encoding: 'utf8'}, function (err, data) {
                parser.parseString(data.replace(/^\uFEFF/, ''), function (err, result) {
                    var config = {
                        testing: {},
                        web: {
                            sites: {
                                reachmail: {}
                            }
                        }
                    };
                    config.testing.account = result.configuration.reachmail[0].testing[0].account[0].$;
                    config.web.sites.reachmail.domainName = result.configuration.reachmail[0].web[0].sites[0].reachmail[0].$.domainName;
                    deferred.resolve(config);
                });
            }
        )
        ;

        return deferred.promise;
    };
```

Not the prettiest thing in the world but it does appear to have the virtue of working. So we'll mod our e2e test to be more better.  


```javascript
    it('should exit dialog on successful login', function (done) {
        ptor.get(basePath + '/app');

        ptor.findElement(protractor.By.className('login')).isDisplayed()
            .then(function (d) {
                ex(d).to.eql(true);
            });
        ptor.findElement(protractor.By.id('accountKey')).sendKeys(config.testing.account.key);
        ptor.findElement(protractor.By.id('username')).sendKeys(config.testing.account.username);
        ptor.findElement(protractor.By.id('password')).sendKeys(config.testing.account.password);
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
```

Alright. Not the coolest thing in the world but it removes the worry about making sure js isn't deployed accidently with usernames and passwords included.
