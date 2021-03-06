libdash on Android
==================
libdash is a library of DASH (Dynamic Adaptive Streaming over HTTP) client. libdash based on libcurl and libxml2 libraries.
This project is going to give the instructions for poring libdash on android. But we need to port libcurl and libxml2 at first.

## Environment:
* OS: Ubuntu 12.04 64-bit
* NDK verion: r9d

## Source Code Downloading
* libcurl download from [here](http://curl.haxx.se/download.html).
* libxml2 download by git `git clone git://git.gnome.org/libxml2`.
* libdash download by git `git clone git://github.com/bitmovin/libdash.git`.

First, we create android project by Eclipse (Ex: **${libdash_project}**).
Create libcurl, libxml2, and libdash folders under **${libdash_project}/jni/**.

### libcurl configuration
1. Set two environment variables **SYSROOT** and **CC**.

    ````
    export SYSROOT=$NDK/platforms/android-14/arch-arm
    export CC="$NDK/toolchains/arm-linux-androideabi-4.6/prebuilt/linux-x86_64/bin/arm-linux-androideabi-gcc --sysroot=$SYSROOT"
    ````
1. Run configuration with android toolchain.
 
    ````
    cd ${LIBCURL_SOURCE}/
    ./configure --with-sysroot=$SYSROOT --host=arm
    ````
1. Copy files into **${libdash_project}/jni/libcurl/**

    ````
    cp -r ${LIBCURL_SOURCE}/lib ${libdash_project}/jni/libcurl/
    cp -r ${LIBCURL_SOURCE}/src ${libdash_project}/jni/libcurl/
    cp -r ${LIBCURL_SOURCE}/include ${libdash_project}/jni/libcurl/
    ````
1. Copy **${LIBCURL_SOURCE}/packages/Android/Android.mk** to **${libdash_project}/jni/libcurl/**
1. Modify **${libdash_project}/jni/libcurl/[Android.mk](./libdash/jni/libcurl/Android.mk)**
    1. Set `LOCAL_PATH := $(call my-dir)` (line 52).
    1. Change `LOCAL_MODULE` (line 79) and `LOCAL_STATIC_LIBRARIES` (line 102) from `libcurl` to other name (Ex: `curl-library`).
    1. Add **$(LOCAL_PATH)/lib** to `LOCAL_C_INCLUDES` (line 73),
        
        i.e.,`LOCAL_C_INCLUDES += $(LOCAL_PATH)/include/ $(LOCAL_PATH)/lib`.
    1. Change last line `include $(BUILD_EXECUTABLE)` to `include $(BUILD_STATIC_LIBRARY)`.

### libxml2 configuration
1. Install tools for building libxml2

    ````
    sudo apt-get install autoconf libtool
    ````
1. Set two environment variables **SYSROOT** and **CC**.

    ````
    export SYSROOT=$NDK/platforms/android-14/arch-arm
    export CC="$NDK/toolchains/arm-linux-androideabi-4.6/prebuilt/linux-x86_64/bin/arm-linux-androideabi-gcc --sysroot=$SYSROOT"
    ````
1. Configure with Android toolchain
    
    ````
    ./autogen.sh
    ./configure --with-sysroot=$SYSROOT --host=arm
    ````
1. Copy files from **${LIBXML2_SOURCE}** to **${libdash_project}/jni/libxml2**

    ````
    find . -maxdepth 1 -name "*.[c|h]" -exec cp {} ${libdash_project}/jni/libxml2/ \;
    ````
1. Create [Android.mk](./libdash/jni/libxml2/Android.mk) for libxml2 under **${libdash_project}/jni/libxml2/**. This makefile very simple, just include all `*.c` and `*.h` files and build for **STATIC_LIBRARY**.

### libdash configuration
1. Copy files into **${libdash_project}/jni/libdash/**

    ````
    cp -r ${LIBDASH_SOURCE}/libdash/libdash/include ${libdash_project}/jni/libdash/
    cp -r ${LIBDASH_SOURCE}/libdash/libdash/source ${libdash_project}/jni/libdash/
    ````
1. Create [Android.mk](./libdash/jni/libdash/Android.mk) for libdash under **${libdash_project}/jni/libdash/**. This Android.mk just include `*.c` and `*.h` files for bulding.

There are the some files under **${LIBDASH_SOURCE}/libdash/qtsampleplayer**, such as libdashframework, and DASH manager. You can resue them if you need.

### Building shared library for Android
1. Create makefiles, [Android.mk](./libdash/jni/Android.mk) and [Application.mk](./libdash/jni/Application.mk) for ndk buiding.
    1. **[Android.mk](./libdash/jni/Android.mk)** only one line `include $(call all-subdir-makefiles)`. It means to find makefile for each subdirectory.
    1. **[Application.mk](./libdash/jni/Application.mk)** defined two variables `APP_STL` and `APP_ABI`.
1. Bulding by NDK tool.

    ````
    cd ${libdash_project}/
    ${NDK}/ndk-build
    ````
    After building, you will get the libdash.so.
    
### Examples for libcurl (Optional)
I create a folder for testing, called **[app](./libdash/jni/app)**. After building, you will get two shared libraries (i.e., .so file), `libdash.so` and `libapp.so`.
Reference to **${libdash_project}/jni/app/[libdash_networkpart_test.cpp](./libdash/jni/app/libdash_networkpart_test.cpp)**
