---
title: Migrating to Facebook's Yarn package manager
date: 2016-11-01 10:43:00
tags: [nodejs, javascript, yarn]
---

In October, Facebook released the first version of their [Yarn package manager](https://github.com/yarnpkg/yarn) for Javascript.

Yarn is a group effort between Facebook, Google, Exponent and Tilde to solve several issues they have faced with the standard npm client around speed and security. The result is a super-fast drop-in replacement for npm that will dramatically speed up install times.

Whereas npm pulls modules from the source every time you run an install, Yarn keeps a local cache (outside of your project directory) and copies from that. This means it's not only far quicker than npm but also works if the machine has no internet connection (for example a secure build server),

The clean npm install time for the [Whipclip web app](https://www.whipclip.com) is typically around 3 minutes. With yarn, it takes 17 seconds. That's a huge improvement not just for local dev but also for our CI workflow. Our unit test task has dropped from 2 minutes 13 seconds to 50 seconds.

``` bash
$ rm -rf node_modules
$ time npm install
real	3m3.092s

$ rm -rf node_modules
$ time yarn 
real	0m16.805s

```


The beauty of Yarn is that it's a drop-in replacement for npm and uses your existing package.json file. You don't need to change your workflow or tech; simply run `yarn` instead of `npm install` and you're all set!

More info: [Facebook releases Yarn](https://code.facebook.com/posts/1840075619545360)
