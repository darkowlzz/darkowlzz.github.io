# darkowlzz.github.io:source

[![Build Status](https://travis-ci.org/darkowlzz/darkowlzz.github.io.svg?branch=source)](https://travis-ci.org/darkowlzz/darkowlzz.github.io)

Build [darkowlzz.github.io](https://darkowlzz.github.io) using [hugo](https://gohugo.io).


## Setup

1. [Install hugo](https://gohugo.io/overview/installing/).
2. Clone the `source` branch of repo with submodules.
```
git clone --recursive --branch source https://github.com/darkowlzz/darkowlzz.github.io.git
```
3. Generate pages:
```
hugo --theme=after-dark
```
4. Serve the pages:
```
hugo server
```

### Update theme (after-dark)

This project uses a forked version of
[after-dark](https://github.com/comfusion/after-dark) hugo theme.

When [after-dark](https://github.com/darkowlzz/after-dark) is updated, the
commit hash of the submodule should be changed with
```
git submodule foreach git pull origin master
```


## Deployment

This project auto-deploys at every commit to `source` branch. A build is
performed at travis and `master` branch is updated with the generated pages.
Deployment script has been taken from
[rcoedo](https://github.com/rcoedo/rcoedo.github.io/tree/source).
