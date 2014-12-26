---
layout: blog
title: "Angular Tree view hunt"
---

So we don't typical pair at work. I feel like I'm so much more productive when we do occasionally pair. I have noticed that I am more productive when I'm blogging as I go. I think its cause I have to actually write what I'm going to do next, like I would when pairing. So here we go again.

<!--more-->

##Current project
I'm working on converting an existing page from Backbone to Angular. The page needs significant upgrades to satisfy the client and I don't want to have to deal with the Backbone anymore. Adding some objects to handle the incoming data was easy enough. Getting the scaffolding setup also not a big deal as we have quite a few Angular pages already.

##Treeview
This is the current sticking point. I'd used a Backbone treeview I found somewhere, and it worked great. It was a little heavy depending on several other jQuery plugins. Now I've been tasked with finding one for Angular. There are quite few out there but I've got a couple requirements.

* Lazy loading
* multi select

So lets look at some candidates.

###[angular.treeview](https://github.com/eu81273/angular.treeview) 
[fiddle](http://jsfiddle.net/8LWUc/1350/)  
At first glance its missing multi select and any built in support for lazy loading lower leaf nodes. However its also very simple. So maybe we can mod it.

Marking the selected node function is simple enough and so adding support for Ctrl clicking should be easy.

```javascript
scope[treeId].selectNodeLabel = scope[treeId].selectNodeLabel 
                                        || function( selectedNode){
    //remove highlight from previous node
    if( scope[treeId].currentNode && scope[treeId].currentNode.selected ) {
        scope[treeId].currentNode.selected = undefined;
    }

    //set highlight to selected node
    selectedNode.selected = 'selected';

    //set currentNode
    scope[treeId].currentNode = selectedNode;
};
```

For the lazy loading I don't see a quick obvious answer. Hmmm...guess we'll go on to another.

###[angular-ya-treeview](https://github.com/ca77y/angular-ya-treeview)
[plunker](http://plnkr.co/edit/mAfWCLmD9NW0CD44gdNF?p=preview)  
This one looks pretty promising. Lazy loading and multi-select are listed as design goals.

Multiselect is demonstrated in the plunker.

However looking at the code I'm seeing that it doesn't actually supoort 'lazy' loading of children, just async loading meaning that it puts them in a $timeout with no value to move them to the next tick. Not what I'm looking for.

```javascript
 var fillChildrenNodes = function(node, value) {
            if (node.$hasChildren) {
                $timeout(function() {
                    angular.forEach(node.$children, function(node) {
                        if (node.$hasChildren) {
                            var children = YaTreeviewService.children(node, options);
                            node.$children = value || YaTreeviewService.nodifyArray(children, node, options);
                        }
                    });
                });
            }
        };
```

I might be able to hack something in via the 'onExpanded' event, but we'll go play around with some other contenders before I commit to modding something.

###

