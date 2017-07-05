---
title: "Securing Custom Deployments via Travis CI"
date: 2017-07-05T17:30:54+05:30
draft: false
---

* Custom Deployment
* Encrypted Secrets in Build Environment
* Limiting Deployments From Specific Branches

[Travis CI - Test and Deploy with Confidence](https://travis-ci.org/). DEPLOY.
**D E P L O Y**.

Travis CI can be used as a publicly hosted deployment system with inbuilt
[support](https://docs.travis-ci.com/user/deployment/) for deployments to
various providers. There are well documented guides about doing the same. But
once in a while, one might end up using some service that's not supported by Travis CI
or even a specific feature that's not supported, although there's support for
the provider.

# Custom Deployments

Travis CI provides ways of run scripts, from the code repo, which can be used
as deployment scripts. Script execution commands can be added to any of the
[build stages](https://docs.travis-ci.com/user/customizing-the-build/),
depending on the requirements. For a deployment script, `after_success` is a
good place to run them.

```yml
# .travis.yml
...
after_success:
    - ./scripts/deploy.sh
...
```

> **The exit code of `after_success`, `after_failure`, `after_script` and 
subsequent stages do not affect the build result. However, if one of these
stages times out, the build is marked as a failure.**


# Encrypted Secrets in Build Environment

When doing custom deployments, the secrets must be stored in environment
variables in encrypted form. Encrypted env vars look like this in .travis.yml
file:

```yml
# .travis.yml
env:
  global:
    - APP_NAME="foo"
    - secure: "N79ZQG6jUTP5o0YSc1dTtb02PV+1j6sjM/DlvtCEcdaiPiEH"
    - secure: "L08rVW1wuAz5P4j3yiAJkxG0Dog5cr8YSPzCCnguDrBy7Ok5"
```

Even the variable names are not shown. To create encrypted variables, use `travis`
cli. Refer their [Encryption Key Guide](https://docs.travis-ci.com/user/encryption-keys).

Sometimes, there's need to encrypt certain files. They have another guide for
that, [Encrypting Files Guide](https://docs.travis-ci.com/user/encrypting-files/).
Encrypting files generate a new file with encrypted content. The decryption key
is saved in travis, under the specific repo.

The encrypted keys and files can only be decrypted when there's a build by the
owner of the repo. So, a build of a PR by a contributor, would fail at decryption
stage because travis has no key associated with that user.

Also refer [Travis CI: Best Practices in Securing Your Data](https://docs.travis-ci.com/user/best-practices-security/).

# Limiting Deployments From Specific Branches

Deployments require secrets. Deployments triggered by PR from contributors would
fail due to no decryption key. Hence, it's safe. Other's can't deploy. But in
addition to that, the build script can be made a little smarter and exit when
certain conditions are met.

Deployment is generally not intended when:

- it's a PR
- not master branch or any specific production branch

For such scenarios, some conditions in the deployment script could be used.

```
if [ "$TRAVIS_BRANCH" != "$DEPLOY_BRANCH" ]; then
    echo "Travis should only deploy from the DEPLOY_BRANCH ($DEPLOY_BRANCH) branch"
    exit 0
else
    if [ "$TRAVIS_PULL_REQUEST" != "false" ]; then
        echo "Travis should not deploy from pull requests"
        exit 0
    else
        # actually deploy
        .....
        .....
    fi
fi
```

This would ensure that deployments are not triggered when a new branch is built
and when a PR is sent. Preventing accidental deployments from branches by the
repo owner.


This site has CI/CD setup on travis and deploys to github pages at every commit
to the `source` branch. Refer [`.travis.yml`](https://github.com/darkowlzz/darkowlzz.github.io/blob/source/.travis.yml)
and [`deploy.sh`](https://github.com/darkowlzz/darkowlzz.github.io/blob/source/scripts/deploy.sh) 
files for working examples. They are inspired by work of
[rcoedo](https://github.com/rcoedo) and a sort of
[tutorial](https://haruair.github.io/post/setup-hugo-blog-on-github-pages-with-travis-ci/)
by [haruair](https://github.com/haruair). 🦉
