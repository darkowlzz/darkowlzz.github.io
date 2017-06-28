---
title: "Learning Hugo"
date: 2017-06-28T16:53:59+05:30
draft: false
---

This site is just a playground to learn [hugo](http://gohugo.io/), how it works
and the workflow. Also, since the generated site is just a collection of static
web pages, this site is hosted on github as github pages and gets automatically
deployed at every commit to the `master` branch.

The hugo project for this site is at the
[`source`](https://github.com/darkowlzz/darkowlzz.github.io/tree/source)
branch. Every commit to this branch triggers a travis build, where the new
pages are generated and with some custom deployment shell commands, the
generated pages are committed to `master` branch. Small and simple CI/CD
setup 🦉
