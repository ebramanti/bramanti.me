---
layout: post
title: "Error handling with ES7's async/await"
date: 2017-10-08
tags:
- javascript
---
One of my co-workers jokingly responded after I said I would _never_ use ES7's async/await over Bluebird's [`Promise.coroutine`](http://bluebirdjs.com/docs/api/promise.coroutine.html) with this:

> Year 2040, Man refuses to acknowledge async/await and insists on using coroutines

It's no secret that I have been stubborn to adopt some new features that have come to JavaScript in ES6/ES7. For example, I do not think the addition of ES6 classes was a good decision. It has removed a need for people to deeply understand prototypal inheritance in JavaScript, and further encourages inheritance over composition. However, I have had a valid reason for holding off on using ES7's async/await (until now).

# What is an async function?
Async/await introduces a solution to a problem previously solved by coroutines. It provides a language-level construct for synchronously executing asynchronous actions, making your JavaScript code's control flow significantly easier to follow.

Below is an example of an async function:
{% highlight js %}
const waitToSayHello = async() => {
    await Promise.delay(2000); // Bluebird utility function
    console.log("Hello!");
};
{% endhighlight %}
The code above is extremely clear: wait for 2 seconds and log. Async/await eliminates the need for timeout callbacks by allowing to specify waiting on asynchronous actions.

# Why have I not been using async functions?
Async functions become more complicated when you have to handle rejected promises. Take the code below, which fetches a Bookshelf User model by email, adds a token on to it, and returns a response from a server:
{% highlight js %}
const fetchUserByEmail = async(email) => {
    const user = await User.forge({ email }).fetch();
    user.set("token", getToken(user));
    return User.save();
};

fetchUserByEmail("edward@bramanti.me")
    .then(user => reply(user));
{% endhighlight %}

Let's say if `edward@bramanti.me` is not found in the database when the user is fetched, the fetch promise rejects with an error called `UserNotFoundError`. We want to respond with a 404 for that particular error type and allow all other errors to throw and be caught by our server. There are two ways to handle an error with an async function:
1. A try/catch block inside the async function.
2. A catch block where the async function is called.

There are [plenty of examples online](https://blog.patricktriest.com/what-is-async-await-why-should-you-care/) about the first option, but in this particular case, I want to focus on #2. Here is what our implementation could look like:
{% highlight js %}
const fetchUserByEmail = async(email) => {
    const user = await User.forge({ email }).fetch();
    user.set("token", getToken(user));
    return User.save();
};

fetchUserByEmail("edward@bramanti.me")
    .then(user => reply(user))
    .catch((e) => {
        if (e instanceof UserNotFoundError) {
            reply().code(404);
        }

        throw e;
    })
{% endhighlight %}
The problem here is that the `catch` block passes all error conditions to one function callback. If there is more than one error type, you need multiple conditional `instanceof` checks, which makes your callback longer and more complex. This may not seem like an issue, since we do handle the `UserNotFoundError`. Nevertheless, we have to re-throw if the error does not match conditional `instanceof` check. If you forget to re-throw, this could cause your server to hang in that block because it has not escaped asynchronous execution.

Bluebird (and in particular, `Promise.coroutine`) diverges from async functions because it supports a feature called [filtered catch](http://bluebirdjs.com/docs/api/catch.html#filtered-catch). This feature allows you to specify one or more error constructors to be handled by a specific callback. This is available in many other languages, such as C# and Java, and is incredibly useful for handling different exception types.

Using `Promise.coroutine` we can make use of this feature in a similar manner to async functions:
{% highlight js %}
const fetchUserByEmail = Promise.coroutine(function*(email) {
    const user = yield User.forge({ email }).fetch();
    user.set("token", getToken(user));
    return User.save();
};

fetchUserByEmail("edward@bramanti.me")
    .then(user => reply(user))
    .catch(UserNotFoundError, () => reply.code(404));
{% endhighlight %}

Filtered catching [is not supported by the ES6's native Promise implementation](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/catch), which is why I have avoided using async/await and have stuck to using `Promise.coroutine`. The problem is clear: async/await returns a promise that does not support the useful, granular filtered catch that I need to cleanly handle the multiple different types of errors I could experience inside these functions.


# My solution to "error handling hell"
To get filtered catch support, I need async functions to return a Bluebird promise. Bluebird has a utility function called [`Promise.method`](http://bluebirdjs.com/docs/api/promise.method.html), which we can wrap our async function with to return a Bluebird promise with filtered catch support:
{% highlight js %}
const fetchUserByEmail = Promise.method(async(email) => {
    const user = await User.forge({ email }).fetch();
    user.set("token", getToken(user));
    return User.save();
});
{% endhighlight %}

Now, our called promise error handling looks exactly like it did with `Promise.coroutine`:

{% highlight js %}
fetchUserByEmail("edward@bramanti.me")
    .then(user => reply(user))
    .catch(UserNotFoundError, reply().code(404));
{% endhighlight %}

This is much cleaner, and now gives async/await functions all the power of Bluebird's `catch`.

# Why is this a big deal?
Particularly with Node, I find myself performing multiple asynchronous actions that return different error types. Filtered catch allows you to clearly express your exception handling with discrete catch blocks, which has an added benefit of making your error handling code more readable. It is the reason why I still use Bluebird over native ES6 promises, and I will most likely continue to do so until it becomes a feature in native promises.

# Further Examples/Reading
I made an example with a custom error type [here on JSFiddle](http://jsfiddle.net/arvyf8r3/), go check it out!

Another way to solve error handling in async function is to use [Dima Grossman's await-to-js](https://github.com/scopsy/await-to-js) package. Instead of using Bluebird promises, he uses a wrapper to return errors like the Go programming language. The wrapper returns a 2-element array with the first element being an error/null and the second element being the resolved value. The syntax becomes much cleaner with ES6 using destructuring statements, and is an interesting take on the issue. You can read more in his announcement/blog post [here](http://blog.grossman.io/how-to-write-async-await-without-try-catch-blocks-in-javascript/).

Thanks for reading!
