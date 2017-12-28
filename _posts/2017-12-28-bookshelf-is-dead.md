---
layout: post
title: "Bookshelf.js is dead"
date: 2017-10-08
tags:
- javascript
---
For the past three years, I've been a huge advocate of [Bookshelf.js](http://bookshelfjs.org/) to my friends and the companies I have worked at. I've built small libraries that depend on it, and I've read most of the source code and understand the internals. I have actively followed open issues and tried to help developers with their problems whenever I had time outside of work.

However, over the past few months I have distanced myself from the project. I dropped my maintainer status (my contributions were just documentation work and code review, no actual features) and left the Bookshelf organization.

# So what?
I was a huge fan of Bookshelf for a few reasons:
1. It was built on the [Knex](http://knexjs.org/) query builder. Knex is very feature-complete and has great maintainer support & documentation (it was also made by the creator of Bookshelf). Bookshelf makes it reasonable to write custom queries using the Knex API.
2. Bookshelf doesn't force you to [create a Mongo-like schema](http://docs.sequelizejs.com/manual/tutorial/models-definition.html). Instead, it delegates your types to your database and migrations directly. This avoids you from having to update your schemas in two places.
3. The API is focused on the common operations for your database models.
4. It had decent documentation for most of its methods.

However, there are a couple things that are horribly wrong with the API (and will most likely never be fixed):
1. The relations API is poorly documented and hard to understand. [Method naming is very easy to mix up](http://bookshelfjs.org/index.html#associations), and as your data model gets more complicated it gets harder to manage this in your models. Most of my time I spent answering my coworkers' questions about Bookshelf revolve around this issue. [Here's proof it is not isolated to my team](https://github.com/bookshelf/bookshelf/issues?utf8=%E2%9C%93&q=is%3Aissue+is%3Aopen+relation).
2. Extending Backbone brought along the [Collections API](http://bookshelfjs.org/#section-Collection), which is confusing and unnecessary. Instead of using arrays of models, Bookshelf uses a Backbone-like `Collection` class to store a list of `Model`s. This leads to the addition of [unnecessary access methods](http://bookshelfjs.org/#Collection-instance-at) and a [weird API with Lodash methods](http://bookshelfjs.org/#Collection-subsection-lodash-methods).

# What prompted me to leave?
[This issue](https://github.com/bookshelf/bookshelf/issues/1600) was the straw that broke my back.

In June 2017, [Tim Griesser](https://github.com/tgriesser/) moved the Bookshelf project from his personal account to an organization and handed admin rights over to a small group of (mostly) non-active maintainers. A few of the maintainers I have worked with have backed away from the project due to other responsibilities, and the newer maintainers have not roadmapped possible features. From the issue above, there are a lot of :raised_hand:s but no lead maintainer who can review PRs and maintain the project.

In addition, issues that are well-documented have been ignored for a long time. Take [this issue from a maintainer I've worked with](https://github.com/bookshelf/bookshelf/issues/803). This issue has been open for almost three years. These problems with Bookshelf's API have been unaddressed, and this was when Bookshelf was much more actively maintained. With even less maintainers and no lead, it is doubtful that these issues will ever be addressed.

On top of this, [all releases](https://github.com/bookshelf/bookshelf/releases) in the past year were small bug fixes. No major features were shipped, and no issues in the API were addressed.

# OK, so Bookshelf is dead. What replaces it?
I've been following a lot of different ORM projects in JavaScript ([including ones from former maintainers](https://github.com/rhys-vdw/atlas)), and the best ORM I have come across so far is [Vincit's](https://www.vincit.fi/en/) [objection.js](http://vincit.github.io/objection.js/).

There are a few things about this library that really won me over:
1. It's still built on top of [Knex](http://knexjs.org/), just like Bookshelf.
2. No more `Collection` API. Objection uses raw JavaScript arrays, and only has an API around [Models](http://vincit.github.io/objection.js/#model270). This is the correct approach to building an ORM API. Most database operations happen at a model-level, and batch operations are still accepted through the [insert method](http://vincit.github.io/objection.js/#insert).
3. [A fantastic Relation API](http://vincit.github.io/objection.js/#relations). Objection has designed an excellent DSL around designing your model relations. While I'm not a huge fan of the "stringified arrays", I understand the decision-making process behind this. This API is truly in a class of its own for JavaScript ORMs.
4. [Excellent, up-to-date documentation](http://vincit.github.io/objection.js/). This is the kind of documentation I expect from a top-tier library. Examples are all ES6 or newer, with a very in-depth API reference and lots of examples. The Vincit team has done an amazing job here, and I strive to write documentation this complete in the libraries I build in the future.

The features above are the stand-outs, but this only scratches the surface. [Graph upserts](http://vincit.github.io/objection.js/#graph-upserts), [simple eager loading](http://vincit.github.io/objection.js/#eager-loading), [paging](http://vincit.github.io/objection.js/#paging), and the list goes on and on. I've also had a chance to meet some of the team behind this library, and they are very smart developers who are excited about evangelizing their library and their company is committed to support its maintenance.

So, for future projects, I will be exclusively using Objection.js. I highly recommend developers who are looking for a rock-solid ORM to [check out the docs](http://vincit.github.io/objection.js) and be sold for yourself on the documentation and simple API.
