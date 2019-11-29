# Hikari_Xcode-NDK
Xcode OLLVM8.0 &amp;&amp; Android NDK r16b OLLVM6.0  

适用于Unity2018 && cocos2d-x-cocos2d-x-3.17.2测试通过。  
测试环境MAC10.14.5 Xcode11.2.1 NDKr16b  
  
Xcode直接用制作好的工具链OLLVM8.0  
NDK编译Unity需要替换llvm文件夹  
NDK编译COCOS需要修改toolchains  

制作NDKr16b OLLVM工具链  

修改Unity工具链:  
$NDK_PATH 为NDK路径  
下载OLLVM6.0解压  
rm -rf $NDK_PATH/toolchains/llvm/prebuilt/darwin-x86_64/*  
cp -r Hikari.xctoolchain/usr/* $NDK_PATH/toolchains/llvm/prebuilt/darwin-x86_64  
完成  
  
修改COCOS工具链:  
cp -r $NDK_PATH/toolchains/llvm $NDK_PATH/toolchains/arm-linux-androideabi-llvm  
cp -r $NDK_PATH/build/core/toolchains/arm-linux-androideabi-clang $NDK_PATH/build/core/toolchains/arm-linux-androideabi-llvm    
修改：  
arm-linux-androideabi-llvm/setup.mk  
---------------------------------- 修改前  
TOOLCHAIN_NAME := arm-linux-androideabi  
BINUTILS_ROOT := $(call get-binutils-root,$(NDK_ROOT),$(TOOLCHAIN_NAME))  
TOOLCHAIN_ROOT := $(call get-toolchain-root,$(TOOLCHAIN_NAME)-4.9)  
TOOLCHAIN_PREFIX := $(TOOLCHAIN_ROOT)/bin/$(TOOLCHAIN_NAME)-  
---------------------------------- 修改后  
TOOLCHAIN_NAME := arm-linux-androideabi  
BINUTILS_ROOT := $(call get-binutils-root,$(NDK_ROOT),$(TOOLCHAIN_NAME))  
TOOLCHAIN_ROOT := $(call get-toolchain-root,$(TOOLCHAIN_NAME)-llvm)  
TOOLCHAIN_PREFIX := $(TOOLCHAIN_ROOT)/bin/$(TOOLCHAIN_NAME)-  
  
自己的安卓项目里面添加  
Application.mk  
NDK_TOOLCHAIN_VERSION := llvm  
需要处理的Android.mk  
android默认编译是导出所有符号的需要隐藏起来  
# -fvisibility=hidden 添加隐藏导出符号  
# -mllvm -enable-strcry 加密字符串  
LOCAL_CFLAGS += -fvisibility=hidden \  
                -mllvm -enable-strcry  
