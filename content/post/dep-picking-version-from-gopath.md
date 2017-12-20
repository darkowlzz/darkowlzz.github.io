---
title: "dep - Picking Version From GOPATH"
date: 2017-12-20T23:36:25+05:30
draft: false
tags: ["dep", "golang", "go"]
---

This post is about a feature in [golang/dep](https://github.com/golang/dep)
that could be used by developers who are not using vendor directory to vendor
their dependencies, but depend on the version of the dependencies checked out
in their GOPATH.

`dep` provides a subcommand, `dep init`, for initializing a go project to use
dep. It creates two files, a manifest file, called Gopkg.toml, and a lock file,
called Gopkg.lock. These files contain details about all the dependencies
analyzed by dep while initializing.

By default, `init` would reach out to the network and check for the latest
available version of the dependencies of the project and set some default
constraints for those dependencies in the manifest. But this behavior might not
work for everyone. For those who do not use vendor directory to keep their
dependencies or any other vendoring tools, and depend on the checked out version
of the projects in their GOPATH, `init` provides a flag to make thing easy.

`dep init -gopath` can be used to tell dep to initialize the project, but check
GOPATH to get the version of the projects. This would still download the whole
VCS source of the projects and store in dep's cache, but it would set the
constraints to what was in the projects in GOPATH. It doesn't reuse the source
of the projects in GOPATH, only hints from the VCS to figure out the version of
the projects. And when it can't find the project in GOPATH, it would fallback
to the default behavior and look for the latest available version of the project.
