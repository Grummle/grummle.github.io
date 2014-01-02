---
layout: basic
title: "Angular From Scratch: Day 4" 
---

Ok, so we've got the page up and it loads info on who the user is and puts some easy stuff in the index.html. We also have a little 
directive that puts up a nice little badge if we are on the DEV/CI/STAGING versions of the site.

So my choices for today are:

-   Menus
-   Login $http interceptor

<!--more-->

I'm thinking the interceptor just cause I haven't done it yet. [This video](http://www.youtube.com/watch?v=W13qDdJDHp8) talks about the new 'around' 
interceptors in 1.2.0 and so thats where we are going to start looking. The idea is that if a request is made and a 401 is returned that a dialog 
should pop up for login and after the user logs in succefully continue with the request. No idea how to do this...  

_Google Montage_  

Ok, so looks like this is gonna be way more cake then I thought it would be. I found [this](https://github.com/witoldsz/angular-http-auth) and it appears that it'll 
basically take care of the 401 interceptor for me. It looks very tight so far. It waits for a 401 in the response and if it finds it emits a event from $rootScope. 
At which point you can handle it and then call authService.loginConfirmed() and it'll replay the previous failing requests. Marvelous!  

As for the dialog that needs to be displayed, I toyed with the idea of maybe making my own dialog handler. I've got to do something after all, but after a quick peek under 
the hood of ui.bootstrap's dialog handler I decided, uh no. So [ui.boostrap](http://angular-ui.github.io/bootstrap/) $dialog it is then.  

So far we've got the Auth interceptor loaded as a module so thats done. ui.bootstrap is also a module so thats done. I added a angular run block to register a listener 
for the auth services 'event:auth-loginRequired' event and handled it by calling the $dialog service. Keeping it in a run block I exclude from the unit tests makes my 
    testing life easier. Which I'm all for. What I've got left.   

-   Login Dialog
   -   Html
   -   Conroller
-   Login Service

And of course all the assciated testing.


A word on project layout. After some futzing with many diffrent MVC'ish type things I've come to _loath_ the whole ```controller/```  and ```view/``` layout. In order to keep both 
organized you end up with a morass of folders and then when you are working on them in tandem which I seem to be always doing you've got 14 levels of folders open at the same 
time and it just annoys the everliving hell out of me. I get lost in the dir structure. So f that we are going to try something different. What I came up with is:   

![this](/images/pagesdir.PNG)

Now everything is all together for pages. That makes sense to me. Yes, yes the html is under the js directory. HORROR! If I come up with a better name for the ```js``` dir later I'll change 
it. So now the Auth.run.js looks like:   

```javascript
angular.module('rm').run(function ($rootScope, $dialog) {
    $rootScope.$on('event:auth-loginRequired', function () {
        $dialog.dialog({
            templateUrl: 'js/pages/loginDialog/loginDialog.html',
            controller: 'LoginDialogCtrl',
            //don't exit on background click
            backdropClick: false,
            //don't exit on esc key
            keyboard:false
        }).open();
    });
});
```

And we'll rip off the html/css from the current login page so its all pretty like. And now when I go to the site without being logged in I get:  
![login](/images/loginfull.PNG)

Which I have to say I think is brilliant :) Alright now we need to flush out the LoginDialogCtrl.  

_TDD Montage_

OK the LoginDialogCtrl ends up looking like so:  

```javascript
angular.module('rm').controller('LoginDialogCtrl', function ($scope, Login, authService, dialog) {
    $scope.login = function () {
        Login.go($scope.acctKey, $scope.username, $scope.password).then(function (response) {
            if (response.success) {
                authService.loginConfirmed();
                dialog.close();
            }
            else {
                response.type = 'error';
                $scope.$broadcast('notify:login', response);
            }
        });
    };
});
```

The truly horrible looking tsts for LoginDialogCtrl look like this:  

```javascript
describe('LoginDialogCtrl', function () {
    beforeEach(module('rm'));

    var loginDialogCtrl;
    var scope;
    var Login;
    var authService;
    var $q;
    var $rootScope;
    var dialog = function () {
    };
    dialog.close = function () {
    };

    beforeEach(inject(function (_$rootScope_, $controller, _$q_, _Login_, _authService_) {
        $rootScope = _$rootScope_;
        scope = $rootScope.$new();
        authService = _authService_;
        $q = _$q_;
        Login = _Login_;
        //injects dialog, ui.bootstrap $dialog
        loginDialogCtrl = $controller('LoginDialogCtrl', {$scope: scope, Login: Login, dialog: dialog});
    }));

    it('should login user', function () {
        sinon.stub(Login, 'go').returns($q.when({success: true}));
        sinon.spy(authService, 'loginConfirmed');
        scope.login();
        $rootScope.$digest();
        expect(authService.loginConfirmed.calledOnce).to.be.ok();
    });

    it('should not login user', function () {
        sinon.stub(Login, 'go').returns($q.when({success: false}));
        sinon.spy(authService, 'loginConfirmed');
        scope.login();
        $rootScope.$digest();
        expect(authService.loginConfirmed.calledOnce).to.not.be.ok();
    });

    it('should close dialog on sucksless', function () {
        sinon.stub(Login, 'go').returns($q.when({success: true}));
        sinon.stub(authService, 'loginConfirmed');
        dialog.close = function () {
        };
        sinon.spy(dialog, 'close');
        scope.login();
        $rootScope.$digest();
        expect(dialog.close.calledOnce).to.be.ok();
    });

    it('should broadcast fail message', function () {
        var response = {success: false, message: 'Oh, hello no!'};
        sinon.stub(Login, 'go').returns($q.when(response));
        sinon.stub(authService, 'loginConfirmed');
        dialog.close = function () {};
        sinon.spy(scope, '$broadcast');
        scope.login();
        $rootScope.$digest();
        expect(scope.$broadcast.calledWith("notify:login", {success: false, message: 'Oh, hello no!', type:'error'})).to.be.ok();
    });
});
```
When we get back success for the login we call  authService.loginConfirmed and it then starts makign the calls again that had failed with 401.
You'll notice the call to $scope.$broadcast if there is an error. Our login endpoint returns info like the following on a fail.  

```javascript
{
    "success": false,
    "message": "Your account key, username or password is invalid."
}
```

So we take that mark it as an erorr and broadcast it to a notifier directive. The notifier directive looks like:

```javascript
angular.module('rm').directive('rmNotifier', function ($timeout) {
    return {
        template: '<div ng-repeat="note in notifications" ng-class="\'alert-\'+note.type" class="alert alert-block" >' +
            //    '<button class="close" data-dismiss="alert">x</button>' +
            '<span>{{note.message}}</span>' +
            '</div>',
        scope: {},
        link: function (scope, elements, attrs) {
            scope.notifications = [];
            scope.$on('notify:' + attrs.rmNotifier, function (event, args) {
                args.type = args.type || 'error';
                scope.notifications.push(args);
                if (!args.perm) {
                    $timeout(function () {
                        scope.notifications = _(scope.notifications).filter(function(note){return note !== args;});
                    }, 4000);
                }
            });
        }
    };
});
```

I did a bad thing and didn't write tests for the notifier. I'll rectify that tomorrow. I promise :) So all together it
looks like this on fail.  

