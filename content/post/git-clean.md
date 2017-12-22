---
title: "go-git - Clean"
date: 2017-12-12T17:26:54+05:30
draft: false
tags: ["git", "go-git", "go"]
---

[go-git](https://github.com/src-d/go-git) now supports `git-clean`. Follow the
["go-git"](/tags/go-git/) tag to read other go-git related posts.

Git provides a way to clean the working tree by deleting any untracked files
with the `git clean` command. By default, `git clean` would delete any untracked
files. Untracked (empty) directories are not deleted. To delete them, `-d`
option can be passed.

Since clean can be harmful if not used carefully, git wouldn't just delete the
files. It requires passing `-f` option to force remove or `-i` option to enter
an interactive deletion mode. This is an attempt to avoid deleting important
untracked files by mistake.

In go-git, since it's a library, we don't need to attempt to avoid such mistakes.
So, go-git doesn't support any interactive or force kind of options. go-git
works a little differently right now. By default, with no options passed, it
would delete the untracked files only in the current working directory, or repo
root. To delete all the untracked files, in all the directories, `Dir` option
can be enabled. The deletion happens immediately, no warnings.

Example usage:

{{< highlight go >}}
r, _ := git.PlainOpen("path/to/the/repo")
worktree, _ := r.Worktree()
worktree.Clean(&git.CleanOptions{})
{{< /highlight >}}

`Clean()` is a method of `Worktree`. Worktree keeps tracks of all the files and
their status. When a clean is run, status of all the files in worktree is
checked and any file whose status is `Untracked`, is deleted. Since Worktree
doesn't considers directories at the moment, their status can't be checked.
Hence, go-git can't clean empty directories at the moment. Only untracked files
are supported.

Example of passing `Dir` option:

{{< highlight go >}}
worktree.Clean(&git.CleanOptions{Dir: true})
{{< /highlight >}}

This would clean all the untracked files under all the directories in the repo.

This implementation of git-clean is not complete. It can't ignore the files in
.gitignore, or exclude files given a pattern, etc. I tried to not complicate
things and implemented just a basic cleaning mode at first. Other features can
be added on top of it with more contributions :)
