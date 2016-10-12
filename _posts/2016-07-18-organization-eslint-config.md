---
layout: post
title:  "Set up an ESLint config for your organization"
date:   2016-07-18
tags:
- javascript
- eslint
- configuration
---
At VideoAmp, we work with Javascript **A LOT**. To maintain code uniformity across multiple projects and avoid common Javascript errors while working, ESLint is an invaluable tool. It has personally made me a better developer by providing helpful static analysis. The tooling and support built around the open-source checker is some of the best in the JS community.

The biggest pain point of working with ESLint is that while it encourages uniformity, some organizations are unaware of the tools available to make their own style guides. While a project-level configuration may work for a small organization with one code repository, it scales poorly. As your company starts working on many different JS projects, new rules can crop up leading to different rules for different projects.

While some projects do require specific rules, you want a good starting point for all of your projects. You want a shared ESLint configuration for your organization. Let me show you how to make one.

## `eslint-config-<YOUR_ORGANIZATION>`
Of course, after I go through the song and dance of setting this up for VideoAmp, I stumble upon [this article about shared configs](http://eslint.org/docs/developer-guide/shareable-configs) from ESLint's documentation. Creating a shareable config is as simple as starting with a naming convention. From the docs:

> Shareable configs are simply npm packages that export a configuration object. To start, create a Node.js module like you normally would. Make sure the module name begins with `eslint-config-`, such as `eslint-config-myconfig`.

After setting up your module, you can easily add an `index.js` entry point. Here's an example:

_index.js_
```js
module.exports = {
    rules: {
        quotes: ["error", "double", "avoid-escape"]
    }
};
```

Once you have this set up, it's as simple as adding this to a project's `.eslintrc`:

_.eslintrc_
```js
{
  "extends": "eslint-config-<YOUR_ORGANIZATION>"
}
```

## Building on top of existing styleguides
Some teams are very opinionated on code style, while some are just looking for a place to start. At VideoAmp, we wanted to combine our opinions with style consensus in the JS community. To that end, we ended up extending Airbnb's base styleguide in our custom config.

If you want to extend an existing styleguide in your organization's config, it's relatively straightforward. Simply add the styleguide to your `peerDependencies` in your `package.json`, and extend it in your `index.js` file.

_index.js_
```js
module.exports = {
    extends: ["eslint-config-airbnb-base"],
    rules: {
        quotes: ["error", "double", "avoid-escape"]
    }
};
```

## One Code Style, Multiple Dialects
So you now have a way to carry across your ESLint configuration, perhaps with even some rules extended from other styleguides. However, say there's a scenario with a pre-ES6 codebase and you need specific rule subsets (one for ES5, one for ES6). Since some of our codebases at VideoAmp are in ES5, we still wanted a way to enforce best practices and rules.

ESLint allows you to share multiple configs by adding another file to your organization styleguide and referencing it in your `.eslintrc`.

In our `.eslintrc` it would look something like this:

.eslintrc
```js
{
  "extends": "eslint-config-<YOUR_ORGANIZATION>/<OTHER_CONFIG>"
}
```

In your organization's config package, you could structure your files like so:
- `base.js` - File where you keep rules for both ES5/ES6
- `es5.js` - File where you keep your specific ES5 rules
- `index.js` - Base entry point with default rules

As an example, you can extend your `index.js` file from base like so:

_index.js_
```js
module.exports = {
    "extends": [
        "eslint-config-airbnb-base",
        "./base",
    ].map(require.resolve),
    "rules": {
        quotes: ["error", "double", "avoid-escape"],
    },
};
```

> As a note, [`require.resolve`](https://nodejs.org/api/globals.html#globals_require_resolve) allows you to look up and return the location of a module without loading it.

If you wanted ES5 rules, it's as simple as specifying that in your `.eslintrc`:

_.eslintrc_
```js
{
  "extends": "eslint-config-<YOUR_ORGANIZATION>/es5"
}
```

## One Config to Rule them All
For your organization, this now means that all of your styles and rules are consolidated in one place. This means you only have to change your organization config module in order to update your styles across your many repos, microservices, and projects.

You can check out VideoAmp's own open-source ESLint configuration module here:  [`eslint-config-videoamp`](https://github.com/VideoAmp/eslint-config-videoamp) ([it's on npm too](https://www.npmjs.com/package/eslint-config-videoamp)).
