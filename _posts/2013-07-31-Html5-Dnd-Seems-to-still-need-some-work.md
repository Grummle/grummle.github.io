---
layout: basic
title: Html5 DND into document/body
---

So I'm trying to implement a drag and drop file upload somewhat like [imgur](http://imgur.com/) or [Github's issues](http://github.com).
 I started with Eric Bidelman's artcle ['Native HTML5 Drag and Drop'](http://www.html5rocks.com/en/tutorials/dnd/basics/#toc-dragover-dragleave),
 and its a great article that goes into more depth then I'd seen before.

 My use case is that we have an HTML editor that our clients use to create an email. We've had clients recently that have been
 dragging and dropping images from other tabs/pages into the editor and depending on what the content was inserting a full image of the form:

```html
<img src="data:image/jpeg;base64,blargdatablargdata....">
```

Which because we weren't checking for it was actually going out in emails. Not good.
So my first job was to stop people from dragging and dropping those types of images in. Suprisingly Google Images was a huge source.
If you go to Google Images and do a search then check the image elements alot of them have "data:image/jpeg" format.
Seems kinda odd to me, though I didn't take the time to really figure it out. So with FF and Chrome at least what would happen
is that when a person would grab and image and drag it over it the ```dataTransfer``` object would have the option for ```'text/html'```
as you'd expect but it'd be hundreds of kilobytes. To fix it I added a listener to the editor element:

```javascript
        if (event.dataTransfer.files.length !== 0) {
            event.preventDefault();
        }
        if (event.dataTransfer.getData('text/html').indexOf('data:') != -1) {
            event.preventDefault();
        }
```

Hacktastic! So if they bring a file in from the desktop it'll be all 'Hell no' and if the ```html "text/html"``` version has
the 'data:' in it it'll swat it down as well.

So I came up with the following:

![Uploading Example](/images/uploading.gif)

Obviously I'm not a designer. Seeing as we don't have one I don't think its _too_ bad. So the first goal for me was to
have the overlay (opacity lost in gif translation) with the upload area appear when someone drags in a file. Ok, that should be easy.

```javascript
$document.on('dragenter',function(e){
    $scope.uploadVisible = true;
    $scope.$apply();
});


$document.on('dragleave',function(e){
    $scope.uploadVisible = false;
    $scope.$apply();
});
```

And first fail. It appears that instead of firing off the event for docuemnt (or even body in another iteration) it fires
off events for each child. Well thats not helpful. So using a suggestion from [SO](http://stackoverflow.com/questions/3144881/how-do-i-detect-a-html5-drag-event-entering-and-leaving-the-window-like-gmail-d)
I tried doing a counter to keep track.

```javascript
    var where = 0;
    $document.on('dragenter', function (e) {
        where = where + 1;
        if (where > 0) {
            $scope.displayUpload = true;
            $scope.$apply();
        }
    });

    $document.on('dragleave', function (e) {
        where = where - 1;
        if (where === 0) {
            $scope.displayUpload = false;
            $scope.$apply();
        }
    });
```

And it works....well kinda. At first I'd thought I had licked the issue, but after moving on I thougth about it for a little
bit and I realized if you do a drag _internal_ to the page this is triggered as well. The page I'm putting this on has a
WYSIWYG editor. The editor lets you drag content around like images and such. So I tried it and sure enough with this iteration
users that drag stuff inside the editor cause the upload overlay to pop up. Doh. I was curious how other people had implemented it
so I gave imgur's a triy and sure enough it suffered from the same issue. If you open up imgur and grab any image on the page
and drag it'll also cause their upload overlay to come up. Not that big a deal for imgur, but kind of a deal breaker for us.

Ok third times a charm. I think, hot outta the vim emulator is:

```javascript
            var internalDrag = false;
            var where = 0;
            $document.on('dragstart', function () {
                internalDrag = true;
            });

            $document.on('dragend', function () {
                internalDrag = false;
            });

            $document.on('dragenter', function (e) {
                if (internalDrag) return;
                where = where + 1;
                if (where > 0) {
                    $scope.displayUpload = true;
                    $scope.$apply();
                }
            });

            $document.on('dragleave', function (e) {
                if (internalDrag) return;
                where = where - 1;
                if (where === 0) {
                    $scope.displayUpload = false;
                    $scope.$apply();
                }
            });
```

If we can see the ```dragstart``` then it had to happen in the browser, so make a note of it. If it didn't then we know
it started somewhere else and we can put up the upload overlay. Works in Chrome 28, but looks like FF 22 has some issues still.



:x
