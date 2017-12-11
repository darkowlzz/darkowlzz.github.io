---
title: "Minikube Config"
date: 2017-07-24T13:48:53+05:30
draft: false
tags: ["k8s", "minikube"]
---

Minikube is a kubernetes(k8s) project to enable running single node k8s cluster
locally inside a VM, which can be used by developers to prepare their apps for
k8s in development phase. For more info, refer their [github project](https://github.com/kubernetes/minikube).

Minikube starts with some sane default VM configs for any modern machine.
```
DefaultMemory       = 2048
DefaultCPUS         = 2
DefaultDiskSize     = "20g"
MinimumDiskSizeMB   = 2000
DefaultVMDriver     = "virtualbox"
```

Which is fine for most of the time. But for various reasons, the defaults may
not be what one needs. The VM configs can be overridden by passing flags like:
```
$ minikube start --memory 1024
```

But let's say, we are constrained by memory on the system and missing the flag
could result in freezing the whole system due to a 2GB memory VM. To avoid this,
minikube config file can be set to override the defaults all the time. The
config file is located at `~/.minikube/config/config.json`, until explicitly
changed. The config can be used to enable/disable k8s add-ons and set various
parameter values for the setup.

To get the names of the configurable fields, run `minikube config -h`.
```
Configurable fields:

 * vm-driver
 * v
 * cpus
 * disk-size
 * host-only-cidr
 * memory
 * log_dir
 * kubernetes-version
 * iso-url
 * WantUpdateNotification
 * ReminderWaitPeriodInHours
 * WantReportError
 * WantReportErrorPrompt
 * WantKubectlDownloadMsg
 * profile
 * dashboard
 * addon-manager
 * default-storageclass
 * kube-dns
 * heapster
 * ingress
 * registry
 * registry-creds
 * default-storageclass
 * hyperv-virtual-switch
 * disable-driver-mounts
```

That's the configurable fields at the time of writing this post. `minikube config`
command lets you change the config without manually editing the config file.

```
$ minikube config set memory 1024
$ minikube config set disk-size 2000MB
```
