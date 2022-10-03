---
layout: default
title: VSCode for Qt development
description: Configure VSCode for Qt development 
---

This post is Part 1 in a series of two where we look at how to configre VSCode

## Introduction

## Goal

The overarching goal is to have a (mostly) automated and consistent way of generating

## State of the art

Add links to posts in literature:

- [KDAB - Using Visual Studio Code for Writing Qt Applications (2019)](https://www.kdab.com/using-visual-studio-code-for-writing-qt-applications/)
- [KDAB - VS Code for Qt Applications – Part 1 (2020)](https://www.kdab.com/using-visual-studio-code-for-qt-apps-pt-1/)
- [KDAB - VS Code for Qt Applications – Part 2 (2020)](https://www.kdab.com/using-visual-studio-code-for-qt-apps-pt-2/)

## VSCode

The workflow is orchestrated by a minimal set of VSCode extensions and configuration files

The goal is to add as less customization as possible and yet achieve a workflow that is flexible and portable.

- **.vscode/settings.json** : TODO: describe how this files overrides the other settings  
- **.vscode/cmake-kits.json** : this is the key file where most of the magic happens. If you're familiar with QtCreator kit you will find a lot of similarities here.
- **.vscode/toolchain_<xxx>.cmake** : a toolchain file to describe compiler and other stuff
- **.vscode/launch.json**
- **.vscode/c_cpp_properties.json** ...find what is the purpose of this file

### Configuration files

The whole build pro

### Extensions

use Felgo extension
...do not install QtTools

### Configuration

``` bash
Target packages -> Network -> 
```

configure for local build and for remote