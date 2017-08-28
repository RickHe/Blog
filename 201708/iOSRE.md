### iOS 10.0.1使用 dumpdecrypted 砸壳

##### iOS 应用逆向工程这本书中讲到的砸壳方法，在 iOS10以后会报错dyld: could not load inserted library 'dumpdecrypted.dylib' because no suitable image found.  Did find: dumpdecrypted.dylib: required code signature missing for 'dumpdecrypted.dylib' Abort trap: 6，分享下解决该问题的思路！
* 从 github 上下载 [dumpdecrypted源码](https://github.com/stefanesser/dumpdecrypted)
* 编译 dumpdecrypted.dylib， 进入 dumpdecrypted 目录运行 make命令，会生成一个dumpdecrypted.dylib文件。
```
DaFenQI@MrZz:~/Desktop/dumpdecrypted% make
`xcrun --sdk iphoneos --find gcc` -Os  -Wimplicit -isysroot `xcrun --sdk iphoneos --show-sdk-path` -F`xcrun --sdk iphoneos --show-sdk-path`/System/Library/Frameworks -F`xcrun --sdk iphoneos --show-sdk-path`/System/Library/PrivateFrameworks -arch armv7 -arch armv7s -arch arm64 -c -o dumpdecrypted.o dumpdecrypted.c 
2017-08-28 10:36:39.838 xcodebuild[3839:38885] [MT] PluginLoading: Required plug-in compatibility UUID DFFB3951-EB0A-4C09-9DAC-5F2D28CC839C for plug-in at path '~/Library/Application Support/Developer/Shared/Xcode/Plug-ins/ClangFormat.xcplugin' not present in DVTPlugInCompatibilityUUIDs
2017-08-28 10:36:40.852 xcodebuild[3845:38914] [MT] PluginLoading: Required plug-in compatibility UUID DFFB3951-EB0A-4C09-9DAC-5F2D28CC839C for plug-in at path '~/Library/Application Support/Developer/Shared/Xcode/Plug-ins/ClangFormat.xcplugin' not present in DVTPlugInCompatibilityUUIDs
`xcrun --sdk iphoneos --find gcc` -Os  -Wimplicit -isysroot `xcrun --sdk iphoneos --show-sdk-path` -F`xcrun --sdk iphoneos --show-sdk-path`/System/Library/Frameworks -F`xcrun --sdk iphoneos --show-sdk-path`/System/Library/PrivateFrameworks -arch armv7 -arch armv7s -arch arm64 -dynamiclib -o dumpdecrypted.dylib dumpdecrypted.o
```
* 将dumpdecrypted.dylib文件拷贝到需要砸壳应用的沙盒Document目录，使用 ifunbox 选中你要的应用程序红框部分为对应沙盒目录
![image.png](http://upload-images.jianshu.io/upload_images/1192292-671cdcd20833360c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* ssh 到 iOS 设备
使用 pp 助手工具设备工具打开 ssh 通道
打开命令行输入ssh root@localhost -p 2222提示的语句登录，默认密码为alpine
登录成功后使用 ps-e 命令查看设备中运行的进程找到可执行文件的路径/var/mobile/Containers/Bundle/Application/XXXXXXXX- XXXX-XXXX-XXXX- XXXXXXXXXXXX/TargetApp.app/
![image.png](http://upload-images.jianshu.io/upload_images/1192292-19682a0302f6e247.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
DaFenQI@MrZz:~/Desktop/dumpdecrypted% ssh root@localhost -p 2222
root@localhost's password: 
DaFenQiiPad:~ root# 
```

* 开始砸壳
```
DYLD_INSERT_LIBRARIES=/path/to/dumpdecrypted.dylib /path/to/executable
```
* 报错
```
DaFenQI@MrZz:~ root# cd /var/mobile/Containers/Data/Application/E2ABB23B-EC66-4DA4-AD3E-E14E20D680B5/Documents
DaFenQI@MrZz:/var/mobile/Containers/Data/Application/E2ABB23B-EC66-4DA4-AD3E-E14E20D680B5/Documents root# DYLD_INSERT_LIBRARIES=dumpdecrypted.dylib "/var/containers/Bundle/Application/46316B03-5DC3-4534-8D40-A29FE9315E22/WeChat.app/WeChat"
dyld: could not load inserted library 'dumpdecrypted.dylib' because no suitable image found.  Did find:
	dumpdecrypted.dylib: required code signature missing for 'dumpdecrypted.dylib'
Abort trap: 6
```
* 解决思路：
google iOS10 dumpdecrypted
结合报错信息
应该是dumpdecrypted.dylib未签名
解决方案使用 ldid 工具的ldid -S dumpdecrypted.dylib命令给dumpdecrypted.dylib签名
然后运行砸壳命令
```
DaFenQiiPad:/root# ldid -S private/var/mobile/Containers/Data/Application/FC66F79B-E67C-4746-845E-AB20778AA036/Documents/dumpdecrypted.dylib
DaFenQiiPad:/root# DYLD_INSERT_LIBRARIES=/private/var/mobile/Containers/Data/Application/FC66F79B-E67C-4746-845E-AB20778AA036/Documents/dumpdecrypted.dylib /var/containers/Bundle/Application/B9DBC5DA-4F84-46E6-8C8D-3BA68CD32AB9/EE_VKO.app/EE_VKO
objc[6582]: Class SSKeychain is implemented in both /System/Library/PrivateFrameworks/StoreServices.framework/StoreServices (0x1acede2b0) and /var/containers/Bundle/Application/B9DBC5DA-4F84-46E6-8C8D-3BA68CD32AB9/EE_VKO.app/EE_VKO (0x101355768). One of the two will be used. Which one is undefined.
mach-o decryption dumper

DISCLAIMER: This tool is only meant for security research purposes, not for application crackers.

[+] detected 64bit ARM binary in memory.
[+] offset to cryptid found: @0x1000ccc58(from 0x1000cc000) = c58
[+] Found encrypted data at address 00004000 of length 15843328 bytes - type 1.
[+] Opening /private/var/containers/Bundle/Application/B9DBC5DA-4F84-46E6-8C8D-3BA68CD32AB9/EE_VKO.app/EE_VKO for reading.
[+] Reading header
[+] Detecting header type
[+] Executable is a plain MACH-O image
[+] Opening EE_VKO.decrypted for writing.
[+] Copying the not encrypted start of the file
[+] Dumping the decrypted data into the file
[+] Copying the not encrypted remainder of the file
[+] Setting the LC_ENCRYPTION_INFO->cryptid to 0 at offset c58
[+] Closing original file
[+] Closing dump file
```
* 成功
