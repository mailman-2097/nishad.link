---
author: Nishad Saithaly
pubDatetime: 2023-03-07T02:28:59Z
title: Getting started with JAMstack
postSlug: getting-started-with-jamstack
featured: true
draft: false
tags:
  - linux
  - wsl
  - windows
  - microsoft
ogImage: "https://raw.githubusercontent.com/mailman-2097/nishad.link/master/public/assets/blog3.svg"
description: Custom WSL in Windows
---

## Table of contents

## Introduction

It is better to create custom WSL instances especially if you want to work with `RHEL clones`.

I found that the RHEL clones available from Microsoft are not fit for purpose.

### Setup Instructions

1. Use the Container image as the base for your WSL

```bash

podman run --name rhel9 registry.redhat.io/rhel9/go-toolset
podman export -o rhel9ws2-image.tar.gz rhel9
wsl --import rhel9a C:\WSL\rhel9a .\rhel9ws2-image.tar.gz
wsl -d rhel9a
```

1. For RHEL you may need a valid license for software updates

```bash
subscription-manager register
# Install sudo and bash completion
dnf install -y sudo bash-completion
```

1. Configure default user

```bash
groupadd -g 1000 nishad
getent group nishad
useradd -m -s /bin/bash -u 1000 -g 1000 nishad
passwd nishad
getent group
usermod -aG wheel nishad
su - nishad
```

1. Configure WSL defaults

```bash
vi /etc/wsl.conf
```
[boot]
systemd = true

[user]
default = nishad

[network]
hostname = WSL-RH9A-AVD

[automount]
root = /
options = "metadata,uid=1000,gid=1000,umask=22,fmask=11,case=off"

# Incase you want to restrict or modify
# options = "metadata,umask=022,fmask=133"

```

You can review advanced settings [here](https://learn.microsoft.com/en-us/windows/wsl/wsl-config)

1. Restart WSL

```bash

wsl --terminate rhel9a
wsl -d rhel9a

```

## Closing Remarks

With this method you can setup multiple linux WSL instances and experiment safely

## References

[Medium Article](https://medium.com/@AnupamMajhi/windows-red-hat-rhel9-with-wsl2-bafa45be0131)
[Redhat Article](https://developers.redhat.com/articles/2023/11/15/create-customized-rhel-images-wsl-environment#workflow)