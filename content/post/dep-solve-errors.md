---
title: "dep - Solve Errors"
date: 2018-02-26T14:51:54+05:30
draft: false
tags: ["dep", "golang", "go"]
---

[`dep`](https://github.com/golang/dep) seems to work great for those who are
able to get it to work with their projects. But often, there are issues that
a new dep user faces while trying out dep for the first time. The issues when
dep fails to solve the dependency graph. When this happens, dep needs some help
from the user to solve the issue.

A lot of people find it hard to get started with dep when the very first
command, `dep init` fails and they don't know how to go ahead with that. `init`
scans the project, collect all the project information and perform a solve to
obtain a solution of the dependency graph. Once a solution of obtained, it
writes out a manifest file(Gopkg.toml) and a lock file(Gopkg.lock). The lock
file contains a complete solution of the dependency graph at a given time. The
manifest contains flexible constraints based on the solution.

An init solve failure error looks like this:
```
init failed: unable to solve the dependency graph: Solving failure: No versions of github.com/foo/exampleA met constraints:
    v1.8.1: Could not introduce github.com/foo/exampleA@v1.8.1 due to multiple problematic subpackages:
	Subpackage github.com/foo/exampleA/logger is missing. (Package is required by github.com/bar/exampleB@v23.1.)
    v1.8.0: Could not introduce github.com/foo/exampleA@v1.8.0 due to multiple problematic subpackages:
	Subpackage github.com/foo/exampleA/logger is missing. (Package is required by github.com/bar/exampleB@v23.1.)
    ...
```

Here, dep is trying to tell what's the issue that is confusing it and it can't
complete the solve. In a way, it's asking for some help. It could appear to
be scary at the beginning, and the error message could be very cryptic.

A closer look at the errors and the projects could help in better understanding
the issue. This example is taken from a real issue that was reported. So,
`github.com/bar/exampleB` at version v23.1 depends on `github.com/foo/exampleA`
by importing `github.com/foo/exampleA/logger`. But package `logger` seems to be
missing from `foo/exampleA` project at version v1.8.1.

dep has the nature of prefering semver releases and it looks at the latest
semver release of a project and then goes down the releases to find the version
that satisfied the requirement. In this case, v1.8.1 is the latest release of
project `github.com/foo/exampleA`. Since `bar/exampleB` needs something that's
not in `foo/exampleA`'s latest release, means that maybe `bar/exampleB` depends
on some newer, unreleased version of `foo/exampleA` or maybe some old version
that had the required package. dep would check all the versions, including
branches to find a compatible version of `foo/exampleA`. If it can't find any,
it would fail.

Since dep checks against all the versions and branches, a failure means that the
package "logger" doesn't exist in any of the releases or branches. Maybe it was
in some branch at some point in time, but some new commits have removed it. One
way to solve this would be to get a newer version of `github.com/bar/exampleB`
that's compatible with a proper release of `github.com/foo/exampleA`. Lets say,
master branch of `github.com/bar/exampleB` is in development and seems to be
working with the new releases of `foo/exampleA`. To tell dep to use the master
branch of `bar/exampleB`, create a `Gopkg.toml` file at the root of your main
project and add:
```
[[constraint]]
  name = "github.com/bar/exampleB"
  branch = "master"
```

With this, the manifest file is manually created. Running `dep init` would fail
because a manifest already exists. So, run `dep ensure` and dep would read the
manifest and know which version of `bar/exampleB` to use. If everything remains
well, the solve would complete and a lock file would be generated. If not, some
more errors similar to previous solve errors would show up and they could be
dealt with in the same way, by telling dep which version of the project to use.

The above case is an example of how to troubleshoot when there's a solve 
failure. If all the projects maintain proper releases there shouldn't be such
issues. In case of `github.com/bar/exampleB`, a new release of the project that
depends on new version of `github.com/foo/exampleA` would also solve the issue.

Another type of solve error looks like this:
```
✗   github.com/foo/exampleA@v0.0.5 not allowed by constraint master
    master from (root)
```

This means that dep checking if version v0.0.5 of foo/example is compatible with
the main project, but the root project (main project) has defined a constraint
for `foo/example` to use "master" branch. This can again be tuned according to
the needs in the Gopkg.toml file, and help dep in deciding which version to use.


## dep hash-inputs

There's a hidden tool in dep that can help in debugging the issues when dep
behaves in an unexpected way. Let's say the main project is
`github.com/foo/projectA`. And in the project, there's say, package "foo", that
imports package "bar" of the same project. The import path would be
`github.com/foo/projectA/bar`. If for some reason, the project in the development
machine has the name of the project parent directory as "Foo" instead of "foo".
The project path is `github.com/Foo/projectA`. Running dep with this setup would
confuse dep.

The solve error would contain error lines about unable to find
`github.com/Foo/projectA`. Usually, dep would ignore the main project. But in this
case, due to the issue in the directory name, dep would not ignore it and try to
find the project and fail.

This wouldn't be apparent for someone who hasn't faced this issue. I recommend
using `dep hash-inputs` to see what is dep able to see in the project. These 
are the inputs that are used by dep to perform a solve and come up with a
complete dependency graph.

An example hash-inputs result would be:
```
$ dep hash-inputs
-CONSTRAINTS-
-IMPORTS/REQS-
github.com/bar/exampleB
github.com/foo/exampleA/common
github.com/foo/exampleA/common/hexutil
github.com/foo/exampleA/common/math
gopkg.in/redis.v3
-IGNORES-
-OVERRIDES-
-ANALYZER-
dep
1

```

If the main project is seen in the above list, it's clear that something is
wrong with the import paths or the project directory organization.
hash-inputs can only be generated when there's a manifest file. Even an empty
Gopkg.toml file is enough to generate hash-inputs results.

More details about the failures in dep could be found at https://golang.github.io/dep/docs/failure-modes.html.
