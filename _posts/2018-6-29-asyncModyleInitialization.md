---
layout: post
title: Async module initialization
---

I want to create a wrapper around AWS-SDK - let's take dynamoDB for example.
My wrapper will have bunch of functions that uses aws-sdk to talk to dynamo. 
All the functions within that wrapper will modify a single table. The tricky part is: That table name is not static and can be different based on the execution. As a matter of fact we use AWS Parameter Store for it which means those config will be available asynchronously, at runtime. Problem is, how do you write the code in a smart way and keep it DRY.

Ideally if the config we're static I would just do womething like:

{% highlight js %}
const putValue = value => dynamo.put({value, table:config.table}));
{% endhighlight %}

However, config are not available, so I can call my configGetter function:
{% highlight js %}
const putValue = value => getConfig()
.then(config => dynamo.put({value, table:config.table})));
{% endhighlight %}

That's nice, but I don't want to do it in 20 different functions.

I also don't want to call my configGetter many times (ideally that function will be responsible for cacheing, but who knows).

So, I thought I can do something like an asyncRequire - I got the idea [here](https://stackoverflow.com/questions/20315434/node-js-asynchronous-module-loading):

{% highlight js %}
const configGetter  = require('configGetter');
module.exports = function (callback) {
  configGetter.get('/etc/passwd').then(callback(config))
};
{% endhighlight %}

that's ok and now whoever needs that will have to do:

{% highlight js %}
require('dynamoWrapper')(doSomething)
{% endhighlight %}

couple of problems with this approach, mostly that the functions like `doSomething` will have to be written like:

{% highlight js %}
const doSomething = config => (a,b) => doSomethingWithConfig
{% endhighlight %}

another problem is that If I need to access my wrapper from different functions, and those functions pass different callback, I will have to require the same file in multiple places - I don't even think it's going to work in modules mocking for testing. Example:

{% highlight js %}
const a = () => {require('dynamoWrapper')(doA)};
const b = () => {require('dynamoWrapper')(doB)};
{% endhighlight %}

This is basically removing the problem from one place to another.
ok, so another option is to have a class and init() that class, and have every function in that class access "this":

{% highlight js %}
class Wrapper {
  constructor() {
    this.config = null;
  }


  init() {
    this.config = config;
  }

  doSomething() {
  // can access this.config here
   }
}
{% endhighlight %}

Problem with that approach, is 1 - that you have to remember to init, 2 - you have to create a new instance every time (maybe singelton will work here), 3 - you can only init within a function, cause otherwise you wouldn't be able to mock the getConfig.
