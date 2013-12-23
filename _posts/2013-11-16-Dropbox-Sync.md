---
layout: basic
title: "Dropbox Sync" 
---

So I talked about wanting to use [dropbox-sync-js](https://github.com/Phlaphead/dropbox-sync-js) to synch a dropbox account with pictures of my daughter.
The library seemed to be very well written though not quite general enough for my purposes on further examination. So time to mod it and see if we can't 
contribute back to the project in the process.  

So the major things I need to address are:  
- Load as module and not run synch
- Ability to specify config file location.
- Ability to specify sync file location.

###Load as Module
Currently when you require the library it runs the synch function which isn't the behaviour I'm looking for. So we'll mod it using this wonderul [example](http://stackoverflow.com/questions/8864365/can-i-know-in-node-js-if-my-script-is-being-run-directly-or-being-loaded-by-an).

```javascript
exports.doSync = doSync;

if(!module.parent){
    main();
}

```

Easy enough. if we run it from the command line like `node sync.js` then it'll run the main function and sync, but if we require it from another place we'll have to call it ourselves.

###Config File Locations
This one is quite a bit tricker. The script as currently written puts the config file and synch files in the home directory and doesn't give you any option about changing that. This has a couple side effects I'm not crazy about. On mac for some reason I have to `sudo node sync.js` in order to get the script to read the config and sync files properly. Not cool. Also besides the hard coded location for synch and config I don't see any other reason it couldn't support multiple dropbox accounts. 