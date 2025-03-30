---
title: 'macbook.init(): a dev machine set-up guide'
date: 2022-09-10 23:05:39
tags:
  - setup
  - guide
category:
photos: /data/images/macbook-init-a-dev-machine-setup-guide/cover.jpg
description: As I started my new job, I decided to log the steps to set up my system, mostly for future me, but also for anyone else.
---
<sup> Photo by [Alexander Shatov](https://unsplash.com/@alexbemore) on [Unsplash](https://unsplash.com)</sup>

I started a new job, and I am excited. I get a new MacBook delivered and after keeping the delicate wrapping paper intact during the intricate unboxing, it dawns upon me that it is not _my_ MacBook. It is a brand-new system devoid of all the custom tools that make my (work) life easier. On top of it, I do not even remember what those tools were, to begin with. Was it `oh-my-zsh`, or was I using `fish`? What did I do last time to get the top bar show Bluetooth? I end up setting small things like these, slowly every day, until it becomes all familiar, and I forget about them.

Not this time, I said to myself. As I start working at Okta, I decided to log the steps to set up my system, mostly for future me, but also for anyone else stumbling on the same StackOverflow articles over and over again.

## Update OS
Yeah, just update the OS now that you have patience. It is probably outdated and the next step _may_ require it.

## Install Xcode
Do not wait on some CLI tool to fail on you later and install Xcode now. Might get lucky running `xcode-select --install`.

## Install Firefox/Brave
Use Safari to install Firefox and Brave and log into them. Also set up extensions like -
  - Password managers
  - Ad blockers
  - Web3 wallets

## Set up terminal environment

### Install iTerm2
Download the latest build [here](https://iterm2.com/downloads/stable/latest)

### Install homebrew
Refer https://brew.sh/#install, or
```sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
### Install oh my zsh
Refer https://ohmyz.sh/#install, or
```sh
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```
### Disable history sharing across zsh sessions
echo "unsetopt share_history\nsetopt no_share_history" >> ~/.zshrc

## Setup keys 🔑
SSH and GPG keys need to be generated/imported and updated __everywhere__, especially GitHub.

### SSH
`ssh-keygen -t ecdsa -b 521`

### PGP
`gpg --import <private-key-file>`

### Install nvm
Refer https://github.com/nvm-sh/nvm#installing-and-updating
```sh
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
```
### Install tmuxinator
```sh
brew insatll tmuxinator
```

### Customize vim
Restore dotfiles and customize vim and other things.


## Next steps
At some point, I should figure out that something is wrong with the trackpad and check that box that says __Tap to click__ in settings. This is a living document and I will keep adding things to it, as I find. Now I should get back to that new system I just configured and get some work done!
