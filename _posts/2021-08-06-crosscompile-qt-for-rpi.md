---
layout: single
title: Cross-compile Qt5 for the Raspberry Pi
tags: [Qt5, Raspberry Pi, C/C++]
header:
    teaser: /assets/img/raspberry_qt_logo.png
---

## UPDATE 
I've replaced the manual configuration (command line and scripts) described below with a more automated approach that relies on [Buildroot for the creation of cross-compilation toolchain and target filesystem]({% post_url 2022-08-06-raspberry_buildroot_xcompile %}).   

## Introduction

Oftentimes when working on embedded projects it is not recommended (or not even possible) to develop directly on the target hardware, due to limitations of some sort. The Raspberry Pi, for example, does not offer a lot of computing power, which is why developing graphical interfaces on it is might be quite time consuming. A much more elegant, and ultimately faster, way of developing it is to write and build code on a host PC, then deploy the application to the (target) Raspberry Pi via a remote connection.

The workflow requires more steps and it might seem convoluted at first (it also requires to be kept up to date), but it is actually a better way to go about the whole development process. 

There are two key aspects to cross-compiling:

- the **toolchain**, i.e. the set of tools (compiler, linker, etc.) available on the development machine (host) needed to build the source code into binary files for the embedded device (target).
- 
- the **target filesystem**, i.e. the set of libraries and other relevant resources representing the target environment of the embedded device

## TL;DR
The 

## Workflow

