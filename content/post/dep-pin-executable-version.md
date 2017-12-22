---
title: "dep - Pin Executable Version"
date: 2017-12-22T23:06:59+05:30
draft: false
tags: ["dep", "golang", "go"]
---

Along with managing project dependencies, [`dep`](https://github.com/golang/dep)
also provides a way to manage the development tools (linters, generators, etc)
to be used with the project. This can be done by adding the tool's full package
path in the `required` field in the manifest file (Gopkg.toml). Unlike other
fields, `required` considers packages, not project. Hence, if a project wants to
install a tool, say dep, `required` should contain `github.com/golang/dep/cmd/dep`.

```toml
required = ["github.com/golang/dep/cmd/dep"]
```

If a project doesn't contain any import path of a package, `required` field can
be used to tell dep to include that. When `dep ensure` is run with `required`
field in manifest, dep would analyze the packages in the `required` field and
create a list of all the packages they depend on individually. These list of
packages are then written into the lock file under separate packages. For
example, package list of github.com/golang/dep/cmd/dep in Gopkg.lock would look
like this:

```toml
[[projects]]
  name = "github.com/golang/dep"
  packages = [
    ".",
    "cmd/dep",
    "internal/feedback",
    "internal/fs",
    "internal/gps",
    "internal/gps/internal/pb",
    "internal/gps/paths",
    "internal/gps/pkgtree",
    "internal/importers",
    "internal/importers/base",
    "internal/importers/glide",
    "internal/importers/godep",
    "internal/importers/govend",
    "internal/importers/gvt",
    "internal/importers/vndr"
  ]
  revision = "8ddfc8afb2d520d41997ebddd921b52152706c01"
  version = "v0.3.2"
```

This `packages` list is used by dep to understand which packages are required
for a proper reproducible build. This list is also used while pruning to figure
out which packages are required and should not be pruned.

To set a constraint on the packages in `required` field, separate
`[[constraint]]` for the same **project**s can be added in the manifest with the
required version. For example, continuing with the above example of dep
executable, constraint for dep could be:

```toml
[[constraint]]
  name = "github.com/golang/dep"
  version = "=0.3.2"
```

`dep` would combine the `required` and `constraint` fields to determine the
version of the package to download.
