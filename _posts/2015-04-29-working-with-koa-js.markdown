---
layout: post
title: Working with Koa.js
date: '2015-04-29 09:03:27'
tags:
- javascript
---

I recently got my feet wet with [Koa.js](http://koajs.com/), a next-generation web framework made by the creators of Express.

<a href="/assets/images/working-with-koa-js/koajs.png"
   data-rjs="/assets/images/working-with-koa-js/koajs.png"
   class="fluidbox-trigger">
  <img src="/assets/images/working-with-koa-js/koajs.png" alt="Koa logo" />
</a>

My project was to design a Koa server for [TechEmpower's Framework Benchmarks](https://github.com/TechEmpower/FrameworkBenchmarks/). I'm going to use this blog post to sound off on some of the things I like about Koa, and some of the things I don't like. The code for the server I contributed to TechEmpower's benchmark can be found [here](blob/master/frameworks/JavaScript/koa/app.js).

**TL;DR**: Koa is a great framework that improves productivity at the expense of making you think differently as a Node developer.

### The Good

#### Extremely Minimal
One of the big promises of Koa is a add-what-you-need model. Therefore, Koa bundles no middleware whatsoever in its core package. This approach is good because it allows you to pick and choose modules that are most important to you. Express has made strides to separate out middleware as well, but Koa has been designed with this mindset, and it plays to its favor.

#### Freedom from Callback Hell
For any Node.js developer, Koa is going to demand you to think differently. This is a really good thing in the long run: Koa completely ditches callbacks, and instead opts to use generators from the upcoming ECMAScript Harmony (ES6) release. Koa even requires a special build flag in Node to run:

        node --harmony app.js

        # Bonus: io.js has necessary ES6 features already, no need for flag
        iojs app.js

Most of understanding Koa, as mentioned in the **tl;dr** above, has to do with understanding how generators work. If you don't know how generators work in JavaScript, [David Walsh wrote a great article explaining ES6's generators](http://davidwalsh.name/es6-generators).

Below I've included an example of how the `yield` keyword allows you to avoid having to write long, nested callbacks in order to get the information you need. In this function, I set a random world's property `randomNumber` to a new random number in the Mongo database (note: the `*` notation denotes an ES6 generator function).

```js
function *worldUpdateQuery() {
  var randomId = getRandomNumber();
  var randomNumber = getRandomNumber();
  var result = yield worlds.update(
    {id: randomId},
    {$set: {randomNumber: randomNumber}}
  );
  return {
    id: randomId,
    randomNumber: randomNumber
  }
}
```

In this case, `worlds.update()` is not the best example of the use of a callback, as it will only return a count of the records modified. However, take a look at how I write my find query for world models, you normally require a callback function in order to retrieve the data from a find.

```js
function *worldQuery() {
  return yield worlds.findOne({id: getRandomNumber()}, '-_id');
}
```

With `yield`, it executes and returns this asynchronous callback so that the data you need is set properly.

This approach minimizes the amount of nesting you normally need for callbacks, and makes your code significantly more readable. This comes at the cost of approaching Node.js development differently. However, I believe the cost is worth it, and generator functions are the future of JavaScript server frameworks.

#### Contexts
Koa takes an interesting approach to contexts. Koa encapsulates Node's request and response objects into a single object. There's a ton of helper methods for building out great APIs, which is a huge plus (API's rock, no joke). Ultimately, this approach simplifies requests and responses in general.

First, let's take a look at a Koa context example from the Koa.js docs. You can read up more on the APIs for this Context object [here](http://koajs.com/#context).

```js
app.use(function *(){
  this; // is the Context
  this.request; // is a koa Request
  this.response; // is a koa Response
});
```

In TechEmpower's Framework Benchmarks, there are requirements that each framework has to meet in order to be included. 6 different routes must be implemented. Some examples are JSON serialization, plaintext serving, and database queries and updates. Let's take a look at the route handler I wrote for JSON serialization.

```js
function *jsonHandler() {
  this.set('Server', 'Koa');
  this.body = {
    message: "Hello, world!"
  }
}
```

Koa's APIs are written so that all I have to do is set `this.body` to whatever I want it to return. Since the `request` and `response` objects are all encapsulated together, it is easy to add data to the response. Normally, web frameworks have API calls that allow you to respond (`reply` for Hapi and `res.send` for Express, to name a few). Koa simply yields the response object with `this.body` (in this example) set to a JSON object. Since Koa functions are designed to have a unified model for middleware, access to these request and response objects are simple and intuitive.

#### Cascading
I'm only going to talk about this briefly, since I was unable to try this out in my own work. Cascading in Koa's middleware functions is awesome. Koa can yield downstream to other functions until one function returns. Control flows back upstream when this is completed. This is difficult to do with callbacks, and Koa shows how cascading can be accomplished more efficiently with ES6 generators.

[Here's a link to an example cascade from Koa.js' website](http://koajs.com/#cascading). Pay close attention to the `yield next` and how the control flow changes how middleware is normally written in classic Node server frameworks.

### The Bad

#### Shaky Module Support
Documentation for most modules you need are not written to help you with Koa. They are written to suitably assist your navigation through callback hell. While Koa does its best to mitigate the issues of this new approach, you are going to have to spend extra time digging to set up your modules properly. While all modules are technically supported out of the box, you may have to defer to wrappers to make things easier.

An example of this is that I used Monk for my MongoDB driver for my Koa server. Without writing custom generator functions, Monk would not work properly for me. This led to frustration and extra code, which to me seemed that Koa was faltering on its promises. However, after some time searching, I found TJ Holowaychuk's [co-monk](https://github.com/tj/co-monk) wrapper that allowed me to easily use Monk the Koa way.

However, things aren't always that easy. I was trying a different MongoDB driver earlier on in my project, and I ended up having to write stuff like this:

```js
var result = yield function(callback) {
    this.mongo.collection('world').update(
      {id: randomId},
      {randomNumber: randomNumber},
      callback
    );
}
```

When you compare this to my query functions I wrote in the examples above, wrapping this manually is quite a pain. While this does still minimize the amount of code you have to write, having to do this all the time can quickly become a chore.

#### Not the Best for Newbies
Working in Koa was incredibly rewarding for two reasons: it made me a better web developer, and it demanded that I have a better understanding of how web frameworks function. However, for a new programmer approaching Koa as their first web framework, it is very unforgiving.

If you haven't learned generators formally, please go learn them. Even looking at ES6's implementation may not be the best approach. I learned generators in Python for my Programming Languages class at LMU. I encourage you to check out [these notes](http://cs.lmu.edu/~ray/notes/intropython/) from my professor, Dr. Toal. Look under the `Generators` section.

Koa's lack of basic middleware functions was daunting at first. Not understanding that routes were not included with Koa was aggravating. _"Why won't my route respond? Oh, because there's no route middleware functions."_ Basic questions like this demand that you spend a lot of time understanding Koa's methodologies and design decisions. It's not as easy to get a server running in 5 seconds as Node frameworks normally promise.

If this is your first time approaching Node server development, or want to just write a quick and simple server, I highly recommend [Hapi.js](http://hapijs.com). It's a lot more forgiving, the documentation absolutely rocks, and I enjoy developing in it more than Express. I've recommended it to many friends working on APIs and web servers, and they have (for the most part) enjoyed working in it.

### Conclusion
Koa is the only framework I can name that plans to put ES6 to good use. It has made some great design decisions: its lightweight, it avoids nested callbacks, and it is uniquely suited for modern API development. Designing this for TechEmpower's framework benchmarks taught me a lot about the advantages of generators, especially when it comes to database queries (automatic parallel execution of queries using the `yield` keyword is awesome).

If you're an experienced/semi-experienced Node.js developer, I highly encourage you to try and write a simple API in it. There's a ton to love, and the number of well-documented middleware modules and wrappers is [growing](https://github.com/koajs/koa/wiki). I hope to use Koa on a future project, as it seemed to handle my needs well and minimized the amount of code I needed to write.
