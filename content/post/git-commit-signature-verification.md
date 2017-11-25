---
title: "Git Commit Signature Verification"
date: 2017-11-23T23:42:00+05:30
draft: false
---

This post is about PGP signature verification of git commits.

* Verify by taking things apart
* Verify using git cli
* Verify using golang's crypto/openpgp package
* Verify using src-d/go-git

Git commits and any kind of data can be signed using GPG (GNU Privacy Guard)
program, available at https://www.gnupg.org/. It implements the OpenPGP standard
for encryption.

# 4 Things

## Data
This is the data/content that is to be communicated or published and requires
signing for verifiable authenticity. This can be public.

## Private Key
This is a secret key which is used to sign data/content. It has an associated
public key. This must never be public.

## Public Key
This is a publicly available key which can be used to verify a signed document.
It has an associated private key. This should be openly distributed and should
be public.

## Signature
This could be a binary blob or ASCII text, generated as a separate file when
detached, or part of document that's signed. This can be public.

The above 4 things are related as:

```
Data + Private Key => Signed Document

Data + Signature + Public Key => Signature Verification
```

The data document can be signed using a private key to generate a signed
document.

The signed document can be verified that it has not be tampered and is created
by the person who says it's from.

These 4 things are involved in PGP signature verification.


# git commit signing

In case of git commits, the commit objects are stored in the filesystem under
`.git/objects/`, which would have subdirectories with commit hash prefix.

```
$ ls .git/objects/
00/   09/   16/   1d/   27/   32/   3c/   46/   53/   5c/   69/   77/
91/   9b/   a1/   ac/   b6/   bd/   c6/   d0/   dc/   e6/   f2/   f9/
01/   0b/   17/   1e/   29/   34/   3e/   47/   54/   5d/   6a/   79/
92/   9c/   a2/   ae/   b7/   be/   c8/   d3/   dd/   e7/   f3/   fb/
```

Inside these directories are the compressed binary commit files. They can be
read using `git cat-file`.

```
$ git cat-file -p 8a9cea36fe052711fbc42b86e1f99a4fa0065deb
tree 6572ba6df4f1fb323c8aaa24ce07bca0648b161e
parent ede5f57ea1280a0065beec96d3e1a3453d010dbd
author darkowlzz <example@darkowlzz.space> 1511197315 +0000
committer darkowlzz <example@darkowlzz.space> 1511197315 +0000
gpgsig -----BEGIN PGP SIGNATURE-----

 iQFHBAABCAAxFiEEoRt6IzxHaZkkUslhQyLeMqcmyU4FAloTCrsTHG1lQGRhcmtv
 d2x6ei5zcGFjZQAKCRBDIt4ypybJTul5CADmVxB4kqlqRZ9fAcSU5LKva3GRXx0+
 leX6vbzoyQztSWYgl7zALh4kB3a3t2C9EnnM6uehlgaORNigyMArCSY1ivWVviCT
 BvldSVi8f8OvnqwbWX0I/5a8KmItthDf5WqZRFjhcRlY1AK5Bo2hUGVRq71euf8F
 rE6wNhDoyBCEpftXuXbq8duD7D6qJ7QiOS4m5+ej1UCssS2WQ60yta7q57odduHY
 +txqTKI8MQUpBgoTqh+V4lOkwQQxLiz7hIQ/ZYLUcnp6fan7/kY/G7YoLt9pOG1Y
 vLzAWdidLH2P+EUOqlNMuVScHYWD1FZB0/L5LJ8no5pTowQd2Z+Nggxl
 =0uC8
 -----END PGP SIGNATURE-----

some commit message
```

`-p` flag is for pretty-printing the content. And that's what a commit file
contains. `tree`, `parent`, `author`, `committer`, `message` and `signature`.
Some of them might be empty depending on the git tree and the type of commit.

An unsigned commit would not have the `gpgsig` signature but other parts would
be the same. 

# Verify by taking things apart

The document content of this commit that is signed when a commit is signed is:
```
tree 6572ba6df4f1fb323c8aaa24ce07bca0648b161e
parent ede5f57ea1280a0065beec96d3e1a3453d010dbd
author darkowlzz <example@darkowlzz.space> 1511197315 +0000
committer darkowlzz <example@darkowlzz.space> 1511197315 +0000

some commit message
```

