---
title: "go-git - Grep"
date: 2017-12-12T18:39:02+05:30
draft: false
tags: ["git", "go-git", "go"]
---

[go-git](https://github.com/src-d/go-git) now supports `git-grep`. Follow the
["go-git"](/tags/go-git/) tag to read other go-git related posts.

- git-grep
- The Amazing Worktree
- Grep Options
- Option Validation
- Grep Result

# git-grep

> git-grep - Print lines matching a pattern
>
> - git-scm.com

git-grep performs pattern matching in the tracked files in the work tree, blobs
registered in the index files, or blobs in given tree objects. The result of
match is sent to stdout, which consists of filename and content of
the line. Some options can be passed to get more attribuets in the result.

Example result:
```
$ git grep "clean"
...
content/post/git-clean.md:with the `git clean` command. By default, `git clean` would delete any untracked
...
```

Internally, git performs a regex match of the provided pattern with all the
tracked file content. The beauty of how this works lies in how the file content
can be queried through the worktree. Instead of checking out to the given commit
or reference and then searching through all the files, worktree is used to
reach all the required files in their correct version.


# The Amazing Worktree

Git worktree maintains a collection of all the files in a repo at a given point.
A commit, a branch, a tag, or any reference have their own worktree which
consists of all the files in the repo at that reference. When, say a branch is 
checked out, worktree of that branch at a given commit is loaded and
that becomes the current worktree.

In go-git, `Worktree` provides a way to get a tree, given a commit hash,
`getTreeFromCommitHash()`. This can be used to obtain a tree and use it to
perform grep on the tree, avoiding actual checkouts. No actual repo state change.
The tree can provide a file iterator to traverse all the tree nodes, which are
the files.

To implement this in go-git, `GrepOptions` and `GrepResult` types were created
to store all the options and result of the operation respectively.


# Grep Options

go-git `GrepOptions` consists of `Pattern`, `PathSpec`, `InvertMatch`,
`CommitHash` and `ReferenceName`. An example of a `GrepOptions` is:

{{< highlight go >}}
opts := GrepOptions{
	Pattern:    regexp.MustCompile("import"),
    CommitHash: plumbing.NewHash("2d55a722f3c3ecc36da919dfd8b6de38352f3507")
	PathSpec:   regexp.MustCompile("go/"),
}
{{< /highlight >}}

This `GrepOptions` tells go-git to match the `Pattern` at the tree of
`CommitHash` and only with the files with path matching the `PathSpec`.

Since it's a library, instead of accepting raw strings as pattern and pathspec,
it accepts pre-compiled regexp objects.


# Option Validation

The `GrepOptions` is validated to ensure that the passed options are not
ambiguous. For example, passing a commit hash and a reference is ambiguous.
Also, if no commit hash or reference is passed, the `HEAD` tree of repo is used.


# Grep Results

Results of git-grep are strings sent to stdout, but that can't be done in case
of go-git. Since it's a library, it returns a type `GrepResult`, consisting of
all the attributes of the result.

`GrepResult` contains `FileName`, `LineNumber`, `Content` and `TreeName`. The
result of a grep in go-git is an slice of `GrepResult`.

Example usage:
{{< highlight go >}}
r, _ := git.PlainOpen("path/to/a/git/repo")
worktree, _ := r.Worktree()
grepResults, err := worktree.Grep(&git.GrepOptions{
    Pattern: regexp.MustCompile("hello world"),
    InvertMatch: true,
})
{{< /highlight >}}

`Grep()` is a method of `Worktee`, similar to `Clean()`. A lot of the operations
are performed on worktree.

Like in case of git-clean, even this is not a complete implementation of
git-grep, for the same reasons, keeping it simple. Right now, it supports passing
a single pattern in `Pattern` and `PathSpec`, it should be able to accept slice
of patterns. It can't perform grep on untracked files. All these and more would
be added on top of this base implementation.

go-git is a really nice project to contribute by implementing well known
concepts of git. And the maintainers are great with their quick reviews and
responses. Thanks to their years of work in the underlying plumbing, things like
`Worktree`, all the git `Objects`, `Storer`, etc simplify implementation of
various operations on top.
