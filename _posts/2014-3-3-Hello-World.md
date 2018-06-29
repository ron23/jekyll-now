---
layout: post
title: Async module initialization
---

I want to create a wrapper around AWS-SDK - let's take dynamoDB for example.
My wrapper will have bunch of functions that uses aws-sdk to talk to dynamo. 
All the functions within that wrapper will modify a single table. The tricky part is: That table name is not static and can be different based on the execution. As a matter of fact we use AWS Parameter Store for it which means those config will be available asynchronously.

Ideally if the config we're static I would just do womething like:
```
const putValue = value => dynamo.put({value, table:config.table}));
```

However, config are not available, so I can wrap it with a configGetter:
```
const putValue = value => getConfig().then(config => dynamo.put({value, table:config.table})));
```

That's nice, but I don't want to do it in 20 different functions.

So, I thought of a few solutions:

1. 