Say, we put this data in a file `commit.txt`.

The signature is:
```
-----BEGIN PGP SIGNATURE-----

 iQFHBAABCAAxFiEEoRt6IzxHaZkkUslhQyLeMqcmyU4FAloTCrsTHG1lQGRhcmtv
 d2x6ei5zcGFjZQAKCRBDIt4ypybJTul5CADmVxB4kqlqRZ9fAcSU5LKva3GRXx0+
 leX6vbzoyQztSWYgl7zALh4kB3a3t2C9EnnM6uehlgaORNigyMArCSY1ivWVviCT
 BvldSVi8f8OvnqwbWX0I/5a8KmItthDf5WqZRFjhcRlY1AK5Bo2hUGVRq71euf8F
 rE6wNhDoyBCEpftXuXbq8duD7D6qJ7QiOS4m5+ej1UCssS2WQ60yta7q57odduHY
 +txqTKI8MQUpBgoTqh+V4lOkwQQxLiz7hIQ/ZYLUcnp6fan7/kY/G7YoLt9pOG1Y
 vLzAWdidLH2P+EUOqlNMuVScHYWD1FZB0/L5LJ8no5pTowQd2Z+Nggxl
 =0uC8
 -----END PGP SIGNATURE-----
```

The above signature is an example of ASCII-armored signature. The space at the
beginning of the lines are wrapped by most of the pgp implementations, so that
shouldn't be an issue.

Say, we put this signature is another file `doc.sig`.

This way of separating the data and signature is referred to as `Detached 
Signature`.

To verify a document with a given signature, we also need a public key of the
signer. A program like gpg uses a `keyring` to store known public keys. It's 
the user's responsibility to verify and keep the public keys up-to-date.

Using gpg, we can verify `commit.txt` with signature `doc.sig` by running:
```
$ gpg --verify doc.sig commit.txt
```
If the signer is in the gpg keyring, that would be returned as the output. Else,
the verification would fail.

**NOTE**: Even a slight change in the content of `commit.txt` (a whitespace,
newline, etc) would result in a failed verification.

**CATCH**: The commit message at the bottom of the commit file ends with a
newline, which isn't visible. Absence of this newline also results in vailed
verification. An example would be in case of verification with `src-d/go-git`.


# Verify using git cli

`git` cli tool has a subcommand `verify-commit` to perform the signature
verification.

```
$ git verify-commit 8a9cea36fe052711fbc42b86e1f99a4fa0065deb
```

