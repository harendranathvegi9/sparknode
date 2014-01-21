#About

This package was built with the purpose of allowing cross-platform communication from node.js to a [sparkcore](http://docs.spark.io/#/api).

Firmware functions and variables are added automatically based on the spark API's HATEOS responses.

###[Example](#example)

#Constructors
1. [Core](#core)
2. [Collection](#collection)

##Common
###Events
The main event you care about is connect.  This is fired when the cloud has returned and all variables and functions should be registered on your instance. This is true for both Core and Collection.

You can also choose to handle an 'error' event, which will be thrown upon any error that does not happen in the context of a callback.

##Core
Create a new instance of a spark core. First parameter is authtoken, second parameter is deviceId.

An object can also be used as the first parameter, as follows:

```js
{
  authtoken: <Your Auth_Token>,
  id: <Your device id>
}
```

###Functions
Each function accepts a string as the parameter, and a callback to be called upon return.

The signature of the callback should be `function(err, data)`.

```javascript
core.brew('coffee', function(err, data) {
  //Do something
});
```

###Variables
Each variable (exposed as functions) accepts a callback as its first parameter, with the same signature as above (err, data).

```javascript
core.variable1(function(err, data) {
  console.log(data);
});
```

Variables also have a property, autoupdate, that can be set with a timeout in milliseconds between each call, or true to use the default 15 second interval or false to cancel autoupdates. Setting this variable will also start the update cycle.

When being used with autoupdate, the variable will fire an 'update' event each time a response is received from the server.

```javascript
//Start autoupdate
core.variable1.autoupdate = 2000;

//Do something on update
core.variable1.on('update', function(value) {
  console.log(value);
});

//Stop update with any falsy value.
core.variable1.autoupdate = false;

```

The last known value can be retreived as a property (value) of the function.

```javascript
console.log(core.variable1.value);
```

##Collection
Even better, get all your spark cores at once, complete with everything they can do.

Once loaded, the collection instance contains all your spark cores by name.

```javascript
collection.core1.doFunction('foo', function(err, data) {
  //Do something
});
```

The default behavior is to cache the output of the cloud api for the initial call in a JSON object at your project root.  If you'd like to override this behavior, you can pass an options object (optional, of course) to the constructor.

```javascript
var collection = new Collection(myAuthToken, { skipCache: true })
```
or
```javascript
var collection = new Collection(myAuthToken, { cacheFile: 'lib/cacheFile.json' } )
```

##Example

```javascript
var sp = require('sparknode');
var collection = new sp.Collection(myAuthToken);
collection.on('connect', function() {
  //Turn on an led
  collection.core1.digitalwrite('d7,HIGH');

  //Brew some coffee, then email me.
  collection.core2.brew('coffee', function(err, timeUntilFinished) {
    if(err) {
      throw err;
    }

    setTimeout(function() {
      //General awesomeness goes here.
      emailMe();
      sendSocketIoMessage();
      addCreamer();
    }, timeUntilFinished);
  })

  //Get a variable
  collection.core2.remainingCoffeeTime(function(err, value) {
    //Do something with value
  })
```

And an example of a single core.

```javascript
var randomCore = new sp.Core(myAuthToken, deviceId);

randomCore.on('connect', function() {
  randomCore.turnOnLights();
});
```

This library should also work cross platform as it doesn't rely on curl behind the scenes.  I'm hoping it also makes it much easier for me to wire custom functions to a webapp.

I'm also tracking some of the data that comes back from the spark cloud on the core objects themselves, such as `online`, though I'm not sure how useful that will end up being.

##Future

Future:

I have several ideas I'd like to implement such a CLI for quick access.
```bash
sparknode core2 brew "coffee"
```
An API for the server sent events will also be a high priority as soon as that cloud API comes out.

I'm also thinking about writing a custom firmware that lets you add many more than 4 functions, directly from the CLI or even programmatically, using string parsing on the client side. I don't know about anyone else, but I don't need 64 characters of input very often, so I figured they'd be more useful this way. Check out the issues tracker to add feature requests and see some of the plans I have.
