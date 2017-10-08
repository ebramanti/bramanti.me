---
layout: post
title: A Few Weeks with Atom
date: '2015-08-04 08:02:48'
---

This is probably the billionth post about someone's experience with Atom, and the `tl;dr` for all of the posts is this:

> While Atom is not perfect, it is my text editor for the forseeable future.

Don't believe me? [Here](https://adcaes.wordpress.com/2015/04/18/switching-to-atom-from-sublime-text/), [here](https://medium.com/@vikram/switching-to-atom-70f18a5848a2), and [here](http://www.edsko.net/2015/03/07/vim-to-atom/).

Why is this? There's a lot of things I really don't like about Atom. It's not enough for me to stop using it. Let's get into it.

### What Isn't Great

**Performance: It's basically a web browser with a text editor in it**

OK, I know it is way more complex than that. However, when you think about the fact that Atom uses Chromium and Node.js, and it's processes include `Atom Helper`, it's hard to not think it's your second Chrome.

<a href="/assets/images/a-few-weeks-with-atom/Screenshot-2015-08-04-00-06-54.png"
   data-rjs="/assets/images/a-few-weeks-with-atom/Screenshot-2015-08-04-00-06-54.png"
   class="fluidbox-trigger">
  <img src="/assets/images/a-few-weeks-with-atom/Screenshot-2015-08-04-00-06-54.png" alt="Chrome away from Chrome" />
</a>