The way this works internally, as per the [`git/gpg-interface.c`](https://github.com/git/git/blob/master/gpg-interface.c),
is by exporting the signature into a temporary file and passing the signature
file to `gpg` tool with the commit content file. The gpg tool refers to the
default `keyring` for signer's public key. This is the same as manually
verifying using gpg tool.


# Verify using golang's crypto/openpgp package

`golang.org/x/crypto/openpgp` has a function `CheckArmoredDetachedSignature()`
to perform signature verification. Since this is not relying on `gpg`, it
requires the keyring to be passed as an argument.

```
func CheckArmoredDetachedSignature(keyring KeyRing, signed, signature io.Reader) (signer *Entity, err error)
```

Since we have been using armored signature, we can use an armored keyring as
well. Export an armored keyring with public key:
```
$ gpg --armor --export you@example.com > mykey.asc
```

This file would contain public key of you@example.com user only, from the
default keyring. Now, this can be read and an `io.Reader` can be created and
passed to `CheckArmoredDetachedSignature()` as the keyring. Or even the public
key string can be used directly to create a reader as:
```go
armoredKeyRingReader := strings.NewReader(`
-----BEGIN PGP PUBLIC KEY BLOCK-----

mQENBFmtHgABCADnfThM7q8D4pgUub9jMppSpgFh3ev84g3Csc3yQUlszEOVgXmu
YiSWP1oAiWFQ8ahCydh3LT8TnEB2QvoRNiExUI5XlXFwVfKW3cpDu8gdhtufs90Q
......
-----END PGP PUBLIC KEY BLOCK-----
`)

signature := strings.NewReader(`
-----BEGIN PGP SIGNATURE-----

 iQFHBAABCAAxFiEEoRt6IzxHaZkkUslhQyLeMqcmyU4FAloTCrsTHG1lQGRhcmtv
 d2x6ei5zcGFjZQAKCRBDIt4ypybJTul5CADmVxB4kqlqRZ9fAcSU5LKva3GRXx0+
 leX6vbzoyQztSWYgl7zALh4kB3a3t2C9EnnM6uehlgaORNigyMArCSY1ivWVviCT
 BvldSVi8f8OvnqwbWX0I/5a8KmItthDf5WqZRFjhcRlY1AK5Bo2hUGVRq71euf8F
 rE6wNhDoyBCEpftXuXbq8duD7D6qJ7QiOS4m5+ej1UCssS2WQ60yta7q57odduHY
 +txqTKI8MQUpBgoTqh+V4lOkwQQxLiz7hIQ/ZYLUcnp6fan7/kY/G7YoLt9pOG1Y
 vLzAWdidLH2P+EUOqlNMuVScHYWD1FZB0/L5LJ8no5pTowQd2Z+Nggxl
 =0uC8
 -----END PGP SIGNATURE-----
`)

commitData, err := os.Open("commit.txt")
...

entity, err := openpgp.CheckArmoredDetachedSignature(armoredKeyRingReader, commitData, signature)
```

In the above code, readers are created using `strings.NewReader` and `os.Open`.
All the inline keys can be read from file using `os.Open` and their readers
could be used in the same way as above to perform a verification.


# Verify using go-git

`gopkg.in/src-d/go-git.v4/plumbing/object`'s Commit type has `Verify` method to
perform verification of a commit, given an armored keyring.
```
func (c *Commit) Verify(armoredKeyRing string) (*openpgp.Entity, error)
```

This is same as `golang.org/x/crypto/openpgp` but the commit information is
implicit. Only an armored keyring is passed.

An example commit object can be created and verified as:
```golang
ts := time.Unix(0000000000, 0)
commit := &Commit{
    Hash:      plumbing.NewHash("8a9cea36fe052711fbc42b86e1f99a4fa0065deb"),
    Author:    Signature{Name: "darkowlzz", Email: "example@darkowlzz.space", When: ts},
    Committer: Signature{Name: "darkowlzz", Email: "example@darkowlzz.space", When: ts},
    Message: `status: simplify template command selection
`,
    TreeHash:     plumbing.NewHash("6572ba6df4f1fb323c8aaa24ce07bca0648b161e"),
    ParentHashes: []plumbing.Hash{plumbing.NewHash("ede5f57ea1280a0065beec96d3e1a3453d010dbd")},
    PGPSignature: `
-----BEGIN PGP SIGNATURE-----

iQFHBAABCAAxFiEEoRt6IzxHaZkkUslhQyLeMqcmyU4FAloTCrsTHG1lQGRhcmtv
d2x6ei5zcGFjZQAKCRBDIt4ypybJTul5CADmVxB4kqlqRZ9fAcSU5LKva3GRXx0+
leX6vbzoyQztSWYgl7zALh4kB3a3t2C9EnnM6uehlgaORNigyMArCSY1ivWVviCT
BvldSVi8f8OvnqwbWX0I/5a8KmItthDf5WqZRFjhcRlY1AK5Bo2hUGVRq71euf8F
rE6wNhDoyBCEpftXuXbq8duD7D6qJ7QiOS4m5+ej1UCssS2WQ60yta7q57odduHY
+txqTKI8MQUpBgoTqh+V4lOkwQQxLiz7hIQ/ZYLUcnp6fan7/kY/G7YoLt9pOG1Y
vLzAWdidLH2P+EUOqlNMuVScHYWD1FZB0/L5LJ8no5pTowQd2Z+Nggxl
=0uC8
-----END PGP SIGNATURE-----
`,
}
    
entity, err := commit.Verify(armoredKeyRing)
...
```

**NOTE**: The commit message contains a newline. While implementing this
verification feature in go-git, I spent a long time debugging what was causing
verification to fail, only to find out that a newline was missing from the
commit message. And that's one of the reasons for writing this post 😅
