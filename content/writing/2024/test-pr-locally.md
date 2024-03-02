---
title: "Test PR Locally"
date: 2024-03-02T16:49:38+08:00
lastmod: 2024-03-02T16:49:38+08:00
draft: false
toc: false
images:
tags:
  - "Git"
---

## How to test PR locally when recieve a Pull Request?

To fetch a remote PR into your local repo,

```
git fetch origin pull/$ID/head:$BRANCHNAME
```

where $ID is the pull request id and $BRANCHNAME is the name of the new branch that you want to create. Once you have created the branch, then simply

```
git checkout $BRANCHNAME
```

For instance, let's imagine you want to checkout pull request #2 from the origin main branch:

```
git fetch origin pull/2/head:MASTER
```

See the [official GitHub documentation](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/checking-out-pull-requests-locally) for more.

## Refrences

[How can I check out a GitHub pull request with git?](https://stackoverflow.com/questions/27567846/how-can-i-check-out-a-github-pull-request-with-git)