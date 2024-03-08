---
layout: post
title: Install Node.js on WSL
date: 2024-03-08
categories: wsl
---

## Install NVM

```console
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/master/install.sh | bash
```

In new console run, install NVM (LTS version)

```console
nvm install --lts
```

Verify

```console
node --version
```