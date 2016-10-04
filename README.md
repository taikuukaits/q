This is two small changes to Q that allow for nicer syntax when executing sequential promises with additional dependencies. 

Personally not many of my functions take 1 parameter. I wish my code was pure enough for that but it isn't.

What I want to write are promises which take multiple arguments and evaluate in series. 

For example, let's say I have a file system and for each operation I need to pass the user making the request (to filter by what they are authorized to see). To init my file browser I want to get a users home folder then get a that folders list of files.

This is what I want:
```javascript
homeFolderForUser(user).then(function (folder) {
     return filesInFolder(user, folder);
}).then(function (wantFiles, wantInFolder)){
     //...
})
```

Currently the way to get it is to do this:
```javascript
homeFolderForUser(user).then(function (folder) {
     return [folder, filesInFolder(user, folder)];
}).spread(function (gotFolder, gotFiles)){
     //...
})
```

But I think this is ugly and want a prettier way. Introducing .with which creates an array from whatever you resolve with and appends all values you include with. And the sleeker looking thenWith which is really just a wrapper for spread. 

```javascript
homeFolderForUser(user).then(function (folder) {
     return filesInFolder(user, folder).with(folder);
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
