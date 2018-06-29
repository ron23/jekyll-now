---
layout: post
title: Async module initialization
---

I want to create a wrapper around AWS-SDK - let's take dynamoDB for example.
My wrapper will have bunch of functions that uses aws-sdk to talk to dynamo. 
All the functions within that wrapper will modify a single table. The tricky part is: That table name is not static and can be different based on the execution. As a matter of fact we use AWS Parameter Store for it which means those config will be available asynchronously.

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

that's ok, but now whoever needs that will have to do:
{% highlight js %}
require('dynamoWrapper')(doSomething)
{% endhighlight %}

problem is, that doSomething is actually in the same place as the wrapper, so it's gonna be something like:
