---
title: "Git Object Alternates"
date: 2017-12-11T16:16:02+05:30
draft: false
tags: ["git", "go-git", "go"]
---

This is another git related post coming out of more contributions to the
[src-d/go-git](https://github.com/src-d/go-git) project. Other related post can
be found by following the "go-git" tag at the bottom of this page.

Git clone offers an option to clone a repo without actually copying the git
objects to a new location, but sharing the objects with the new repo. This can
be done by using the `--shared` or `-s` option. It's for local repos only.

```
$ git clone -s <sourcerepo> <newrepo>
```

This would clone the source repo files into a new repo and create an alternates
files at `.git/objects/info/alternates`, with content as an absolute path to the
source repo's object directory.

For example:
```
/Users/username/rep1//.git/objects
```
Or, in Windows:
```
C:\\Users\\username\\repo1\\.git\\objects
```

This tells git where to look for the objects in a repo. According to
[gitrepository-layout](https://www.kernel.org/pub/software/scm/git/docs/gitrepository-layout.html)
documentation, there could be multiple paths in the file (one per line) and git
would look them up one-by-one. Also, these paths could be relative as well. But
relative to the source object directory, not repo. And that mostly looks
something like this (if the source and shared repo are at the same level):

```
../../../sourcerepo//.git/objects
```

This shared object lookup works in go-git too now for both *nix and Windows.

In go-git, `.git/` is abstracted as an internal type, `DotGit`. Since, it's
part of an internal package, it can't be used by a user of go-git. But is
available to be used in go-git development. `DotGit` has a method `Alternates()`
which reads the alternates file and returns a list of all the resolved `DotGit`s
found in the alternates.

