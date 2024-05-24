---
title: "The easiest way to set up git on your system"
seoTitle: "Set Up Git Easily on Your System"
seoDescription: "Learn how to install and configure Git on Windows, Mac, and Linux easily"
datePublished: Thu May 23 2024 11:01:48 GMT+0000 (Coordinated Universal Time)
cuid: clwj57zbh000m09mgguurdoc7
slug: the-easiest-way-to-set-up-git-on-your-system
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/842ofHC6MaI/upload/931e678a202e606c328fdd720caeafff.jpeg
tags: programming, git, beginner, beginners

---

This article will help you install and configure git on your system.

# Install git

### Windows

You can download and install git from [here](https://git-scm.com/download/win). The process is similar to installing any other application on windows

### Mac OS

There are multiple ways to install git on Mac OS. The easiest is through brew using the following command `brew install git`

### Linux

Git is available in almost every main stream distro's repository. You can install git from your distro's repositories' as you would install any other package.

For Example,

On Ubuntu : `sudo apt install git`

On fedora: `sudo dnf install git`

On Arch: `sudo pacman -S git`

# Configure git

Firstly, open your terminal on linux/mac and the git bash terminal if you are on windows. You have got to configure your git instance with a username, your email and set your default branch to main. You can do it with the following commands.

* `git config --global user.name "<Name>`
    
* `git config --global user.email "<email>`
    
* `git config --global init.defaultBranch main`
    

These are the basic commands that I run to set up git on my system.