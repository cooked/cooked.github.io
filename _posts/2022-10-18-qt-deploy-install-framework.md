---
layout: single
title: Deploy apps with Qt Install Framework
tags: [Qt, Qt Installer Framework]

header:
  teaser: https://www.ics.com/sites/default/files/styles/blog_detail/public/images/install%201000x400.jpg
#    teaser: https://doc.qt.io/qtinstallerframework/images/ifw-overview.png

---

A step-by-step guide to create installers for your Qt applications.  

## Context

It took me few trial and errors before being able to successfully package and deploy a Qt application. It turned out that all the information is available in Qt official documentation but somehow is too wordy and does not provide a clear and concise workflow overview (at least to my taste). The following is an attempt to fill that gap.  

These notes assume you:

- are on a Windows machine  
- have a working installation of Qt5
- have installed Qt Installer Framework tool (QtIF)

## Workflow

At a very high level deploying a Qt application can be described by these few steps (after the application is built have it built):

1. prepare folder structure and configuration files expected by the QtIF
2. add to the structure the application executable, any application resource files/folders and all the application dependencies (Qt, QML and lower level)
3. pack everything into an executable installer

The following sections detail each one of those steps.

### STEP 1 - Configuration

The QtIF expects a very specific folder structures and a number of configuration files, as described in the [online documentation](https://doc.qt.io/qtinstallerframework/ifw-tutorial.html)
(Creating a Package Directory, Creating a Configuration File, Creating a Package Information File)

``` yaml
- <installer folder>
  - config
    - config.xml
  - packages
    - <package>
      - data
        - <any data folder you need depployed>
        - <any data file you need depployed>
      - meta
        - package.xml
```

### STEP 2 - Import the data

This is possibly be the most critical step of the whole process. It consists of:

- copying the application .exe file to the <package>/data folder (taking it from the directory where QtCreator build it to)  
- run [**windeployqt.exe**](https://doc.qt.io/qt-5/windows-deployment.html) with the proper flags to **resolve ALL dependencies**
- copy any external resource needed alongside the application (i.e. any other resource not been resolved by the previous step)

``` bash
cd <installer folder>\packages\<package>/data
# copy/paste your application exe in the folder above

> C:\Qt\5.15.2\mingw81_64\bin\windeployqt.exe --qmldir <your-app-qml-source-folder> yourApp.exe

```

The Windows deployment tool windeployqt is designed to automate the process of creating a deployable folder containing the Qt-related dependencies (libraries, QML imports, plugins, and translations) required to run the application from that folder. To have an overview of all the dependencies that the tool might be able to solve have a look [here](https://doc.qt.io/qt-5/windows-deployment.html#creating-the-application-package).

{: .notice--warning}
 **IMPORTANT** - The correct folder to pass to **--qmldir** is the one where **YOUR QML files** are, which normally coincides with your application source folder. DO NOT pass the Qt5 QML folder, that might result in missing dependencies and/or installing unnecessary dependencies. 

{: .notice--info}
 **NOTE** - **windeployqt.exe** comes with Qt and you will find it in **<Qt-installation folder>\bin\windeployqt.exe**. You need to use the full path unless the Qt folder is in your PATH.

### STEP 3 - Build the installer

```bash
# change to the <root installer> folder
cd  <yourAppProjectFolder>/installer

# run the 'binarycreator' (tool shipped with the Install Framework..)..need to get its path though
# - passing the location of the config file (-c)
# - passing the location of the packages (-p)
# - provide the name of the installer artifact to be created

> <QtIFW_Path>/bin/binarycreator.exe -c config/config.xml -p packages yourAppIntaller.exe

```  
