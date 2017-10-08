---
layout: post
title: Are you serial, Promise.all?
date: '2016-03-04 19:13:51'
tags:
- javascript
---

About a month ago, I came across an interesting problem at work where a database migration with [Knex](knexjs.org) was not working properly. An example similar to the actual problem is shown below:

{% highlight js %}
const Promise = require('bluebird');

exports.up = function(knex) {
  return Promise.all([
    knex.schema.renameTable('tomatoes', 'potatoes'),
    knex.schema.table('potatoes', (table) => table.string('name'))
  ]);
};
{% endhighlight %}

The first promise renames a table from `tomatoes` to `potatoes`. The following promise adds the string column `name` to the newly renamed table `potatoes`. While there may seem to be nothing wrong here, this code resulted in an error, saying the table `potatoes` did not exist.

It turns out the problem was the Promise collection itself. I had a preconceived notion that the API for [`Promise.all`](http://bluebirdjs.com/docs/api/promise.all.html) was a function that resolved a promise array sequentially. It turns out it [resolves asynchronous tasks concurrently](https://github.com/petkaantonov/bluebird/issues/134#issuecomment-37192469)!

[There was an issue open at one point for a sequential Promise collection method](https://github.com/petkaantonov/bluebird/issues/134), and the author of Bluebird does give a [compelling argument](https://github.com/petkaantonov/bluebird/issues/134#issuecomment-36976462) for why `Promise.series` was never added to Bluebird. Nevertheless, I wanted to solve my current issue without using callbacks. I needed a way to serially resolve an array of promises and return an array of results.

My solution was to extend Bluebird with a `Promise.series` method that will take in an Iterable and return a Promise that resolves similarly to `Promise.all`, but with all of the promises executed serially through using [`Promise.reduce`](http://bluebirdjs.com/docs/api/promise.reduce.html).

Here's the code I wrote for `Promise.series`:

{% highlight js %}
const Promise = require('bluebird');

Promise.series = (promiseArr) => {
  return Promise.reduce(promiseArr, (values, promise) => {
    return promise().then((result) => {
      values.push(result);
      return values;
    });
  }, []);
};
{% endhighlight %}

The above implementation of series uses `Promise.reduce` to return a promise that will iterate through the promise array, and resolve each promise one after the other. This function will return an array of resolved values (the starting value is the empty array, and results are pushed in as the reducer executes).

**There is a caveat**: for this method to work, we must wrap each promise in a function, almost like a factory. Promises must be wrapped because as soon as a promise is created, it begins execution (see `new Promise()` documentation [here](http://bluebirdjs.com/docs/api/new-promise.html)).

Simple example below (thanks to [madeofpalk](https://news.ycombinator.com/user?id=madeofpalk) on Hacker News):

{% highlight js %}
const func1 = () => new Promise((resolve, reject) => {
    console.log('func1 start');
    setTimeout(() => {
        console.log('func1 complete');
        resolve();
    }, 20);
});

const func2 = () => new Promise((resolve, reject) => {
    console.log('func2 start');
    setTimeout(() => {
        console.log('func2 complete');
        resolve();
    }, 10);
});

Promise.series([func1, func2]);
// Resolves to:
// func1 start
// func1 complete
// func2 start
// func2 complete
{% endhighlight %}

This gives us the serial execution we desire, and it allows our example above to work!

So, let's update the initial example with the new series method we have:

{% highlight js %}
const Promise = require('bluebird');

exports.up = function(knex) {
  return Promise.series([
    () => knex.schema.renameTable('tomatoes', 'potatoes'),
    () => knex.schema.table('potatoes', (table) => table.string('name'))
  ]);
};
{% endhighlight %}

Presto! The first promise now resolves before the second promise transitions to pending, which means that the `potatoes` table now exists and we can add columns to it. By extending Bluebird, we now have a function for those rare cases when the performance benefits of parallel execution are derailed by a need for serial execution.

A serial solution for serial people.

[Discuss on Hacker News!](https://news.ycombinator.com/item?id=11227069)

_Edit: I've updated the post with a better example to more clearly demonstrate the problem I was trying to solve._

_Edit 2: As a bunch of people on Hacker News have pointed out, [my initial implementation of `Promise.series`](https://gist.github.com/jadengore/daa0fcf243325ab458a7) does not work because the promises are already executing if passed directly into `Promise.series` without being wrapped in a function that returns a new Promise (essentially, a factory). I have therefore gone ahead and performed a major rewrite of the implementation of `Promise.series` and the examples, with a new section explaining this. Special thanks to [flattersatz](https://news.ycombinator.com/user?id=flattersatz) and [madeofpalk](https://news.ycombinator.com/user?id=madeofpalk)'s comments, and to a [great article by David Atchley](http://www.datchley.name/promise-patterns-anti-patterns/) that helped me along the way to a better solution._
