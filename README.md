# qt-4.8.6-cross-compile-for-beaglebone-black

1. Introduction

To get Qt application running on embedded linux development environment it is necessary to build Qt source using compatible Cross compiler/ Toolchain.

The Qt source should be build in the host environment such that the application generated with it will be able to execute in the embedded environment. For this Cross compiler/ Toolchain play a major role and hence a working Cross compiler/ Toolchain is required.

Requirement: Build Qt for cortex arm-8

Resources used in this discussion:

    Host machine: Ubuntu 12.04, LTS, 32-bit
    Qt source: qt-everywhere-opensource-src-4.8.6
    Cross complier/Toolchain: gcc-linaro-arm-linux-gnueabihf-4.7-2013.04-20130415_linux

Device Used: Beaglebone black with Ubuntu 13.04 running on it

Qt creator: Version 2.7.0 based on Qt5.0.2(32-bit), used for developing the Qt application

         2. Toolchain Extraction and Installation

2.1 Extract the Toolchain using the following commands:

unxz gcc-linaro-arm-linux-gnueabihf-4.7-2013.04-20130415_linux.tar.xz

tar -xvf gcc-linaro-arm-linux-gnueabihf-4.7-2013.04-20130415_linux.tar

2.2 Copy the extracted Toolchain contents into location /usr/local/linaro (a sample location, it can be copied to any other location also)

cp gcc-linaro-arm-linux-gnueabihf-4.7-2013.04-20130415_linux/ /usr/local/linaro 

Thus the Toolchain is ready to use for building the Qt source.

3. Pre-requisitesfor Qt Build

The Qt source which is downloaded from the official Qt website needs to be extracted.

gunzip qt-everywhere-opensource-src-4.8.6.tar.gz

tar -xvf qt-everywhere-opensource-src-4.8.6.tar

A new set of mkspecs needs to be created that tells the qmake which Toolchain it needs to reference when Makefiles are generated. Qt has some example mkspecs that can be directly used if the Toolchain being used matches with any of those else a custom mkspecs needs to be created with appropriate naming convention. The example mkspecs are present inside the directory mkspecs/qws.

The mkspecs consists of 2 files:

    qmake.conf: This is a list of qmake variable assignments that tells qmake what flags to pass through to the compiler, which compiler to use etc.
    qplatformdefs.h: This is a header file with various platform specific #includes and #defines. Often this refers to an existing qplatformdefs.h file from another generic mkspec.

Steps to be followed for configuring and building the Qt source are as follows:

cd qt-everywhere-opensource-src-4.8.6/mkspecs/qws

mkdir linux-arm-linaro-g++

cp linux-arm-g++/qmake.conf linux-arm-linaro-g++

cp linux-arm-g++/qplatformdefs.h linux-arm-linaro-g++

According to the Toolchain used make the following changes in the qmake.conf file.

#

# qmake configuration for building with linux-arm-linaro-g++

#

include(../../common/linux.conf)

include(../../common/gcc-base-unix.conf)

include(../../common/g++-unix.conf)

include(../../common/qws.conf)

#Compiler Flags to take advantage of the ARM architecture

QMAKE_CFLAGS_RELEASE = -O3 -march=armv7-a -mtune=cortex-a8 -mfpu=neon -mfloat-abi=hard

QMAKE_CXXFLAGS_RELEASE = -O3 -march=armv7-a -mtune=cortex-a8 -mfpu=neon -mfloat-abi=hard

# modifications to g++.conf

QMAKE_CC = /usr/local/linaro/bin/arm-linux-gnueabihf-gcc (The path depends where the Toolchain has been installed)

QMAKE_CXX= /usr/local/linaro/bin/arm-linux-gnueabihf-g++

QMAKE_LINK= /usr/local/linaro/bin/arm-linux-gnueabihf-g++

QMAKE_LINK_SHLIB = /usr/local/linaro/bin/arm-linux-gnueabihf-g++

# modifications to linux.conf

QMAKE_AR = /usr/local/linaro/bin/arm-linux-gnueabihf-ar cqs

QMAKE_OBJCOPY = /usr/local/linaro/bin/arm-linux-gnueabihf-objcopy

QMAKE_STRIP = /usr/local/linaro/bin/arm-linux-gnueabihf-strip

load(qt_config)

