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
$NDK_PATH/build/core/toolchains/arm-linux-androideabi-llvm/setup.mk  
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
修改:
$NDK_PATH/build/core/toolchains/setup-toolchain.mk  
把2改为3  
ifneq ($(words $(TARGET_TOOLCHAIN_LIST)),2)  
ifneq ($(words $(TARGET_TOOLCHAIN_LIST)),3)  
    $(call _ndk_error,Expected two items in TARGET_TOOLCHAIN_LIST, \  
        found "$(TARGET_TOOLCHAIN_LIST)")  
endif  
   
自己的安卓项目里面添加  
Application.mk  
NDK_TOOLCHAIN_VERSION := llvm  
需要处理的Android.mk  
android默认编译是导出所有符号的需要隐藏起来                  
# 如何隐藏C/C++编译生成的函数符号  
二进制文件中包含了代码中的字符串，在运行中这些字符串将被加载到内存中，被程序所采用。这些字符串如果不做特殊处理，那么通过一些反编译工具（如IDA Pro等）能全部看得到。  
## 回到函数符号的隐藏，一般可以在gcc编译选项中加入如下编译选项：  
-ffunction-sections, -fdata-sections会使compiler为每个function和data item分配独立的section。 --gc-sections会使ld删除没有被使用的section。
链接操作以section作为最小的处理单元，只要一个section中有某个符号被引用，该section就会被放入output中。
这些选项一起使用会从最终的输出文件中删除所有未被使用的function和data， 只包含用到的unction和data。 

    CFLAGS += -ffunction-sections -fdata-sections -fvisibility=hidden -mllvm -enable-strcry
    LDFLAGS += -Wl,--gc-sections
## 在xcode中，需要修改如下编译选项（加粗部分）：
    Strip Stype -> Non-Global Symbols  
    Use Separate Strip -> Yes
    Other Linker Flags -> -Xlinker -x
    Debug Information Level -> Line Tables only
    Generate Debug Symbols -> No
    Symbols Hidden by Default -> Yes
## 通过上述设置，库内部的函数符号就可以隐藏了。而我们需要一些导出函数给其他人用的话，可以在需要导出的函数前面加上：         
    __attribute__((visibility("default")))