Performance is really hit or miss from me. Atom has come a long way with `v1.0`, and it is very usable compared to where it was when I first tried it out a year ago. [Electron](http://electron.atom.io/), the cross-platform framework that drives Atom, allows for a lot of great built-in features: Automatic updates, crash reporting, built-in Windows installers, debugging & profiling, and native menus & notifications for each platform.

However, it's web technologies running in Chromium and Node.js. Chrome memory management on the Mac is abysmal. I've come a long way since the 4GB dark ages of my 2011 MacBook Air, but I can see many low-end computers using up precious resources to run Atom. Users expect something much lighter than an IDE when it comes to text editors, and while Atom is no Eclipse it has used more RAM than Sublime Text in pretty much all scenarios, regardless of packages running.

**Built-in Packages: Not as great as Sublime Text**

I'll be honest, this subheading is complete clickbait. The whole point of Atom is to customize to your heart's content to get the experience that you want, and if you can't figure it out, follow some guides. However, I find the built-in packages with Atom still lacking even in `v1.0` on very basic things.

A simple example is file renaming. Sublime Text has a very fast and nimble way to rename your current working file that's built into your Command Palette.

<a href="/assets/images/a-few-weeks-with-atom/Screenshot-2015-08-04-00-16-36.png"
   data-rjs="/assets/images/a-few-weeks-with-atom/Screenshot-2015-08-04-00-16-36.png"
   class="fluidbox-trigger">
  <img src="/assets/images/a-few-weeks-with-atom/Screenshot-2015-08-04-00-16-36.png" />
</a>

While Atom has this feature as well, it must be installed with an external package ([tree-view](https://github.com/atom/tree-view)) and it has [many issues opened directly related to just the rename function](https://github.com/atom/tree-view/issues?utf8=%E2%9C%93&q=is%3Aissue+is%3Aopen+rename).

<a href="/assets/images/a-few-weeks-with-atom/Screenshot-2015-08-04-00-19-34.png"
   data-rjs="/assets/images/a-few-weeks-with-atom/Screenshot-2015-08-04-00-19-34.png"
   class="fluidbox-trigger">
  <img src="/assets/images/a-few-weeks-with-atom/Screenshot-2015-08-04-00-19-34.png" />
</a>

For a pretty common task in a working project, I'd expect this to be integrated as a pre-installed function, but it is not.

This may seem like a nitpick, but I've had other experiences like this while attempting to set up Atom. I do not have the desire like some to spend a week customizing my editor to get basic features I'm looking for. Atom has a lot of the ones I love, but it's missing some of the critical ones that really up my workflow. It's taken a greater deal of time to set up Atom than it has with Sublime.

### Why I'm Staying

**Updates: C'mon Sublime...**

Sublime Text 3 has been in beta since January 2013. It's August 2015. Since January 2013, we have had 15 updates to Sublime Text. These updates are performed mostly by one developer.

The last update performed to Sublime Text came in March 26, 2015. That's a little over 4 months ago. While this gives testament to the stability of Sublime Text 3 (while still being in beta), it's time to admit it's been in beta for over two years. Updates have come fewer and farther in between. Package support is still there, but beginning to dwindle. Which leads to my next point...

**Open Software Makes all the Difference: The Package Community**

I'm about to transform from hating on Atom's limited built-in packages to applauding it's absolutely _huge_ developer community. [So many people have contributed to Atom, it's frankly insane](https://github.com/atom/atom/graphs/contributors). Any package I've found for Atom is at least usable, if not as perfect as my Sublime workflow. Performance is getting better, since Atom is getting updated constantly. Atom bet on packages using web technologies, and is reaping the rewards. Packages are coming out left and right for everything you can possibly imagine. Atom also uses `.tmLanguage` for syntax highlighting: all you have to do is `CSON` and go. Ultimately, the support from fellow open-sourcers (sorcerers?) will win this war.

What Atom has shown is the majority of their packages, once configured properly, are better than they are in Sublime. Why? To the next, and most important reason I'm staying.

**The First text editor I've used with a better GUI than Sublime Text**

Seriously, look at this magnificence...

<a href="/assets/images/a-few-weeks-with-atom/Screenshot-2015-08-04-00-40-08.png"
   data-rjs="/assets/images/a-few-weeks-with-atom/Screenshot-2015-08-04-00-40-08.png"
   class="fluidbox-trigger">
  <img src="/assets/images/a-few-weeks-with-atom/Screenshot-2015-08-04-00-40-08.png" alt="Cash." />
</a>

Stock theme (One Dark), and I've only changed a few things. I really have not even thought of switching to another theme. This one just looks so dang good, it's dark like Sublime, and even Monokai's classic allure has not beaten me out for how much I respect the design of Atom.

The minimap focuses more on blocks than the actual words, and it looks plain awesome. Tetris, anyone?
<a href="/assets/images/a-few-weeks-with-atom/Screenshot-2015-08-04-00-42-39.png"
   data-rjs="/assets/images/a-few-weeks-with-atom/Screenshot-2015-08-04-00-42-39.png"
   class="fluidbox-trigger">
  <img src="/assets/images/a-few-weeks-with-atom/Screenshot-2015-08-04-00-42-39.png" alt="Tetris" />
</a>

The package [file-icons](https://github.com/DanBrooker/file-icons) is quite awesome too: it shows a visual display of file types so it is easier to see what you are looking at in a project. It's an absolute must-have.

Finally, the ability to customize your settings with a graphical user interface is a huge plus. With Sublime, you would have to look at defaults in a `.json` file and then manually add what you wanted in a User-Specific settings file. Atom has an entirely HTML-powered interface to visualize settings, and `cson` for when you want to customize further. Here's a look at the settings page:
<a href="/assets/images/a-few-weeks-with-atom/Screenshot-2015-08-04-00-48-20.png"
   data-rjs="/assets/images/a-few-weeks-with-atom/Screenshot-2015-08-04-00-48-20.png"
   class="fluidbox-trigger">
  <img src="/assets/images/a-few-weeks-with-atom/Screenshot-2015-08-04-00-48-20.png" />
</a>

### Final Verdict

Sublime Text's days are numbered. For those of you who haven't switched, you have obviously bought in to the ecosystem and have come to depend on the reliability of Sublime. A year ago, I tried Atom and absolutely would not have switched from Sublime Text. Today, with 1.0's release, Atom is now ready for prime-time.

It has an incredible package community, great support (Github, you might have heard of them), and an awesome user interface.

If you don't own a copy of Sublime and are dismissing that window every ten saves... what are you doing? [Download Atom now!](https://atom.io/)

Did you pay $70 for Sublime Text, and feel committed? [Try Atom for two weeks](https://atom.io/). Use it full-time, and [shoot me a tweet](http://twitter.com/home/?status=@Jadengore) letting me know what you think.



I'm genuinely curious because I do miss some things about Sublime. Nevertheless, I think I'm sticking with the [hackable text editor for the 21st century](https://atom.io/).
