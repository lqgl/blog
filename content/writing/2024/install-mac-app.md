---
title: "Mac apps that cannot be opened or files are damaged"
date: 2024-01-19T11:56:40+08:00
lastmod: 2024-01-19T11:56:40+08:00
draft: false
toc: false
images:
tags:
  - Mac M1
  - App Install
---

There are two solutions to this problem:

1. System Preferences -> Security & Privacy -> Security -> Modify to Anywhere

2. Open the terminal and enter:
- For macOS10.12 - 10.15.7: sudo spctl --master-disable
- For macOS11 and later versions: sudo spctl --global-disable