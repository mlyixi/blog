---
title: "Xcode6.1下ios8.1免证书真机调试"
date: 2014-08-30T18:08:52+08:00
tags: ["iOS"]
categories: ["OSX"]
toc: true
---

## iOS9开始已经允许开发者创建免费证书进行真机调试了.

一个必要条件是机器必需为[越狱](http://en.pangu.io/)且装有`appsync`,不然不能运行. 现在appsync for iOS 8 还是beta阶段,但也可以用了,具体见[官方说明](https://github.com/angelXwind/AppSync).

对于低版本的xcode其实有一个比较方便的软件:[JailCoder](http://oneiros.altervista.org/jailcoder/),可惜作者不更新了.

自己写了个软件[CodeSignBreak](https://github.com/mlyixi/CodeSignBreak),用兴趣的可以试试,也可以pull request.

* 制作证书iPhone Developer

保证Name为`iPhone Developer`, CertificationType为`CodeSigning`,然后`Let me override defaults`选中,其它默认Next就可以了.
 
* 修改SDKSettings.plist
/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS8.1.sdk/SDKSettings.plist中的`CODE_SIGNING_REQUIRED`和`ENTITLEMENTS_REQUIRED`的值为`NO`
 
* 修改Info.plist
/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Info.plist中添加`PROVISIONING_PROFILE_ALLOWED`和`PROVISIONING_PROFILE_REQUIRED` 值为 NO;查找`XCiPhoneOSCodeSignContext` 并替换成 `XCCodeSignContext`
 
* 自签名脚本
/Applications/Xcode.app/Contents/Developer/iphoneentitlements/gen_entitlements.py,权限777.
推荐用下面这种方法

```zsh
mkdir /Applications/Xcode.app/Contents/Developer/iphoneentitlements
cd /Applications/Xcode.app/Contents/Developer/iphoneentitlements
curl -O http://www.alexwhittemore.com/iphone/gen_entitlements.txt
mv gen_entitlements.txt gen_entitlements.py
chmod 777 gen_entitlements.py
```
[备份地址](http://img4blog.qiniudn.com/gen_entitlements.txt)

也可以手动(CV大法容易出错)

```python
#!/usr/bin/env python

import sys
import struct

if len(sys.argv) != 3:
	print "Usage: %s appname dest_file.xcent" % sys.argv[0]
	sys.exit(-1)

APPNAME = sys.argv[1]
DEST = sys.argv[2]

if not DEST.endswith('.xml') and not DEST.endswith('.xcent'):
	print "Dest must be .xml (for ldid) or .xcent (for codesign)"
	sys.exit(-1)

entitlements = """
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>application-identifier</key>
    <string>%s</string>
    <key>get-task-allow</key>
    <true/>
</dict>
</plist>
""" % APPNAME

f = open(DEST,'w')
if DEST.endswith('.xcent'):
	f.write("xfaxdex71x71")
	f.write(struct.pack('>L', len(entitlements) + 8))
f.write(entitlements)
f.close()
```

* 在Xcode中添加run script(BuildPhases下点菜单Editor然后addBuildPhases,灰色不可点?你先选中buildPhases中的一项就好了).
仍然推荐用下载的方法:[地址](http://img4blog.qiniudn.com/runscript.txt)

CV大法不推荐= =.

```zsh
export CODESIGN_ALLOCATE=/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/usr/bin/codesign_allocate
if [ "${PLATFORM_NAME}" == "iphoneos" ] || [ "${PLATFORM_NAME}" == "ipados" ]; then
/Applications/Xcode.app/Contents/Developer/iphoneentitlements/gen_entitlements.py "my.company.${PROJECT_NAME}" "${BUILT_PRODUCTS_DIR}/${WRAPPER_NAME}/${PROJECT_NAME}.xcent";
codesign -f -s "iPhone Developer" --entitlements "${BUILT_PRODUCTS_DIR}/${WRAPPER_NAME}/${PROJECT_NAME}.xcent" "${BUILT_PRODUCTS_DIR}/${WRAPPER_NAME}/"
fi
```

* 在Xcode中保证code signing identity为don’t code sign.

* 接下来的是适应iOS8

现在都是以前的步骤,ios7及之前的到这里已经可以了.但是如果是ios8,你一运行,发现`Code Signing Error`错误:No code signing identities found.

但是我明显配置了Don't Code Sign了啊,查看了错误日志(藏得有点深),发现正常target是过了的,但是还有一个target:tests出现错误,OK,那么就把它也设为Don't Code Sign,完工!