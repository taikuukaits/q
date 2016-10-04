This is two small changes to Q that allow for nicer syntax when executing sequential promises with additional dependencies. 

Personally not many of my functions take 1 parameter. I wish my code was pure enough for that but it isn't.

What I want to write are promises which take multiple arguments and evaluate in series. 

This is what I want:
```javascript
getHomeFolderForUser(user).then(function (folder) {
     return getFilesInFolderForUser(user, folder);
}).then(function (wantFiles, wantInFolder)){
     //...
})
```

Currently the only way to get it is to do this:
```javascript
getHomeFolderForUser(user).then(function (folder) {
     return [folder, getFilesInFolderForUser(user, folder)];
}).spread(function (gotFolder, gotFiles)){
     //...
})
```

But I think this is ugly and want a prettier way. Introducing .with which creates an array from whatever you resolve with and appends all values you include with. And the sleeker looking thenWith which is really just a wrapper for spread. 

```javascript
getHomeFolderForUser(user).then(function (folder) {
     return getFilesInFolderForUser(user, folder).with(folder);
}).thenWith(function (files, folder)){
     //...
})
```

I chose 'with' because it reads cleanly as 'return promise with value' and I used thenWith to distinguish it from a regular with.

This is a hack on Q and this is how it's implemented:
```javascript
    //as a new method on promise
    Promise.prototype.with = function (value) {
        if (this.withs) {
            this.withs.push(value);
        } else {
            this.withs = [value];
        }
        return this;
    };

    Promise.prototype.thenWith = Promise.prototype.spread;
```
```javascript
//and inside of deferred.resolve()
   if (deferred.promise.withs) {
        become(Q([value].concat(deferred.promise.withs)));
   } else {
        become(Q(value));
   }
```
