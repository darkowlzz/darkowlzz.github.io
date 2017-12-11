---
title: "Helm Chart Config Preview"
date: 2017-07-26T22:09:57+05:30
draft: false
tags: ["k8s", "helm"]
---

[Helm](https://github.com/kubernetes/helm) is a kubernetes(k8s) package manager
and it enables downloading k8s charts and installing them in k8s cluster. Charts
are configured with default values, which can be 
[customized](https://docs.helm.sh/using_helm/#customizing-the-chart-before-installing)
before installation.

`helm inspect [chart]` shows the `Chart.yaml` and `values.yaml` content of a
given chart. The configurations are stored in `values.yaml` and
`heml inspect values [chart]` shows only the content of `values.yaml`.

`values.yaml` of mariadb chart:
```
---
image: bitnami/mariadb:10.1.23-r2

## Specify an imagePullPolicy (Required)
imagePullPolicy: IfNotPresent

## Specify password for root user
# mariadbRootPassword:

## Create a database user
# mariadbUser:
# mariadbPassword:

## Create a database
# mariadbDatabase:
...
```

[Customizing the Chart Before Installing](https://docs.helm.sh/using_helm/#customizing-the-chart-before-installing)
describes how these configurations can be customized by passing a file name or
`--set` flags. But this would not preview the set configurations and install
immediately. Or fail if there were any unexpected values found in the
configuration.

To preview the customized configuration, helm dry-run and debug execution modes
can be coupled. This shows the resulting configuration and since it's a dry-run,
it won't result in actual installation.

```
$ helm install --dry-run --debug stable/mariadb --set mariadbDatabase=somedbname
...
NAME:   goodly-quoll
REVISION: 1
RELEASED: Wed Jul 26 22:31:22 2017
CHART: mariadb-0.6.3
USER-SUPPLIED VALUES:
mariadbDatabase: somedbname

COMPUTED VALUES:
image: bitnami/mariadb:10.1.23-r2
imagePullPolicy: IfNotPresent
mariadbDatabase: somedbname
...
```

**NOTE**: dry-run result doesn't mean the configuration is correct. It's just a
preview of the passed configuration parsed by helm. If there's any
misconfiguration, the installation would fail.
