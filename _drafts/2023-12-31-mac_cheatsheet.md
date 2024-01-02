---
layout: single
title: Mac OS cheatsheet

toc: true
toc_label: "Contents"
toc_icon: "none"

tag: [Mac OS, Software]

header:
    teaser: /assets/img/apple/terminal.png
---

All the basic stuff I always forget.

## Show hidden files/folders

Open up a Terminal and run: 

```bash
# show
defaults write com.apple.Finder AppleShowAllFiles true
killall Finder

# hide
defaults write com.apple.Finder AppleShowAllFiles false
killall Finder
```

## Setup SSH key

This is a 2-step process; first create the key, then add it to the SSH agent for good

```bash
# create (leave passphrase empty)
ssh-keygen -t ed25519 -C "your_email@example.com"

# start agent
eval "$(ssh-agent -s)"

# set up config file (with following content)
touch ~/.ssh/config
#Host github.com
#  AddKeysToAgent yes
#  IdentityFile ~/.ssh/id_ed25519

# add key
ssh-add ~/.ssh/id_ed25519

# copy content to GitHub
pbcopy < ~/.ssh/id_ed25519.pub
```


## References

- [Generating a new SSH key and adding it to the ssh-agent](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
- [Adding a new SSH key to your account](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)