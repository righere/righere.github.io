---
title: 编译ffmpeg4Android
date: 2016-10-10 23:35:12
tags: [ffmpeg, android, 移植, 编译]
---

本文主要实现了FFmpeg的编译和移植，首先在linux下将官网下载的源码编译成.so文件，然后使用android-studio配合NDK工具，将.so文件移植到android项目当中，简单地介绍了如何一步步完成FFmpeg的编译流程

<!--more-->
## 1. 准备的编译工具

Git，NDK

安装git，检查本地git，`git --version`

直接用命令符安装： `sudo apt install git`

也可以去官网下载，[git官网下载](https://git-scm.com/downloads)

如果已经有了git，想更新到最新版，可以输入 `git clone https://github.com/git/git`，再进行编译安装


然后下载NDK(现在已经有13的版本了)，推荐使用android studio安装ndk，下载的ndk路径默认在AndroidSDK的ndk-boudle文件中

## 2. 配置NDK的环境
使用terminal配置电脑的环境,(个人电脑，我直接以管理员权限配置的系统环境变量)
> sudo gedit /etc/profile

打开之后，把我们的ndk路径配置进来，
```
	#Android NDK
	export ANDROID_NDK=/path/to/ndk(你的ndk解压路径)
	export PATH=$PATH:$ANDROID_NDK
```

## 3. 下载ffmpeg的最新的源码
新建一个ffempeg的工作文件夹，例如 `mkdir workplace`
> git clone git clone https://git.ffmpeg.org/ffmpeg.git

也可以直接去官网下载：[ffmpeg官网下载](https://ffmpeg.org/download.html)

## 4. 修改ffmpeg的configure文件
为了保证我们编译出的文件是以.so的后缀名
将文本中的3209-3212这4行：
```
SLIBNAME_WITH_MAJOR='$(SLIBNAME).$(LIBMAJOR)'
LIB_INSTALL_EXTRA_CMD='$$(RANLIB) "$(LIBDIR)/$(LIBNAME)"'
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_VERSION)'
SLIB_INSTALL_LINKS='$(SLIBNAME_WITH_MAJOR) $(SLIBNAME)'
```

修改改为：
```
SLIBNAME_WITH_MAJOR='$(SLIBPREF)$(FULLNAME)-$(LIBMAJOR)$(SLIBSUF)'
LIB_INSTALL_EXTRA_CMD='$$(RANLIB) "$(LIBDIR)/$(LIBNAME)"'
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_MAJOR)'
SLIB_INSTALL_LINKS='$(SLIBNAME)'
```
## 5. 编写编译脚本
写脚本的直观上的好处就是省去了一步步的执行编译命令，通常的在编译之前都需要进行配置，设置相应的环境变量，比如指定编译工具、编译平台等等，查看所有的配置选项可以在ffmpeg目录执行如下命令：
> ./configure --help

当然此脚本中，我们只需要设置几个我们比较关心的配置。

    1. NDK的路径：
        NDK=/path/to/ndk

    2. 编译的ndk plaform版本：
        SYSROOT=$NDK/platforms/android-23/arch-arm/

    3. 交叉编译工具:
        TOOLCHAIN=$NDK/toolchains/arm-linux-androideabi-4.9/prebuilt/linux-x86_64

    4. CPU平台：
        CPU=arm

    5. 编译完成后安装目录(当前目录下的android文件夹)：
        PREFIX=./android/$CPU

确定我们需要的配置之后，在ffmpeg的目录下面新建一个脚本 `build4android.sh`

```
	#!/bin/sh
	NDK=/path/to/ndk
	SYSROOT=$NDK/platforms/android-23/arch-arm/
	TOOLCHAIN=$NDK/toolchains/arm-linux-androideabi-4.9/prebuilt/linux-x86_64
	CPU=arm
	PREFIX=./android/$CPU
	ADDI_CFLAGS="-marm"
	./configure \
		--prefix=$PREFIX \
		--enable-shared \
		--disable-static \
		--disable-doc \
		--disable-ffmpeg \
		--disable-ffplay \
		--disable-ffprobe \
		--disable-ffserver \
		--disable-avdevice \
		--disable-doc \
		--disable-symver \
		--cross-prefix=$TOOLCHAIN/bin/arm-linux-androideabi- \
		--target-os=linux \
		--arch=arm \
		--enable-cross-compile \
		--sysroot=$SYSROOT \
		--extra-cflags="-Os -fpic $ADDI_CFLAGS" \
		--extra-ldflags="$ADDI_LDFLAGS" \
		$ADDITIONAL_CONFIGURE_FLAG
	make clean
	make
	make install
```
## 6. 开始编译
进入我们的FFmpeg的源码目录，执行刚才的脚本：
（先增加执行权限 `sudo chmod +x build4android.sh`,以防权限不够）
> ./build4anroid.sh

脚本执行中：![脚本执行中图片](http://of6x0sb2r.bkt.clouddn.com/buildffmpeg_1.png)

脚本执行完成:![脚本执行完成图图片](http://of6x0sb2r.bkt.clouddn.com/finishffmpeg.png)

生成的`android/arm/`目录中的文件：

![生成的.so文件图片](http://of6x0sb2r.bkt.clouddn.com/ffmpeglib.png)


## 7. 移植我们的.so文件

### 7.1 编写Android.mk文件

```
    
    LOCAL_PATH := $(call my-dir)
    
    
    include $(CLEAR_VARS)
    
    LOCAL_MODULE:= libavcodec
    
    LOCAL_SRC_FILES:= lib/libavcodec-57.so
    
    LOCAL_EXPORT_C_INCLUDES:= $(LOCAL_PATH)/include
    
    include $(PREBUILT_SHARED_LIBRARY)
    
    
    include $(CLEAR_VARS)
    
    LOCAL_MODULE:= libavformat
    
    LOCAL_SRC_FILES:= lib/libavformat-57.so
    
    LOCAL_EXPORT_C_INCLUDES:= $(LOCAL_PATH)/include
    
    include $(PREBUILT_SHARED_LIBRARY)
    
    
    
    include $(CLEAR_VARS)
    
    LOCAL_MODULE:= libswscale
    
    LOCAL_SRC_FILES:= lib/libswscale-4.so
    
    LOCAL_EXPORT_C_INCLUDES:= $(LOCAL_PATH)/include
    
    include $(PREBUILT_SHARED_LIBRARY)
    
    
    include $(CLEAR_VARS)
    
    LOCAL_MODULE:= libavutil
    
    LOCAL_SRC_FILES:= lib/libavutil-55.so
    
    LOCAL_EXPORT_C_INCLUDES:= $(LOCAL_PATH)/include
    
    include $(PREBUILT_SHARED_LIBRARY)
    
    
    include $(CLEAR_VARS)
    
    LOCAL_MODULE:= libavfilter
    
    LOCAL_SRC_FILES:= lib/libavfilter-6.so
    
    LOCAL_EXPORT_C_INCLUDES:= $(LOCAL_PATH)/include
    
    include $(PREBUILT_SHARED_LIBRARY)
    
    
    include $(CLEAR_VARS)
    
    LOCAL_MODULE:= libswresample
    
    LOCAL_SRC_FILES:= lib/libswresample-2.so
    
    LOCAL_EXPORT_C_INCLUDES:= $(LOCAL_PATH)/include
    
    include $(PREBUILT_SHARED_LIBRARY)
    
    
    
    include $(CLEAR_VARS)
    
    LOCAL_MODULE:= FFmpeg_codec
    
    LOCAL_SRC_FILES:= ffmpegndkbuild.c
    
    LOCAL_C_INCLUDES += $(LOCAL_PATH)/include 
    
    LOCAL_LDLIBS := -llog  -lz
    
    LOCAL_SHARED_LIBRARIES := avcodec avfilter avformat avutil  swresample swscale
    
    include $(BUILD_SHARED_LIBRARY)

```

### 7.2 编写Application.mk文件

```
    APP_ABI :=armeabi,armeabi-v7a
    APP_PLATFORM := android-23
    APP_OPTIM := release
    APP_STL := gnustl_static
```
### 7.3 编写C文件



### 7.4 ndk-build


