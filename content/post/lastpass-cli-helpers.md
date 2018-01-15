---
title: "Lastpass Cli Helpers"
date: 2018-01-15T22:45:57+05:30
draft: false
tags: ["lastpass"]
---

[lpass](https://github.com/lastpass/lastpass-cli) is a lastpass cli client with
a lot of subcommands and options to do a lot of things which are hard to
remember. Based on my day-to-day usage of it, I created a set of bash functions
to simplify the usage. An up-to-date version of the functions can be found
[here](https://github.com/darkowlzz/dotfiles/blob/master/bashfiles/lpass.bash).

There are four functions:

`lpget()` to get the password of a given site.
```
$ lpget google.com
copied google.com secret to clipboard
```

`lpuser()` to get the username of a given site.
```
$ lpget google.com
```

`lpsearch()` to perform grep on all the sites in an account.
```
$ lpsearch google.com
google.com [id:12345678]
google.com-work [id:987654]
```

`lpsg()` to perform search and get password of a given site.
```
$ lpsg google.com
copied google.com secret to clipboard
```

When there are multiple sites with similar names:
```
$ lpsg google.com 2
copied google.com-work secret to clipboard
```

`lpsg` uses `lpget` and `lpsearch` to search and copy the required secret.
