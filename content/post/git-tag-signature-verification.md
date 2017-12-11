---
title: "Git Tag Signature Verification"
date: 2017-11-25T18:51:26+05:30
draft: false
tags: ["git", "go-git", "go"]
---

This post is also a sequel of other posts related to git and signature
verification
([Git Commit Signature Verification](/post/git-commit-signature-verification)
and [Git Commit Timestamp](/post/git-commit-timestamp)). This one is based on
adding tag signature verification in
[src-d/go-git](http://github.com/src-d/go-git).

- Verify by taking everything apart
- Verify using src-d/go-git

Git tags are signed similar to commits with small differences in their storage
format. Like commit, which are stored in `.git/objects/`, tag are stored in
`.git/refs/tags/`.
```
$ ls .git/refs/tags/
v0.1  v0.2  v0.3
```

Since tags are fixed reference to git commits, these file contain hash of the
commit they refer to.

```
$ cat .git/refs/tags/v0.1
064f92fe00e70e6b64cb358a65039daa4b6ae8d2
```

In this case, `064f92fe00e70e6b64cb358a65039daa4b6ae8d2` is the commit hash of
the commit tag `v0.1` refers to.

Also, in this case, `v0.1`, `v0.2` and `v0.3` point to the same commit but only
tag `v0.2` is signed. `v0.1` and `v0.3`, not being signed have the same content.
But `v0.2` contains a different hash. It contains hash of an actual tag object.

```
$ cat .git/refs/tags/v0.2
5ec495852a6ca4f4082cc86b11d56c732b5ece4b

$ cat .git/refs/tags/v0.3
064f92fe00e70e6b64cb358a65039daa4b6ae8d2
```

Reading the commit hash of unsigned tag with `cat-file` shows the commit object
content.
```
$ git cat-file -p 064f92fe00e70e6b64cb358a65039daa4b6ae8d2
tree 2ff896e009f8b165bb36583653e2868c30eb4127
parent d2ae550ad09a39d67ca18afbd300ce83d90f30a1
author Sunny <example@darkowlzz.space> 1511523499 +0000
committer Sunny <example@darkowlzz.space> 1511523499 +0000
gpgsig -----BEGIN PGP SIGNATURE-----

 iQFHBAABCAAxFiEEoRt6IzxHaZkkUslhQyLeMqcmyU4FAloYBLcTHG1lQGRhcmtv
 d2x6ei5zcGFjZQAKCRBDIt4ypybJTlD6CACEaXd+yoFGMJdtan475CfkgiEc90/3
 W2iIKRvU8sBogfDtfZxYb02RKiemxbaBgTKJDoW7Nzn+XitSNtwHwcUZpNv3dm0g
 Dx46KV090Hv8lJbKFwaORJ8FrGhtOpSluN7l/zY9cmVU6ltInWwnj3l/7Pp89hnA
 9ir/nrtwliJEyRG8D1nbs8cg7F2Gg5BJwSAfM5F0pCj2ka99vIyHHMSRl7zrln0n
 eGObrsjN+EG784uvpOSyquejC02tRrcauLkbz5HmcXkzq8F4QFpcOWVwEbbR1dqB
 lLFrbzXa4h5kvexMofJoCFT8zTLpRQXO6lD4OoCkFn+IqPMRLaJ1GFr8
 =LU+k
 -----END PGP SIGNATURE-----

Some commit message
```

And reading the hash of signed tag shows the content of a tag object.
```
$ git cat-file -p 5ec495852a6ca4f4082cc86b11d56c732b5ece4b
object 064f92fe00e70e6b64cb358a65039daa4b6ae8d2
type commit
tag v0.2
tagger Sunny <example@darkowlzz.space> 1511524851 +0000

Some tag message
-----BEGIN PGP SIGNATURE-----

iQFHBAABCAAxFiEEoRt6IzxHaZkkUslhQyLeMqcmyU4FAloYCg8THG1lQGRhcmtv
d2x6ei5zcGFjZQAKCRBDIt4ypybJTs0cCACjQZe2610t3gfbUPbgQiWDL9uvlCeb
sNSeTC6hLAFSvHTMqLr/6RpiLlfQXyATD7TZUH0DUSLsERLheG82OgVxkOTzPCpy
GL6iGKeZ4eZ1KiV+SBPjqizC9ShhGooPUw9oUSVdj4jsaHDdDHtY63Pjl0KvJmms
OVi9SSxjeMbmaC81C8r0ZuOLTXJh/JRKh2BsehdcnK3736BK+16YRD7ugXLpkQ5d
nsCFVbuYYoLMoJL5NmEun0pbUrpY+MI8VPK0f9HV5NeaC4NksC+ke/xYMT+P2lRL
CN+9zcCIU+mXr2fCl1xOQcnQzwOElObDxpDcPcxVn0X+AhmPc+uj0mqD
=l75D
-----END PGP SIGNATURE-----
```

A signed tag object contains `object`, `type`, `tagger`, tag message/comment
and signature. Value of `object` is the same as the commit hash the tag
refers to.

Apart from the attributes of tag object and commit object, signature is at the
bottom in a tag object but in a commit object, signature is above message.


# Verify by taking things apart

Similar to [Git Commit Signature Verification](/post/git-commit-signature-verification),
to perform a manual signature verification by taking things apart, create a
file, say `tag.txt` and copy the tag content except the signature.
```
$ git cat-file -p 5ec495852a6ca4f4082cc86b11d56c732b5ece4b
object 064f92fe00e70e6b64cb358a65039daa4b6ae8d2
type commit
tag v0.2
tagger Sunny <example@darkowlzz.space> 1511524851 +0000

Some tag message
```

Create another file `doc.sig` and copy the signature only.
```
-----BEGIN PGP SIGNATURE-----

iQFHBAABCAAxFiEEoRt6IzxHaZkkUslhQyLeMqcmyU4FAloYCg8THG1lQGRhcmtv
d2x6ei5zcGFjZQAKCRBDIt4ypybJTs0cCACjQZe2610t3gfbUPbgQiWDL9uvlCeb
sNSeTC6hLAFSvHTMqLr/6RpiLlfQXyATD7TZUH0DUSLsERLheG82OgVxkOTzPCpy
GL6iGKeZ4eZ1KiV+SBPjqizC9ShhGooPUw9oUSVdj4jsaHDdDHtY63Pjl0KvJmms
OVi9SSxjeMbmaC81C8r0ZuOLTXJh/JRKh2BsehdcnK3736BK+16YRD7ugXLpkQ5d
nsCFVbuYYoLMoJL5NmEun0pbUrpY+MI8VPK0f9HV5NeaC4NksC+ke/xYMT+P2lRL
CN+9zcCIU+mXr2fCl1xOQcnQzwOElObDxpDcPcxVn0X+AhmPc+uj0mqD
=l75D
-----END PGP SIGNATURE-----
```

To verify that the tag has not been tampered and it's signed by the person who
says to have, run:
```
$ gpg --verify doc.sig tag.txt
```

Ensure that the public key of the signer is in gpg keyring before verifying.


# Verify using src-d/go-git

`gopkg.in/src-d/go-git.v4/plumbing/object`'s Tag type has `Verify` method now,
to perform signature verification of a tag, given an armored keyring.
```
func (c *Tag) Verify(armoredKeyRing string) (*openpgp.Entity, error)
```

An example tag object can be created and verified as:
```golang
ts := time.Unix(1511524851, 0)
loc, _ := time.LoadLocation("Asia/Tokyo")
tag := &Tag{
    Name:   "v0.2",
    Tagger: Signature{Name: "Sunny", Email: "example@darkowlzz.space", When: ts.In(loc)},
    Message: `This is a signed tag
`,
    TargetType: plumbing.CommitObject,
    Target:     plumbing.NewHash("064f92fe00e70e6b64cb358a65039daa4b6ae8d2"),
    PGPSignature: `
-----BEGIN PGP SIGNATURE-----

iQFHBAABCAAxFiEEoRt6IzxHaZkkUslhQyLeMqcmyU4FAloYCg8THG1lQGRhcmtv
d2x6ei5zcGFjZQAKCRBDIt4ypybJTs0cCACjQZe2610t3gfbUPbgQiWDL9uvlCeb
sNSeTC6hLAFSvHTMqLr/6RpiLlfQXyATD7TZUH0DUSLsERLheG82OgVxkOTzPCpy
GL6iGKeZ4eZ1KiV+SBPjqizC9ShhGooPUw9oUSVdj4jsaHDdDHtY63Pjl0KvJmms
OVi9SSxjeMbmaC81C8r0ZuOLTXJh/JRKh2BsehdcnK3736BK+16YRD7ugXLpkQ5d
nsCFVbuYYoLMoJL5NmEun0pbUrpY+MI8VPK0f9HV5NeaC4NksC+ke/xYMT+P2lRL
CN+9zcCIU+mXr2fCl1xOQcnQzwOElObDxpDcPcxVn0X+AhmPc+uj0mqD
=l75D
-----END PGP SIGNATURE-----
`,
}

entity, err := tag.Verify(armoredKeyRing)
...
```

For details about creating `armoredKeyRing`, check the [Git Commit Signature Verification](/post/git-commit-signature-verification) post.

Also, like commit message, a tag message too ends with a newline.
