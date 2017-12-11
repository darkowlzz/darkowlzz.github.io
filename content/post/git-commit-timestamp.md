---
title: "Git Commit Timestamp"
date: 2017-11-24T16:24:40+05:30
draft: false
tags: ["git", "go-git", "go"]
---

This is related to the previous post, [Git Commit Signature Verification](/post/git-commit-signature-verification).
That post was based on the work of implementing git commit signature
verification in [src-d/go-git](http://github.com/src-d/go-git). This post is a
sequel to that in a way.

The previous post ended with an example of how to create a custom commit and
run verification on it.

{{< highlight go>}}
ts := time.Unix(0000000000, 0)
commit := &Commit{
    Hash:      plumbing.NewHash("8a9cea36fe052711fbc42b86e1f99a4fa0065deb"),
    Author:    Signature{Name: "darkowlzz", Email: "example@darkowlzz.space", When: ts},
    Committer: Signature{Name: "darkowlzz", Email: "example@darkowlzz.space", When: ts},
    Message: `status: simplify template command selection
`,
    ...
}
{{< /highlight >}}

The problem here is that the timestamp being added to the commit has no location
attached to it. Hence, it uses the local timezone of the machine where it runs.
Due to this, the test started failing when run on machines in different
timezones.

The signature that was generated was against a fixed timezone and the above code
would change the commit timezone everytime it runs depending on the location of
the machine.

Git commits are written with timestamps containing the timezone of the machine.
Once written, it's not regenerated or changed by normal reading operations.

```
commit 13df556177b33fcfe1187343bd54837e48ec0b52
Merge: 9d269a6 770a444
Author: -------------------------
Date:   Mon Nov 20 13:28:18 2017 -0500
```

A fix for the above issue would be to add a fixed location to the created
timestamp.

{{< highlight go>}}
loc, _ := time.LoadLocation("Asia/Tokyo")
...
    Author:    Signature{Name: "darkowlzz", Email: "example@darkowlzz.space", When: ts.In(loc)},
    ...
{{< /highlight >}}


That would ensure that the timestamp is always the same in that commit.

The previous post had a note about signature verification failure due to an
invisible newline at the end of commit message, this post highlights another
reason for the verification to fail, timestamp.