![happy fail whale](/images/loginfail.PNG)

And on success login it just ditches the dialog and your good to go.   


I'm actually writing this the next day. I found quirk last night that drove me to distraction. I was trying to write a protractor 
e2e test to test the login dialog. The first issue I hit was I didn't know how to 'wait' for an element. If you just bang out something like:  

```javascript
        ptor.findElement(protractor.By.id('accountKey')).sendKeys('fark');
        ptor.findElement(protractor.By.id('username')).sendKeys('fark');
        ptor.findElement(protractor.By.id('password')).sendKeys('fark');
        ptor.findElement(protractor.By.id('loginDialogSubmit')).click();
        ptor.findElement(protractor.By.className('alert-error')).then(function(){done()});
```

You'll get an error everytime. It takes the $http call some time to return and for the $digest to complete so when you looke for the alert it 
doesn't exist yet. You have to modify it to be something like.

```javascript
        ptor.findElement(protractor.By.id('accountKey')).sendKeys('fark');
        ptor.findElement(protractor.By.id('username')).sendKeys('fark');
        ptor.findElement(protractor.By.id('password')).sendKeys('fark');
        ptor.findElement(protractor.By.id('loginDialogSubmit')).click();

        ptor.wait(function(){
           return ptor.isElementPresent(protractor.By.className('alert-error'));
        },5000).then(function(){
                done();
        });
```

This'll give angular 5s to get its act together and get the error up on the screen. Selenium webdriver looks like its a big topic and I look forward 
to mastering the quirks and learning some patterns to deal with our typical use cases. Until then hacky fudge will work.  

It was during this that I discovered my sweet grunt/mocha/selenium tests where broke. If an exception is throw during an async mocha test grunt gets it an 
blows its top and exits hard. This qoute from [grunt-mocha-hack](https://github.com/gregrperkins/grunt-mocha-hack) sums it up quite nicely.  

```
"Grunt really doesn't like running after uncaughtExceptions.

But Mocha really likes letting uncaughtExceptions run free."
```

So after furious googling and following every link in a dozen different github issues I was still pretty stumped. I tried domains, but though they allowed me to catch the error I wasn't able to get the error back to mocha so it didn't really buy me what I was looking for. Ended up doing what I hate and running around making changes trying a bunch of stuff and messing up the nice code I'd created all 
day. So now I get to fix it all commit it with the issue and then go back and look for a solution and futz with it from a solid starting place.

![inhack](https://github-camo.global.ssl.fastly.net/29ea39bd791c6424cdac2d6c3d03523de58e9738/687474703a2f2f692e696d6775722e636f6d2f315130396d436a2e706e67)

