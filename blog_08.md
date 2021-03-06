# Zero to Angular in Seconds - Crypto Edition

Welcome back to our blog series about how to get started quickly with
AngularJS using the PubNub AngularJS library. In a couple of recent
episodes, we covered a tiny but powerful example of how to create
a real-time Angular Chat application in less than
[100 lines of code](http://www.pubnub.com/blog/angularjs-101-from-zero-to-angular-in-seconds/),
as well as how to [extend the example with multiplexing](http://www.pubnub.com/blog/building-multiplexing-into-your-angularjs-application/).

The code for today's example is available here:

* https://github.com/pubnub/angular-js/blob/master/app/crypto.html

In _this_ installment we're looking at Cryptography in PubNub, which
lets your app leverage additional encryption to augment the
transport-level encryption provided by PubNub itself over HTTPS. This is
useful because it makes messages opaque *before they are sent*, and prevents
third parties from snooping even if the messages are intercepted en route
to the recipient (or after the fact). The biggest advantage of crypto in PubNub
is that it allows your app to use secure communications with just one additional
line of config (!). Stay tuned and we'll show you just what line of code that
is. Thanks to the magic of Angular and PubNub, it's super easy to make this
happen with minimal code tweaks.

First off, let's take a look at the situations where you might take
advantage of cryptography:

* You want to encrypt messages "end-to-end": at the "first hop", as well as en route to the destination
* You want to encrypt messages so they can be "revealed" later
* You want to rotate cipher keys at periodic intervals so that clients are forced to refresh the key
* You just feel like learning more about encryption and using one of PubNub's advanced features!

So, now that you know you want to try out crypto, how do you
do it? It's pretty easy with the PubNub Angular application.

# Step 1: Get Your Includes On

Setup of the PubNub Angular library is exactly the same as with the original Zero-to-Angular example:

```
<!doctype html>
<html>
<head>
<script src="https://cdn.pubnub.com/pubnub.min.js"></script>
<script src="https://cdn.pubnub.com/pubnub-crypto.min.js"></script>
<script src="//ajax.googleapis.com/ajax/libs/angularjs/1.0.8/angular.min.js"></script>
<script src="//code.jquery.com/jquery-1.10.1.min.js"></script>
<script src="http://pubnub.github.io/angular-js/scripts/pubnub-angular.js"></script>
<link rel="stylesheet" href="//netdna.bootstrapcdn.com/bootstrap/3.0.2/css/bootstrap.min.css">
</head>
<body>
```

What does all this stuff do?

* pubnub.min.js : the main PubNub communication library for JavaScript
* pubnub-crypto.min.js : encryption features in case you need to enable them someday
* angular.min.js : that AngularJS goodness we all know and love
* jquery-1.10.1.min.js : bring in some JQuery
* pubnub-angular.js : bring in the official PubNub SDK for AngularJS
* bootstrap.min.css : bring in the bootstrap styles

Once these are all set, you’re good to start coding!

# Step 2: Set Up Your HTML Layout and Dynamic Content

Setup of the content DIV is the same as with the original Zero-to-Angular example:

```
<div class="container" ng-app="PubNubAngularApp" ng-controller="EncryptedChatCtrl">
```

AngularJS needs to be able to find your app. To make that happen,
we add an 'ng-app' attribute to the div element we want to
Angular-ize. In addition, we need to specify an AngularJS controller
function that takes care of binding all the logic we need. If you
look in the script tag at the end of the page, you'll see where we
set up the EncryptedChatCtrl function.

```html
<h4>Online Users</h4>
<ul>
  <li ng-repeat="user in users">{{user}}</li>
</ul>
```

Here, create a dynamic list of users simply by using a ```ul```
element and an ```li``` element that's set up to iterate over all
of the items in $scope.users. For the purposes of this demo, each
user object is a simple string.

```html
<h4>Chat History {{messages.length}}</h4>
```

In this section, we're just displaying a dynamic header that includes
the length of the messages array (from $scope.messages).

```html
<form ng-submit='publish()'>
  <input type="text" ng-model='newMessage' />
  <input type="submit" value="Send" />
</form>
```

All right! This is the first interactive feature - a simple text box
that binds its content to ```$scope.newMessage```, and a submit button
for the form. The form submit function is bound to the ```$scope.publish```
function. What does it do? We'll find out soon!

```html
<div class="well">
<ul>
   <li ng-repeat="message in messages">{{message}}</li>
</ul>
</div>
```

As you probably already guessed, we're using the Angular 'ng-repeat'
function to iterate over the ```$scope.messages``` array to display the
list of messages in a UL (unordered list) element.

WAIT A MINUTE! I didn't see anything about encryption in the HTML.
That's because the PubNub library takes care of it all under the
hood - pretty sweet huh?

So right now you may be asking, how does it all work? Let's
check out the JavaScript!

# Step 3: JavaScript – Where the Magic Happens

Just like the previous blog entry, we'll wrap up by taking a stroll through
the JavaScript to see what's happening. You'll recognize a bunch from last time:

```
angular.module('PubNubAngularApp', ["pubnub.angular.service"])
.controller('ChatCtrl', function($rootScope, $scope, $location, PubNub) {
  // make up a user id (you probably already have this)
  $scope.userId   = "User " + Math.round(Math.random() * 1000);
  // make up a channel name
  $scope.channel  = 'The Angular ENCRYPTED Channel';
  // pre-populate any existing messages (just an AngularJS scope object)
  $scope.messages = ['Welcome to ' + $scope.channel];
```

Just like last time, we:

* Declare an Angular module that matches our ng-app declaration
* Declare a Controller that matches our ng-controller declaration
* Set up the user id as a random string (your app probably has its own logic for this)
* Set up a starting message

That's pretty cool. Let's see what we have next.

Whoa! Here's the good stuff!

```
if (!$rootScope.initialized) {
  // Initialize the PubNub service
  PubNub.init({
    subscribe_key: 'demo',
    publish_key: 'demo',
    cipher_key: 'changeme-noireallymeanit',   /* adds a cipher key for secure message encryption (CHANGE THIS!) */
    uuid:$scope.userId
  });
  $rootScope.initialized = true;
}
```

This is pretty much the same as any other Angular PubNub application.

The one and only *difference* we care about is:

* We add a *Cipher Key* argument to the options in the PubNub.init() function call

You might be asking now - "Are you serious - that's it?" Yes,
all you have to do is provide a secret key, and the PubNub client
library takes care of the rest. PubNub never knows about this key,
all of the encryption takes place in the browser so you know there's
nobody snooping on your data in between.

Even better, this encryption feature is available on
[dozens of platforms](http://www.pubnub.com/developers/),
so your JavaScript app can communicate securely with PHP,
Java, Ruby, .NET, you name it!

You can even change the cipher key later - just make sure there is
a way for clients to discover the new key and reload the application
in the browser with the new key.

```javascript
  // Subscribe to the Channel
  PubNub.ngSubscribe({ channel: $scope.channel });

  // Create a publish() function in the scope
  $scope.publish = function() {
    PubNub.ngPublish({
      channel: $scope.channel,
      message: "[" + $scope.userId + "] " + $scope.newMessage
    });
    $scope.newMessage = '';
  };

  // Register for message events
  $rootScope.$on(PubNub.ngMsgEv($scope.channel), function(ngEvent, payload) {
    $scope.$apply(function() {
      $scope.messages.push(payload.message);
    });
  });
```

The important things about the code above are:

* We use PubNub.ngSubscribe to register for event callbacks (note: only call it once per channel or you'll get duplicate message events!)
* We create a 'publish()' function in the scope to send messages to the channel using PubNub when the user clicks "send"
* We register for message events using the Angular native $rootScope.$on function
* The channel message events are named using a PubNub-Angular-specific string that we obtain using PubNub.ngMsgEv (which is shorthand for "Angular channel message event name")


```javascript
// Register for presence events (optional)
$rootScope.$on(PubNub.ngPrsEv($scope.channel), function(ngEvent, payload) {
  $scope.$apply(function() {
    $scope.users = PubNub.ngListPresence($scope.channel);
  });
});

// Pre-Populate the user list (optional)
PubNub.ngHereNow({
  channel: $scope.channel
});
```

Here, we're just registering for presence events so we know when users
join and leave the channel. Once that's set up, we can use the "ngHereNow"
function to get the user list when the controller loads. Pretty nifty!

```
});
</script>
</body>
</html>
```

... And we're done! Hopefully this helped you get started with PubNub
message encryption and AngularJS without much trouble. Please keep
in touch, and give us a yell if you run into any issues!



# P.S. Special Footnote for aspiring AngularJS Experts

If you look closely at the example code in the ```crypto.html``` source listing,
you'll actually notice that we have 2 HTML sections: one using encryption and one
not. We did this so that the demo would be more interesting, but it also helped
us pick up on an advanced AngularJS feature along the way that we'd like to share
with you.

So, if you ever have the question: "How can I have more than one Angular ng-app
active at the same time with different controllers in my Angular application?"

Now you'll know! It's like this:

* In the HTML, create 2 elements with different ng-app and different ng-controller tags
* In the JavaScript, add an angular.bootstrap call to initialize the second controller! (see below)

```
// we must call angular "bootstrap" since we're running two separate
// angular apps for normal/encrypted views
angular.bootstrap($('#unencrypted'),['PubNubAngularUnencryptedApp']);
```

This is because AngularJS's initialization is optimized for the single application
case, and just needs a little help to notice any additional ones.
