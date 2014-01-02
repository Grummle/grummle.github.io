---
layout: basic
title: "Angular From Scratch: Day 3" 
---

Ah day 3. It feels like this might become an actual thing. So were where we....  

<!--more-->

-   Library decison time.
-   populate ```?``` placeholders in ```index.html```. Its an html page and has no concept of who is looking at it.
-   Menu: Reachmail has a menu that sticks around on all of the pages...

Time to make some decisions about libraries. So stable Angular is 1.0.7 and is ancient. Unstable that has been around for awhile is 1.1.5, but my understanding is that 
1.2.0RC1 has alot of changes on top of that. A good video with alot of the changes is [here](http://www.youtube.com/watch?v=W13qDdJDHp8). As this probably isn't going 
straight into production in the next week, I think I'm gonna go with the RC and pray they get it moved along to stable before we ship. Now where to get it from. Sweet! 
Bower has it. Not seeing it on Google CDN and a co-worker has been less then impressed with the speed of cdnjs so I guess we are going to be hosting it ourselves.

Ok, time to setup bower config.

``` bash
 bower init
 ```

Fill out that stuff. Now we need to set the directory that bower will install to. We'll have to create a ```.bowerrc``` and add:  

```javascript
{
        "directory": "js/lib/"
}
```

Now we'll add a couple libraries I know we'll need off the top of my head.  

```bash
bower install angular-mocks#1.2.0-rc.1 --save
bower install angular#1.2.0-rc.1 --save
bower install underscore
bower install jquery#1.8.1 --save
```

K, guess that takes care of libraries for now. Saving to git.....
Doh, looks like the underscore project is the full underscore project which we aren't interested in. So looks like we actually want.  

```bash
bower install component-underscore
```

Ok now that we seem to have everything setup lets get some actual freaking code going. First up we need to get the user and basic info for the index page. 
We'll need something like the following...  

```javascript
{
    "companyName": "Reachmail Acceptance Test Account",
    "accountKey": "RMACCEPTTEST",
    "username": "accept",
    "version": "0.0.0.0",
    "roles": {
        "admin": true,
        "mailings": true,
        "lists": true,
        "schedule": true,
        "downloadList": true,
        "advertisers": true,
        "surveys": true,
        "forms": true,
        "image": true,
        "document": true,
        "template": true,
        "outlook": true,
        "mailingReports": true,
        "surveyReports": true
    }
}
```

This actually comes from an endpoint I wrote during a previous attempt to take some things in RM to angular. I think its sound so in the interests of doting the i's and crossing the t's 
I'm going to delve in and make sure its all setup and properly tested on the backend.  

[_Code slaying Montage_](http://www.youtube.com/watch?v=25XyWhfpSiM)

Ok, well apparently I was thorough before and the endpoint is setup as well as having acceptance tests against it (C# test that hits the endpoint and inspects the json returned. 
Props to [MOB](http://www.mikeobrien.net/) for the internal DSL we use to acceptance test the endpoints)  

So now we need to setup up some of the basic angular stuff. Get angular on the page and create a module we start dumping stuff into. We'll add a (suprise!) app.js that creates the rm module. We'll add that to the ```index.html```.   

**Optimizations**  
So at this point we are starting to add js files to our project and we're starting to add them to index.html. If you do a file per function/piece for our app we are going to end up with quite a few files and its going to slow 
load times down. We've flirted with requirejs in the past and I've played with browserify.js on the side a touch, but at this point we really have no idea whats going to be best for us. My current plan right now is that optimizations 
are nice but I shouldn't touch them till I have an actual need. So I'm gonna forge ahead and not worry about doing any optimizations. In the future I think that concatenation will probably get us most of the way there. 
So I'm not gonna worry about how I specify the dependencies either at the moment. Also I'm going to be dumping everything into a single Angular module. Besides breaking out things that you'd 
like to distribute to other audiences I don't currently understand what modules are for and [this](http://www.youtube.com/watch?feature=player_detailpage&v=ZhfUv0spHCY#t=2059) seems to back up my decision.  

So we'll tack all this to the bottom of the body.  

```html
    <script type="text/javascript" src="js/lib/angular/angular.js"></script>
    <script type="text/javascript" src="js/app.js"></script>
</body>
```

Now we need to create a service to make the call and return the 'who' data. At this point its probably time to take a look at unit testing. In the past I know there was an issue with using mocha and angular-mocks. I think its been resolved...  

_Furious googling_

Ok, so looks like its now supported more in 1.2.0RC so we'll try and set this up the normal angular way rather then jump through all the hoops I used to. On a side note those hoops taught me quite a bit about Angular DI. I had to setup the injector myself to get it working with mocha in the past. We'll see if it goes any better now.

_Futzing..._

Ok so wwhat I eneded up with is setting up karma with ```karma init``` and using [karma-mocha](https://github.com/karma-runner/karma-mocha) to help grunt run the tests.   

Got a bit side tracked. I got a .jshintrc from [here](https://gist.github.com/haschek/2595796) and added it in. Then had to make all the files match it. Back to what I was doing. 
Lets see if mocha is a first class citizen in angular testing now. Nice, it appears that it is. I didn't used to be able to do:  

```javascript
    beforeEach(inject(function(_$compile_,_$rootScope_){
    }));
```

As it would moan inject was undefined.

Ok so on to writing the code. So we need a service that will pull the who data for us. So we make a factory for 'who' and we need som test.....

_Furious typeing_

So we end up with this:  

```javascript
app.factory('Who', function ($http) {
    var path = '/api/accounts/who/';

    var Who = function (data) {
        angular.extend(this, data);
            if (this.roles && (this.roles.advertisers ||
                this.roles.surveys ||
                this.roles.forms ||
                this.roles.image ||
                this.roles.document ||
                this.roles.template ||
                this.roles.outlook)) this.roles.tools = true;

            if (this.roles && (this.roles.mailingReports || this.roles.surveyReports)) this.roles.reports = true;
    };

    Who.get = function () {
        return $http.get(path).then(function (response) {
            return new Who(response.data);
        });
    };

    return Who;
});
```
And a test like  

```javascript
describe('Who', function () {
    beforeEach(module('rm'));

    var Who;
    var whoResponse = {
        "companyName": "Reachmail Acceptance Test Account",
        "accountKey": "RMACCEPTTEST",
        "username": "accept",
        "version": "0.0.0.0",
        "roles": {
            "admin": true,
            "mailings": true,
            "lists": true,
            "schedule": true,
            "downloadList": true,
            "advertisers": true,
            "surveys": true,
            "forms": true,
            "image": true,
            "document": true,
            "template": true,
            "outlook": true,
            "mailingReports": true,
            "surveyReports": true
        }
    };
    var $httpBackend;

    beforeEach(inject(function ($injector) {
        Who = $injector.get('Who');
        $httpBackend = $injector.get('$httpBackend');
    }));

    afterEach(function () {
        $httpBackend.verifyNoOutstandingExpectation();
        $httpBackend.verifyNoOutstandingRequest();
    });

    it('should return an object', function () {
        expect(Who).to.not.be(undefined);
        expect(new Who()).to.not.be(undefined);
    });

    it('should get a who from server', function () {
        $httpBackend.when('GET', '/api/accounts/who/').respond(whoResponse);
        $httpBackend.expectGET('/api/accounts/who/');

        var who = Who.get();
        who.then(function (who) {
            expect(who.username).to.eql('accept');
        });

        $httpBackend.flush();
    });

    it('should have report role', function () {
        $httpBackend.when('GET', '/api/accounts/who/').respond(whoResponse);
        var who = Who.get();
        who.then(function (who) {
            expect(who.roles.reports
            ).to.eql(true);
        });
        $httpBackend.flush();
    });

    it('should have tool role', function () {
        $httpBackend.when('GET', '/api/accounts/who/').respond(whoResponse);
        var who = Who.get();
        who.then(function (who) {
            expect(who.roles.tools
            ).to.eql(true);
        });
        $httpBackend.flush();
    });

    it('shouldn\'t have reports role', function () {
        whoResponse.roles = {};
        $httpBackend.when('GET', '/api/accounts/who/').respond(whoResponse);
        var who = Who.get();
        who.then(function (who) {
            expect(who.roles.tools
            ).to.not.be.ok();
        });
        $httpBackend.flush();
    });

    it('shouldn\'t have tools role', function () {
        whoResponse.roles = {};
        $httpBackend.when('GET', '/api/accounts/who/').respond(whoResponse);
        var who = Who.get();
        who.then(function (who) {
            expect(who.roles.tools
            ).to.not.be.ok();
        });
        $httpBackend.flush();
    });
});
```

For some reason it _feels_ fugly to me. Not sure what else to do though.... Anyway on to getting this into our ```index.html```.

Ok so what we can do is make a run block for our app, but I think we'll need to exclude it from our unit tests...yep errors galore.  

_Fiddling with config_

Ok so the run block looks like:

```javascript
angular.module('rm').run(function (Who, $rootScope) {
    $rootScope.who = Who.get();
});
```

And now we can stick some ng-binds in the index.html and get rid of the annoying ```?``` hanging around.

```html
<li><a href="/process_logout.asp" title="Sign Out"><span ng-bind="who.accountKey">?</span> - SIGN OUT</a></li>

ReachMail Version <span ng-bind="who.version">?</span>
```

Since this html is sent and rendered before Angular has a chance to do anything with it you use the ng-bind directive so the interpolation string doesn't 
show up.

You'll notice we are binding directly to the promise returned at who. Angular is smart enough to know its a promise and deal with it. Nice. Ok nex thing is that 
we have an icon'ish thing we stick at the bottom that lets us know when we are on dev and ci. It looks like:  

<div style="color:white;text-shadow:rgba(0,0,0,0.247059) 0px -1px 0px;background-color:rgb(185,74,72);border-radius:3px;width:31px;padding:3px;">DEV</div>

Well more or less. So I think this would be a glorious place to do a directive. We'll make a directive that creates the tag based on the URL the page is currently at. Our scheme 
is like this ```go.dev.reachmail.net``` is our development and ```go.ci.reachmail.net``` is our CI server and ```go.reachmail.net``` is production. So if we look at $location then we 
should be able to tell easily enough where we are at and choose to show the warning or not. Now the trick is to remember how I do unit testing with a directive. I've done it before. With most browsers is cake, but some like firefox don't fire some events when the directive is not attached to the DOM. This one doesn't have any events to listen for so its mute for now.

_Overly hurried coding..._  

So for that directive we end up with.

```javascript
angular.module('rm').directive('rmDevTag',function(){
    return {
        scope:{},
        template:'<div class="environment label label-important" ng-show="environment">{{environment}}</div>',
        replace: true,
        link:function(){
        },
        controller:function($scope,$location){
            var piece = $location.host().split('.')[1].toUpperCase();
            if(piece !== 'REACHMAIL') $scope.environment = piece;
        }
    };
});
```

And the Tests:  

```javascript
describe('rmDevTag', function () {
    beforeEach(module('rm'));

    var template = '<span rm-dev-tag></span>';
    var $compile;
    var $document;
    var $location;
    var scope;

    beforeEach(inject(function ($injector) {
        $compile = $injector.get('$compile');
        $document = $injector.get('$document');
        $location = $injector.get('$location');
        scope = $injector.get('$rootScope').$new();
    }));

    it('should do nothing for production', function () {
        $location.$$host = 'go.reachmail.net';

        var devTag = $compile(template)(scope);
        scope.$apply();

        $document.find('body').append(devTag);

        expect(devTag.hasClass('ng-hide')).to.be(true);
    });

    it('should show CI', function () {
        $location.$$host = 'go.ci.reachmail.net';

        var devTag = $compile(template)(scope);
        scope.$apply();

        $document.find('body').append(devTag);

        expect(devTag.hasClass('ng-hide')).to.be(false);
        expect(devTag.html()).to.eql('CI');
    });

    it('should show DEV', function () {
        $location.$$host = 'go.dev.reachmail.net';

        var devTag = $compile(template)(scope);
        scope.$apply();

        $document.find('body').append(devTag);

        expect(devTag.hasClass('ng-hide')).to.be(false);
        expect(devTag.html()).to.eql('DEV');
    });

    it('should show STAGING', function () {
        $location.$$host = 'go.staging.reachmail.net';

        var devTag = $compile(template)(scope);
        scope.$apply();

        $document.find('body').append(devTag);

        expect(devTag.hasClass('ng-hide')).to.be(false);
        expect(devTag.html()).to.eql('STAGING');
    });
});
```

Another nice thing is that if you are using karma you can click on the debug button and see the output from your directives.  
![karma debug](/images/karmdebug.png) 


Ok that was a nice set of warm ups. Everything is working and now 'We're cooking with Gas!'
                            