Note: -mfloat-abi=hard or -mfloat-abi=softfp depends on the type of Toolchain being used.

The qplatformdefs.h file refers to existing generic one.

#include “../../linux-g++/qplatformdefs.h”

4. Configuring Qt for Embedded Environment

Configure Qt such that it uses the custom mkspecs and hence the Toolchain. Run the following command to configure the Qt:

./configure -prefix /usr/local/Qt4.8.6 -embedded arm -platform qws/linux-x86-g++ -xplatform qws/linux-arm-linaro-g++ -static -no-mmx -no-rpath -no-3dnow -no-sse -no-sse2 -no-glib -no-cups -no-largefile -no-accessibility -no-openssl -no-gtkstyle -little-endian -qt-gfx-linuxfb -fontconfig -no-sql-mysql -no-sql-odbc -no-sql-psql -no-sql-sqlite -no-sql-sqlite2 -no-webkit -no-qt3support -nomake examples -nomake demos -nomake docs -nomake translations -qt-freetype -qt-libjpeg -qt-libpng -qt-mouse-tslib -qt-kbd-tty -qt-kbd-linuxinput -qt-mouse-tslib -qt-mouse-linuxinput

Some of the options brief explanation is as below:

    -prefix <path> – The host build of Qt-Embedded will be installed under this prefix
    -embedded – Enable support for the embedded options. This includes the QWS window system into the build and enables so other embedded related options
    -xplatform <mkspecs file to use> – Cross compile for the target platform using the environment specified in the mkspec file
    -static – Build the Qt libraries as static.
    -qt-gfx-linuxfb – Enable linux framebuffer. This option is used so that the application uses the linux framebuffer instead of direct framebuffer or X
    -qt-kbd-tty – Enable support for the tty keyboard driver 
    -qt-kbd-linuxinput – Enable a keyboard linux input in the QtGui library
    -qt-mouse-linuxinput – Enable a mouse linux input in the QtGui library
    -qt-mouse-tslib – Enable touch screen support via the generic tslib touch screen library

5. Build and Install

To build and install the cross compiled version of Qt follow the below instructions:

make

sudo make install (as Qt4.8.6 is been installing in /usr/local it needs root permissions)

Note: make confclean – Clean the previous builds i.e. remove all build files and the previous configurations.

Thus the cross compiled to Qt will be successfully installed in the directory /usr/local/Qt4.8.6.

6. Transferring the Libraries onto the Board

The cross compiled Qt libraries need to be transferred onto the target embedded environment (i.e Beaglebone black in this discussion). Steps needed to be followed are as given below.

From host machine:

Enter into the installed Qt directory, generate the fontdir and fontscale which is required for the Qt application to find the fonts and transfer the libraries using scp transfer. 

cd /usr/local/Qt4.8.6/lib/fonts

sudo mkfontscale

sudo mkfontdir

cd /usr/local/Qt4.8.6

scp -r lib @ip_address:(required path)

In target environment:

The transferred libraries must be copied to the place as same as that in the host machine (It’s not mandatory).

sudo cp -r lib /usr/local/Qt4.8.6

7. Qt Creator Build Changes

To run the Qt application in embedded environment which is developed using the Qt creator it is necessary to make changes in the Qt creator build options. Step needed to be followed is as below.

7.1 In Qt creator, the following steps need to be followed for selection of device type:

7.1.1 Tools -> Options -> Devices (For selecting device to deploy application)

Add -> Generic Linux Devices -> start wizard -> Enter all the fields -> Finish -> If connection is successful then the testexit stating Device test successful 

7.2 Tools -> Options -> Build & Run: 

7.2.1 Compilers -> Add -> GCC —> Name — Linaro_GCC —> Compiler path -> browse — /usr/local/linaro/bin/arm-linux-gnueabihf-g++ —> ABI -> arm-linux-generic-elf-32bit

7.2.2 Qt Versions -> Add -> browse — /usr/local/Qt4.8.6/bin/qmake

7.2.3 Kits -> Name — Linaro Device —> Device type -> Generic Linux Devices —> Device -> Select the device (as stated in 7.1.1) —> Compiler -> Select Linaro_GCC —> Debugger -> Edit -> browse — /usr/local/linaro/bin/arm-linux-gnueabihf-gdb —> Qt version -> Select the version (as stated in 7.2.2) —> Apply —> Ok

7.3 Project settings is as follows:

