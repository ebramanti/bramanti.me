---
layout: post
title: ORM vs. Database in Node.js
date: '2015-09-25 19:24:22'
tags:
- javascript
---

I've been working a lot with Node lately. Currently, I'm building an API using Express.js, with PostgreSQL for the database and Bookshelf for the ORM. Overall, it's been a good experience (I've worked with Backbone before, and Bookshelf is based on Backbone's model/collection paradigm). However, I put my foot down today and said enough was enough. Debugging an issue I ran into became impassable. Helpful stack traces? Nope.

As I dug into this issue, I've come to realize something as a Node developer: a lot of ORM/database issues are actually a side effect of using Javascript as a server-side language.

<a href="{% asset_path 'orm-vs-database-nodejs/elephant-bookshelf.jpg' %}"
   data-rjs="{% asset_path 'orm-vs-database-nodejs/elephant-bookshelf.jpg' %}"
   class="fluidbox-trigger">
  <img src="{% asset_path 'orm-vs-database-nodejs/elephant-bookshelf.jpg' %}" />
</a>

### The Problem: Integers

My debugging journey started at an API resource that would take in an `id` and delete the specified resource. Let's call this resource `potato`.

You would be able to access an individual potato from this URI:

> `api/v1/potato/`

Here's what the request looked like:

> `DELETE api/v1/potato/120382342342341`

The test that was failing should `404` if a resource is not found, so I made up some arbitrary id to show that the API would indeed respond with a `404` that the requested resource was not found. However, instead of getting a `404`, the API was giving me a `500`. In my test logs, the error `numutils.c at line 65` was all I had to work with. I dusted off my C hat and pressed on.

My first inkling was to think this was a problem with V8 or one of the native libraries that I'm using in the project. Then, as I looked through the error output I saw `"routine":"pg_atoi"` in the response. `pg`? Clearly a Postgres issue. So I've narrowed the issue down to Bookshelf. After about 15 minutes of looking at the query I use on this route (it was a little more customized than a classic `save` command), I was unable to find anything wrong with it. In addition, all of my tests with well-formed id's were passing.

This isn't the first time I've had an error message returned with a C error from Postgres. This was, however, the first time that I had an error message from Postgres that I was unable to solve by fixing improper syntax with my ORM. The time had finally come to take a dive into Postgres' source code.

##### Postgres Source Code: Magic, and Numbers

`numutils.c at line 75` was all that I had to go off of. [`numutils.c`](http://doxygen.postgresql.org/numutils_8c.html) is a very important file to Postgres' standard library, and in particular `pg_atoi`, the function in question for integer parsing. Here's an excerpt of where my problem was:

{% highlight c %}
case sizeof(int32):
    if (errno == ERANGE
#if defined(HAVE_LONG_INT_64)
    /* won't get ERANGE on these with 64-bit longs... */
        || l < INT_MIN || l > INT_MAX
#endif
        )
        ereport(ERROR,
                (errcode(ERRCODE_NUMERIC_VALUE_OUT_OF_RANGE),
        errmsg("value \"%s\" is out of range for type integer", s))); // line 75
    break;
{% endhighlight %}

Now that I have localized the issue, it is clear that Postgres' `MAX_INT` is less than the value of the `id` that I have for the potato above (`120382342342341`). After [a quick read through Postgres' numeric datatypes](http://www.postgresql.org/docs/9.1/static/datatype-numeric.html), I learned that Postgres' `MAX_INT` corresponds with signed 32-bit numbers (-2147483648 to +2147483647).

This is an interesting Javascript problem. [Javascript's `MAX_INT` is larger than Postgres](http://stackoverflow.com/questions/307179/what-is-javascripts-highest-integer-value-that-a-number-can-go-to-without-losin), which means that numbers like `120382342342341` are fair game in Javascript but not in PostgreSQL. The next logical step? Enforce Postgres' integer requirements at the ORM level. Bookshelf clearly is not enforcing this, so I should just open a PR and add this feature, right?

Bookshelf's site describes the ORM as "designed to work well with PostgreSQL, MySQL, and SQLite3." There are other databases beyond this list which also support Bookshelf's ORM, which means that Bookshelf would have to enforce different rules for different datatypes. **The ultimate problem is this: Javascript is weakly typed, and enforcing types at the ORM level goes against Javascript's design** (love it or hate it). I knew that the only way forward from this issue was a system that would allow for smarter validation of route parameters in my API.

### The Answer: Middleware

I came up with an elegant solution so that you are able to delete a potato, yet enforce PostgreSQL's `MAX_INT` requirement. That way, you can elegantly handle the error of an out-of-bounds integer potato `id`. The answer, as stated in the header above, is middleware.

Before I hit the potato route, we need a way to validate the route parameter according to the rules of what a well-formed `id` should look like. Since `id`s are autoincremented, we know that `id > 0`. We now also know the limit of our `id` parameter: `MAX_INT` in Postgres.

Beyond validating the range of the `id`, we also need to validate the numericality of the `id`. I've been using [validate.js](http://validatejs.org/) to validate my request bodies, and the utility works well. However, route parameter validation is important as well, and validate.js does not provide this functionality. Without this validation users can pass in anything from "foo" to an out-of-bound Postgres integer as a route parameter. This can lead to unexpected behavior and non-descriptive `500` errors from the API.

Here's a solution for the potato example:

{% highlight js %}
var INT_MAX = 2147483647;
var isInvalidId = function(routeParam) {
  var integer = parseInt(routeParam);
  return isNaN(integer) && (integer > INT_MAX || integer < 0);
};

var idParamValidator = function(req, res, next) {
  if (req.params.id) {
    if (isInvalidId(req.params.id)) {
      // error out, 400 Bad Request
    } else {
      next();
    }
  } else { // ignore this, not using id route param
    next();
  }
};
{% endhighlight %}

As you can see, I validate that routes that make use of the `id` route param in Express falls into this middleware validation. We validate that this is an integer using [`parseInt()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/parseInt), and then check if the result of the parse is `NaN`. If not, then we validate that it falls in our range. If it passes, great! On to the API logic. If not, we `400` with some useful error data for the client (id out of range).

### Conclusion

#### Understand your API/ORM/Database data differences
One important thing to remember about this post is that there was no way to plan for this issue because I did not understand the difference between Javascript and Postgres datatypes. **Understanding the entire stack, including the database, avoids issues like this one.** We may leverage ORMs in the day-to-day to save us the technical overhead of worrying about SQL, but we must consider the entire stack (in my case Javascript, Bookshelf and Postgres) and possible friction between its pieces if we are to create well-designed web applications.

#### Validate route parameters in your API
I would say that most Javascript API engineers have great request body validation tools and leverage them well. Tools like [validate.js](http://validatejs.org/) and [joi](https://github.com/hapijs/joi) provide great schemas for validation. Nevertheless, **it is just as important to validate your route parameters as your body parameters in an API.** This safeguards you from issues like the one I have described in this post.

My solution is a very simple middleware for one case, but as APIs get more complex, they could have multiple types of route parameters. [express-validator](https://github.com/ctavan/express-validator) allows you to validate route parameters in Express.js with a nice library of tools. [Joi](http://hapijs.com/tutorials/validation#path-parameters) does this as well, but you have to be using [Hapi.js](http://hapijs.com/) to have this functionality. Ultimately, there are tools out there to do this for you. It is up to you to leverage them to safeguard your API.
