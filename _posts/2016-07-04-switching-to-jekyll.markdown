---
layout: post
title:  "Switching to Jekyll"
date:   2016-07-04
categories: jekyll blog
---
I have decided to move my blog from Ghost to Jekyll. While I am a huge Node fanboy, and would love to continue supporting Ghost and all of its dependent technologies (Bookshelf), I have found Jekyll to be a better fit for me. I've outlined some of my reasoning below:


1. **Version Control with Github**

Ghost does not really have a concept of a VCS, and I've found that incredibly frustrating when writing blog posts. While I can save my posts, I can't commit piece by piece and keep track of my work. Jekyll is designed to use Github Pages, and it's dead simple: push to your gh-pages branch and done. Managing my blog on Github means one less service to worry about.

2. **Local vs remote blogging**

While Ghost provides an awesome CMS and interface for your posts, I find myself just wanting to jump into my editor and write Markdown. While I can copy/paste piece by piece into Ghost's editor, I find this workflow to be defeating the point of using such a powerful CMS as Ghost.

3. **Free vs Paid**

I've been a loyal customer to [Runkite](runkite.com) for over a year now. While they have always provided dependable service, they still run an old version of Ghost, make it difficult to upload themes, and the service costs $5/mo. While there are ways to [use Ghost for free](http://blog.edouardjamin.fr/host-your-ghost-blog-for-free/), Github Pages requires 0 configuration.

4. **It's true, I like Ruby**

The folks at [Philosophie](philosophie.is) broke down my preconceived notions about Ruby after putting me on a Ruby project for a few months. I love working with it on the web, and being able to use its templating engine has been great. I want to support this community, and Jekyll is an important part of it.

5. **Everything's Static**

Jekyll outputs all files to static HTML/CSS/JS. I really like the idea of a blog that's simple and static, instead of having to hit a server that is essentially serving static assets and that's it.

5. **Jekyll has a Ghost exporter**

It was dead simple to move my blog from Ghost using [jekyll-ghost-importer](https://github.com/eloyesp/jekyll_ghost_importer). One command, and all of your posts and drafts are automatically set up for Jekyll. It was surprisingly simple, with almost everything working right out of the box.

So, for now, goodbye Ghost. I'll continue to advocate for using technologies that use Node (and Bookshelf too), but Jekyll has been such a breeze that I can't imagine recommending another technology for developers to set up their first blog with. If you're curious about my theme, I'm using a modified version of [Kactus](https://github.com/nickbalestra/kactus).

Hope you enjoy the new look!