7.3.1 Project -> Add kit -> Select Linaro Device

7.3.2 Linaro Device -> Build -> build environment -> set LD_LIBRARY_PATH to /usr/local/Qt4.8.6/lib

7.3.3 Linaro Device –> Run -> Arguments -> -qws –> System environment –> set QTDIR to /usr/local/Qt4.8.6 —> set QT_QWS_FONTDIR to /usr/local/Qt4.8.6/lib/fonts

7.4 Following lines of code needs to be added in the .pro file of the Qt project.

target.files = Test (Project name)

target.path = (deployment path on the target environment)

INSTALLS += target

8. Execution of Binary on the Target

Once the Qt applications are build successfully, press on the Run button in the Qt creator so that it deploys the binary on to the target and executes the application.

9. Additional Information

9.1 Another Toolchain that can be used when Beaglebone Black is being used, i.e. angstrom-2011.03-i686-linux-arm7a-linux-gnueabi-toolchain-qte-4.6.3 Toolchain. But the problem faced with this Toolchain was that no texts were visible when application run on the target though the fonts were available in the respective directory and the paths has been set.

9.2 Some of the additional environmental variables that can be set are as follows :

 QWS_DISPLAY=LinuxFB:mmWidth=(width):mmHeight=(height)

 QWS_MOUSE_PROTO=tslib:/dev/input/event0

 QWS_KEYBOARD=LinuxInput:/dev/keyboard

9.3 While running the Qt application in the target environment we can use the following necessary options as given below:

    -fn<font> – Defines application font
    -bg<color> – Sets the default background color
    -btn<color> – Sets default button color
    -fg<color> – Sets default foreground color
    -geometry <width>+<height>+<Xoffset>+<Yoffset> – Sets the client geometry of the first window that is shown
    -keyboard – Enables keyboard
    -nokeyboard – Disables the keyboard
    -mouse – Enables the mouse cursor
    -nomouse – Disables the mouse cursor
    -qws – Runs the application as server application
    -display – Specifies the screen driver

10. Issues Faced

As I told earlier when I started the process of cross compilation I was totally newbie and in this process I faced many difficulties. Though they are not such a big issues still I would like to share these as some might get advantage seeing these when they find the same issues.

10.1 At first I had configured Qt source without stating that either it should use Linux framebuffer or Direct framebuffer or X. So the application to run it searches in a sequence Linux framebuffer –> Direct framebuffer –> X. So in my case when the Qt application runs it was searching for X server. But the OS which was installed on the target board did not had X server. So its necessary to state at the time of Qt source configuration to use Linux framebuffer by giving an option — -qt-gfx-linuxfb.

10.2 Sometimes when application runs on the target board, following error may pop up:

bash: ./Test -qws: No such file or directory

Type the following command and find the link missing:

readelf -l Test: displayed that there was link missing for ld-linux-so.3 (in my case)

cd /lib

sudo ln -f /lib/ld-linux-armhf.so.3 /lib/ld-linux.so.3

10.3 Another possible case may be it cannot find the EGL libraries. 

./Test -qws error while loadong shared libraries: libEGL.so.1 cannot open shared object files: No such file or directory

export LD_LIBRARY_PATH=/usr/local/arm-linux-gnueabihf/mesa-egl (path needs to set to resolve the above error).

10.4 If any font issue is being faced, it’s necessary to configure the Qt source by adding options like –fontconfig and -qt-freetype so that it uses the Free Type fonts.

11. Websites referred

For Qt source download: 

    http://qt-project.org/downloads

For Toolchain download:

    https://releases.linaro.org/13.04/components/toolchain/binaries
    http://web.archive.org/web/20130823131954/http://www.angstrom-distribution.org/toolchains/

References for build process: 

    http://processors.wiki.ti.com/index.php/Building_Qt
    https://qt-project.org/wiki/Building_Qt_for_Embedded_Linux
    http://qt-project.org/doc/qt-4.8/qt-embedded-install.html
    http://qt-project.org/doc/qt-4.8/qt-embedded-crosscompiling.html
    http://derekmolloy.ie/beaglebone/qt-with-embedded-linux-on-the-beaglebone/
    http://www.cloud-rocket.com/2013/07/building-qt-for-beaglebone/
    http://treyweaver.blogspot.com.es/2010/10/setting-up-qt-development-environment.html
