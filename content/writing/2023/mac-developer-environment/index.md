---
title: "Mac Developer Environment"
date: 2023-10-30T09:58:20+08:00
lastmod: 2023-10-30T09:58:20+08:00
draft: false
toc: true
images:
tags:
  - untagged
---

## 配置 Homebrew
### 下载并安装 
[官网Shell](https://brew.sh/)
Or
[Install pkg](https://github.com/Homebrew/brew/releases/latest)

### 设置环境变量
从 macOS Catalina(10.15.x) 版开始，Mac使用zsh作为默认Shell，使用.zprofile，所以对应命令：
```
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

如果是 macOS Mojave 及更低版本，并且没有自己配置过zsh，使用.bash_profile：
```
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.bash_profile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

### 参考文档
[M1芯片Mac上Homebrew安装教程](https://zhuanlan.zhihu.com/p/341831809)

## 修改环境变量
### Zsh 增加环境变量
1. Edit ~/.zprofile
    ```
    sudo vi ~/.zprofile
    ```
2. Add new path
    ```
    export PATH="$PATH:[NEW_DIRECTORY]/bin"
    ```
3. Update ~/.zprofile
    ```
    source ~/.zprofile
    ```
4. Check PATH
    ```
    echo $PATH
    ```

### 参考文档
[Adding a new entry to the PATH variable in ZSH](https://stackoverflow.com/questions/11530090/adding-a-new-entry-to-the-path-variable-in-zsh)