the steps below are a combination of all the old links (see above) I found but in the end mostly based on the [https://github.com/abhiTronix/raspberry-pi-cross-compilers/blob/master/QT_build_instructions.md](https://github.com/abhiTronix/raspberry-pi-cross-compilers/blob/master/QT_build_instructions.md)

### Prepare Raspberry Pi

TODO: currently installed full system on SD but replace with download+chroot method

```bash
# 1. update raspberry
sudo apt-get update
sudo apt-get dist-upgrade
sudo reboot
sudo rpi-update
sudo reboot

# enable rsync sudo
echo "$USER ALL=NOPASSWD:$(which rsync)" | sudo tee --append /etc/sudoers

# 2. install development packages
sudo apt-get build-dep qt5-qmake
sudo apt-get build-dep libqt5gui5
sudo apt-get build-dep libqt5webengine-data
sudo apt-get build-dep libqt5webkit5
sudo apt-get install libudev-dev libinput-dev libts-dev libxcb-xinerama0-dev libxcb-xinerama0 gdbserver

		# (optional, only for qtmultimedia)
sudo apt install libjpeg-dev libpng-dev libtiff-dev
sudo apt install libavcodec-dev libavformat-dev libswscale-dev libv4l-dev
sudo apt install libxvidcore-dev libx264-dev
		# (optional) gstreamer (multimedia) packages
sudo apt install libgstreamer1.0-0 gstreamer1.0-plugins-base gstreamer1.0-plugins-good gstreamer1.0-plugins-bad gstreamer1.0-plugins-ugly gstreamer1.0-libav gstreamer1.0-doc gstreamer1.0-tools gstreamer1.0-x gstreamer1.0-alsa gstreamer1.0-gl gstreamer1.0-gtk3 gstreamer1.0-pulseaudio
sudo apt install libgstreamer1.0-dev  libgstreamer-plugins-base1.0-dev

# 3. create folder for Qt
sudo mkdir /usr/local/qt5.15
sudo chown -R pi:pi /usr/local/qt5.15

# 4. create symlinks
sudo ln -sf -r /usr/include/arm-linux-gnueabihf/asm /usr/include
sudo ln -sf -r /usr/include/arm-linux-gnueabihf/gnu /usr/include
sudo ln -sf -r /usr/include/arm-linux-gnueabihf/bits /usr/include
sudo ln -sf -r /usr/include/arm-linux-gnueabihf/sys /usr/include
sudo ln -sf -r /usr/include/arm-linux-gnueabihf/openssl /usr/include
sudo ln -sf /usr/lib/arm-linux-gnueabihf/crtn.o /usr/lib/crtn.o
sudo ln -sf /usr/lib/arm-linux-gnueabihf/crt1.o /usr/lib/crt1.o
sudo ln -sf /usr/lib/arm-linux-gnueabihf/crti.o /usr/lib/crti.o

# 5. (optional) setup ssh key folders, if not there (keys are copied over from PC later)
mkdir .ssh
cd .ssh
touch authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

### Prepare Host

```bash
# basic prep
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install gcc git bison python gperf pkg-config gdb-multiarch

### 
# do keys  /// alow root login without password..... https://raspberrypi.stackexchange.com/questions/48056/how-to-login-as-root-remotely
ssh-keygen -t rsa -C pi@ -P "" -f ~/.ssh/rpi_pi_id_rsa
cat ~/.ssh/rpi_pi_id_rsa.pub | ssh pi@192.168.1.172 'cat >> .ssh/authorized_keys && chmod 640 .ssh/authorized_keys'

# create dir
mkdir ~/qtrpi4
sudo mkdir ~/qtrpi4/tools
sudo mkdir ~/qtrpi4/build
sudo mkdir ~/qtrpi4/sysroot
sudo mkdir ~/qtrpi4/sysroot/usr
sudo mkdir ~/qtrpi4/sysroot/opt
sudo chown -R 1000:1000 ~/qtrpi4   # makes you own the folder
cd ~/qtrpi4

# sync stuff from raspy
rsync -avz --rsync-path="sudo rsync" --delete pi@192.168.1.172:/lib sysroot
rsync -avz --rsync-path="sudo rsync" --delete pi@192.168.1.172:/usr/include sysroot/usr
rsync -avz --rsync-path="sudo rsync" --delete pi@192.168.1.172:/usr/lib sysroot/usr
rsync -avz --rsync-path="sudo rsync" --delete pi@192.168.1.172:/opt/vc sysroot/opt

### download fix-symlink script (all sources are the same file)
#wget https://raw.githubusercontent.com/riscv/riscv-poky/master/scripts/sysroot-relativelinks.py
wget https://raw.githubusercontent.com/abhiTronix/rpi_rootfs/master/scripts/sysroot-relativelinks.py
sudo chmod +x sysroot-relativelinks.py
./sysroot-relativelinks.py sysroot

# download and config (a recent) compiler/toolchain
# https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-a/downloads
cd ~/qtrpi4/tools
#wget https://developer.arm.com/-/media/Files/downloads/gnu-a/10.3-2021.07/binrel/gcc-arm-10.3-2021.07-x86_64-arm-none-linux-gnueabihf.tar.xz
#tar xf gcc-arm-10.3-2021.07-x86_64-arm-none-linux-gnueabihf.tar.xz
wget https://sourceforge.net/projects/raspberry-pi-cross-compilers/files/Raspberry%20Pi%20GCC%20Cross-Compiler%20Toolchains/Buster/GCC%208.3.0/Raspberry%20Pi%203A%2B%2C%203B%2B%2C%204/cross-gcc-8.3.0-pi_3%2B.tar.gz
tar xf cross-gcc-*.tar.gz
cd ~/qtrpi4

### download and config qt 
# TODO... change with clone repo and checkout branch

wget http://download.qt.io/archive/qt/5.15/5.15.2/single/qt-everywhere-src-5.15.2.tar.xz  # -O qt-src.tar.xz
tar xf qt-everywhere-src-5.15.2.tar.xz
# adjust compiler
cp -R qt-everywhere-src-5.15.2/qtbase/mkspecs/linux-arm-gnueabi-g++ qt-everywhere-src-5.15.2/qtbase/mkspecs/linux-arm-gnueabihf-g++
sed -i -e 's/arm-linux-gnueabi-/arm-linux-gnueabihf-/g' qt-everywhere-src-5.15.2/qtbase/mkspecs/linux-arm-gnueabihf-g++/qmake.conf

# qt config
# https://doc.qt.io/qt-5/configure-linux-device.html
cd qtrpi4/build
../qt-everywhere-src-5.15.2/configure -release -opengl es2 -eglfs -device linux-rasp-pi4-v3d-g++ \
-device-option CROSS_COMPILE=~/qtrpi4/tools/cross-pi-gcc-8.3.0-2/bin/arm-linux-gnueabihf- \
-sysroot ~/qtrpi4/sysroot \
-prefix /usr/local/qt5.15 \
-extprefix ~/qtrpi4/qt5.15 \
-opensource -confirm-license -skip qtscript -skip qtwayland -skip qtwebengine -nomake tests -make libs -pkg-config -no-use-gold-linker -v -recheck \
-L$HOME/qtrpi4/sysroot/usr/lib/arm-linux-gnueabihf \
-I$HOME/qtrpi4/sysroot/usr/include/arm-linux-gnueabihf
make -j$(nproc)
make install

# NOTE 1
# -sysroot    // this is where the rpi system is contained, on the PC
# -prefix     // this is where the compiled qt will be copied to, on the RPI (the folder created in step3 of rpi preparation above) 
# -extprefix  // this is where the compiled qt is placed on the PC

# NOTE 2
# run "rm -rf *" before running config again

# qt deploy
cd ~/qtrpi4
rsync -avz --rsync-path="sudo rsync" --delete qt5.15 pi@192.168.1.172:/usr/local
```

### Summary from CONFIGURE 

Note that **qt5** shall be configured after gstreamer 1.18.4 and opencv 4.5.3 installation
    
```bash
Building on: linux-g++ (x86_64, CPU features: mmx sse sse2)
Building for: devices/linux-rasp-pi4-v3d-g++ (arm, CPU features: neon)
Target compiler: gcc 10.3.1
Configuration: cross_compile compile_examples largefile neon precompile_header shared shared rpath release c++11 c++14 c++17 c++1z concurrent dbus reduce_exports stl
Build options:
    Mode ................................... release
    Optimize release build for size ........ no
    Building shared libraries .............. yes
    Using C standard ....................... C11
    Using C++ standard ..................... C++17
    Using ccache ........................... no
    Using new DTAGS ........................ no
    Relocatable ............................ yes
    Using precompiled headers .............. yes
    Using LTCG ............................. no
    Target compiler supports:
    NEON ................................. yes
    Build parts ............................ libs
Qt modules and options:
    Qt Concurrent .......................... yes
    Qt D-Bus ............................... yes
    Qt D-Bus directly linked to libdbus .... yes
    Qt Gui ................................. yes
    Qt Network ............................. yes
    Qt Sql ................................. yes
    Qt Testlib ............................. yes
    Qt Widgets ............................. yes
    Qt Xml ................................. yes
Support enabled for:
    Using pkg-config ....................... yes
    udev ................................... yes
    Using system zlib ...................... yes
    Zstandard support ...................... no
Qt Core:
    DoubleConversion ....................... yes
    Using system DoubleConversion ........ yes
    GLib ................................... yes
    iconv .................................. no
    ICU .................................... yes
    Built-in copy of the MIME database ..... yes
    Tracing backend ........................ <none>
    Logging backends:
    journald ............................. no
    syslog ............................... no
    slog2 ................................ no
    PCRE2 .................................. yes
    Using system PCRE2 ................... yes
Qt Network:
    getifaddrs() ........................... yes
    IPv6 ifname ............................ yes
    libproxy ............................... no
    Linux AF_NETLINK ....................... yes
    OpenSSL ................................ yes
    Qt directly linked to OpenSSL ........ no
    OpenSSL 1.1 ............................ yes
    DTLS ................................... yes
    OCSP-stapling .......................... yes
    SCTP ................................... no
    Use system proxies ..................... yes
    GSSAPI ................................. no
Qt Gui:
    Accessibility .......................... yes
    FreeType ............................... yes
    Using system FreeType ................ yes
    HarfBuzz ............................... yes
    Using system HarfBuzz ................ yes
    Fontconfig ............................. yes
    Image formats:
    GIF .................................. yes
    ICO .................................. yes
    JPEG ................................. yes
        Using system libjpeg ............... yes
    PNG .................................. yes
        Using system libpng ................ yes
    Text formats:
    HtmlParser ........................... yes
    CssParser ............................ yes
    OdfWriter ............................ yes
    MarkdownReader ....................... yes
        Using system libmd4c ............... no
    MarkdownWriter ....................... yes
    EGL .................................... yes
    OpenVG ................................. no
    OpenGL:
    Desktop OpenGL ....................... no
    OpenGL ES 2.0 ........................ yes
    OpenGL ES 3.0 ........................ yes
    OpenGL ES 3.1 ........................ yes
    OpenGL ES 3.2 ........................ yes
    Vulkan ................................. yes
    Session Management ..................... yes
Features used by QPA backends:
    evdev .................................. yes
    libinput ............................... yes
    INTEGRITY HID .......................... no
    mtdev .................................. yes
    tslib .................................. no
    xkbcommon .............................. yes
    X11 specific:
    XLib ................................. yes
    XCB Xlib ............................. yes
    EGL on X11 ........................... yes
    xkbcommon-x11 ........................ yes
QPA backends:
    DirectFB ............................... no
    EGLFS .................................. yes
    EGLFS details:
    EGLFS OpenWFD ........................ no
    EGLFS i.Mx6 .......................... no
    EGLFS i.Mx6 Wayland .................. no
    EGLFS RCAR ........................... no
    EGLFS EGLDevice ...................... yes
    EGLFS GBM ............................ yes
    EGLFS VSP2 ........................... no
    EGLFS Mali ........................... no
    EGLFS Raspberry Pi ................... no
    EGLFS X11 ............................ yes
    LinuxFB ................................ yes
    VNC .................................... yes
Qt Sql:
    SQL item models ........................ yes
Qt Widgets:
    GTK+ ................................... no
    Styles ................................. Fusion Windows
Qt PrintSupport:
    CUPS ................................... yes
Qt Sql Drivers:
    DB2 (IBM) .............................. no
    InterBase .............................. no
    MySql .................................. no
    OCI (Oracle) ........................... no
    ODBC ................................... yes
    PostgreSQL ............................. yes
    SQLite2 ................................ no
    SQLite ................................. yes
    Using system provided SQLite ......... no
    TDS (Sybase) ........................... yes
Qt Testlib:
    Tester for item models ................. yes
Serial Port:
    ntddmodm ............................... no
Qt SerialBus:
    Socket CAN ............................. yes
    Socket CAN FD .......................... yes
    SerialPort Support ..................... yes
Further Image Formats:
    JasPer ................................. no
    MNG .................................... no
    TIFF ................................... yes
    Using system libtiff ................. yes
    WEBP ................................... yes
    Using system libwebp ................. yes
Qt QML:
    QML network support .................... yes
    QML debugging and profiling support .... yes
    QML just-in-time compiler .............. yes
    QML sequence object .................... yes
    QML XML http request ................... yes
    QML Locale ............................. yes
Qt QML Models:
    QML list model ......................... yes
    QML delegate model ..................... yes
Qt Quick:
    Direct3D 12 ............................ no
    AnimatedImage item ..................... yes
    Canvas item ............................ yes
    Support for Qt Quick Designer .......... yes
    Flipable item .......................... yes
    GridView item .......................... yes
    ListView item .......................... yes
    TableView item ......................... yes
    Path support ........................... yes
    PathView item .......................... yes
    Positioner items ....................... yes
    Repeater item .......................... yes
    ShaderEffect item ...................... yes
    Sprite item ............................ yes
QtQuick3D:
    Assimp ................................. yes
    System Assimp .......................... no
Qt Scxml:
    ECMAScript data model for QtScxml ...... yes
Qt Gamepad:
    SDL2 ................................... no
Qt 3D:
    Assimp ................................. yes
    System Assimp .......................... no
    Output Qt3D GL traces .................. no
    Use SSE2 instructions .................. no
    Use AVX2 instructions .................. no
    Aspects:
    Render aspect ........................ yes
    Input aspect ......................... yes
    Logic aspect ......................... yes
    Animation aspect ..................... yes
    Extras aspect ........................ yes
Qt 3D Renderers:
    OpenGL Renderer ........................ yes
    RHI Renderer ........................... no
Qt 3D GeometryLoaders:
    Autodesk FBX ........................... no
Qt Bluetooth:
    BlueZ .................................. no
    BlueZ Low Energy ....................... no
    Linux Crypto API ....................... no
    Native Win32 Bluetooth ................. no
    WinRT Bluetooth API (desktop & UWP) .... no
    WinRT advanced bluetooth low energy API (desktop & UWP) . no
Qt Sensors:
    sensorfw ............................... no
Qt Quick Controls 2:
    Styles ................................. Default Fusion Imagine Material Universal
Qt Quick Templates 2:
    Hover support .......................... yes
    Multi-touch support .................... yes
Qt Positioning:
    Gypsy GPS Daemon ....................... no
    WinRT Geolocation API .................. no
Qt Location:
    Qt.labs.location experimental QML plugin . yes
    Geoservice plugins:
    OpenStreetMap ........................ yes
    HERE ................................. yes
    Esri ................................. yes
    Mapbox ............................... yes
    MapboxGL ............................. yes
    Itemsoverlay ......................... yes
QtXmlPatterns:
    XML schema support ..................... yes
Qt Multimedia:
    ALSA ................................... yes
    GStreamer 1.0 .......................... yes
    GStreamer 0.10 ......................... no
    Video for Linux ........................ yes
    OpenAL ................................. no
    PulseAudio ............................. no
    Resource Policy (libresourceqt5) ....... no
    Windows Audio Services ................. no
    DirectShow ............................. no
    Windows Media Foundation ............... no
Qt TextToSpeech:
    Flite .................................. no
    Flite with ALSA ........................ no
    Speech Dispatcher ...................... no
Qt Tools:
    Qt Assistant ........................... yes
    Qt Designer ............................ yes
    Qt Distance Field Generator ............ yes
    kmap2qmap .............................. yes
    Qt Linguist ............................ yes
    Mac Deployment Tool .................... no
    makeqpf ................................ yes
    pixeltool .............................. yes
    qdbus .................................. yes
    qev .................................... yes
    Qt Attributions Scanner ................ yes
    qtdiag ................................. yes
    qtpaths ................................ yes
    qtplugininfo ........................... yes
    Windows deployment tool ................ no
    WinRT Runner Tool ...................... no
Qt Tools:
    QDoc ................................... yes

Note: Also available for Linux: linux-clang linux-icc

Note: PKG_CONFIG_LIBDIR automatically set to /home/stefano/qtrpi4/sysroot/usr/lib/pkgconfig:/home/stefano/qtrpi4/sysroot/usr/share/pkgconfig:/home/stefano/qtrpi4/sysroot/usr/lib/arm-linux-gnueabihf/pkgconfig

Note: PKG_CONFIG_SYSROOT_DIR automatically set to /home/stefano/qtrpi4/sysroot

Qt is now configured for building. Just run 'make'.
Once everything is built, you must run 'make install'.
Qt will be installed into '/home/stefano/qtrpi4/qt5.15'.

Prior to reconfiguration, make sure you remove any leftovers from
the previous build.
```


### Finalize Raspberry Pi setup

```bash
echo /usr/local/qt5.15/lib | sudo tee /etc/ld.so.conf.d/qt5.15.conf
sudo ldconfig
```

### Configure QtCreator
On the host computer.  

following [https://www.interelectronix.com/configuring-qt-creator-ubuntu-20-lts-cross-compilation.html](https://www.interelectronix.com/configuring-qt-creator-ubuntu-20-lts-cross-compilation.html)

adding debugger [https://www.youtube.com/watch?v=1d2bh7iUKNc&list=PLFsidzAJDEbBr3l0BNMDcDQlUM04GRlf2](https://www.youtube.com/watch?v=1d2bh7iUKNc&list=PLFsidzAJDEbBr3l0BNMDcDQlUM04GRlf2)

### Configure Qt project

deploy applications (official) - [https://doc.qt.io/qtcreator/creator-deployment-embedded-linux.html](https://doc.qt.io/qtcreator/creator-deployment-embedded-linux.html)

[https://www.ics.com/blog/configuring-qt-creator-raspberry-pi](https://www.ics.com/blog/configuring-qt-creator-raspberry-pi)

add this to the .pro file

```bash
target.path = /home/pi/
INSTALLS += target
```

## Troubleshooting 
### Fine tune EGLFS

(official) [https://doc.qt.io/qt-5/embedded-linux.html](https://doc.qt.io/qt-5/embedded-linux.html)

 [https://forum.qt.io/topic/84827/qt5-eglfs-screen-resolutio](https://forum.qt.io/topic/84827/qt5-eglfs-screen-resolution)

```bash
QT_QPA_EGLFS_PHYSICAL_WIDTH = 
QT_QPA_EGLFS_PHYSICAL_HEIGHT = 
```

**IMPORTANT!!!** For correct scaling and dimensions remove any DPI scaling from the main.cpp (see below) 

```bash
QCoreApplication::setAttribute(Qt::AA_EnableHighDpiScaling);
```

### Error(s) related to DRM

**Could not queue DRM page flip on screen on RaspberryPi**

Could not queue DRM page flip on screen HDMI

[https://forums.raspberrypi.com/viewtopic.php?t=252614](https://forums.raspberrypi.com/viewtopic.php?t=252614)     // this explains for example that when X is running no other app can attach

[https://stackoverflow.com/questions/64731435/problem-when-deploying-qt-app-on-raspberry-pi-4-could-not-queue-drm-page-flip-o](https://stackoverflow.com/questions/64731435/problem-when-deploying-qt-app-on-raspberry-pi-4-could-not-queue-drm-page-flip-o)

[https://bugreports.qt.io/browse/QTBUG-72538?focusedCommentId=526636&page=com.atlassian.jira.plugin.system.issuetabpanels%3Acomment-tabpanel#comment-526636](https://bugreports.qt.io/browse/QTBUG-72538?focusedCommentId=526636&page=com.atlassian.jira.plugin.system.issuetabpanels%3Acomment-tabpanel#comment-526636)

[https://github.com/UvinduW/Cross-Compiling-Qt-for-Raspberry-Pi-4/issues/10](https://github.com/UvinduW/Cross-Compiling-Qt-for-Raspberry-Pi-4/issues/10)

```bash
# add this line to .profile file on RPi
QT_QPA_EGLFS_ALWAYS_SET_MODE=1

```

## Testing

- TODO: test hardware acceleration


## External Resources  

[QT5 Cross compile GUI with buildroot for raspberry pi3](https://youtu.be/5SnXJTwIunY)

[buildroot qt5 32bit mesa opengl on raspberry pi 3](https://www.youtube.com/watch?v=soergaJPy64)

[https://github.com/pbouda/buildroot-qt-dev/tree/master/config](https://github.com/pbouda/buildroot-qt-dev/tree/master/config)

[Cross-compile Qt applications for your Raspberry Pi 3 - 1. Install QtRpi from scratch](https://youtu.be/YYOjdwT5UuQ?list=PLFsidzAJDEbBr3l0BNMDcDQlUM04GRlf2)

[Cross-compile Qt applications for your Raspberry Pi 3 - 2. Configure Qt Creator](https://youtu.be/1d2bh7iUKNc?list=PLFsidzAJDEbBr3l0BNMDcDQlUM04GRlf2)

### About filesystem

have a look here to automate stuff even more [https://github.com/abhiTronix/rpi_rootfs](https://github.com/abhiTronix/rpi_rootfs)

### About toolchains

[https://github.com/Pro/raspi-toolchain/](https://github.com/Pro/raspi-toolchain/)

original - [https://www.kampis-elektroecke.de/raspberry-pi/qt/](https://www.kampis-elektroecke.de/raspberry-pi/qt/)

interelectronix (working till a certain steps but uses too old compiler)- [https://www.interelectronix.com/qt-on-the-raspberry-pi-4.html](https://www.interelectronix.com/qt-on-the-raspberry-pi-4.html)

uvinduw (another that is ALMOST working) - [https://github.com/UvinduW/Cross-Compiling-Qt-for-Raspberry-Pi-4](https://github.com/UvinduW/Cross-Compiling-Qt-for-Raspberry-Pi-4)

interelectronix - QtCreator (ok, ut doesn't show then how to start a project, which will fail of course) - [https://www.interelectronix.com/configuring-qt-creator-ubuntu-20-lts-cross-compilation.html](https://www.interelectronix.com/configuring-qt-creator-ubuntu-20-lts-cross-compilation.html)

neuralmotion qtrpi - [https://github.com/neuronalmotion/qtrpi](https://github.com/neuronalmotion/qtrpi)

mechatronicsblog - [https://mechatronicsblog.com/cross-compile-and-deploy-qt-5-12-for-raspberry-pi/](https://mechatronicsblog.com/cross-compile-and-deploy-qt-5-12-for-raspberry-pi/)

[https://www.youtube.com/watch?v=5SnXJTwIunY](https://www.youtube.com/watch?v=5SnXJTwIunY)

[https://medium.com/@au42/the-useful-raspberrypi-cross-compile-guide-ea56054de187](https://medium.com/@au42/the-useful-raspberrypi-cross-compile-guide-ea56054de187)

[https://github.com/Pro/raspi-toolchain]
