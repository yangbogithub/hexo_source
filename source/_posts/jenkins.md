---
title: Jenkins持续集成ios项目总结
date: 2017-02-10 14:26:48
tags: [Jenkins,ios]
categories: 技术
---

1.`添加插件｀Python plugin`可以运行`python`脚本。
2.调用`pod install`提示错误`ArgumentError - invalid byte sequence in US-ASCII`；
需要添加环境变量，在 `系统管理`->`系统设置`->`全局属性`->`Environment variables` 添加建值：`LC_ALL:en_US.UTF-8`
3.出现`command not found`可以使用命令全路径;例如pod命令的全路径为`usr/local/bin/pod`.
查询命令的全路径可以用命令`which [命令名]`,例如`which pod`.
4.`python`脚本编码问题可以在文件开始位置添加`# -*- coding: utf-8 -*-`。
5.单元测试报告可以用转成junit格式，这样就可以使用Jenkins的插件显示测试报告。
具体实现方式：
1) 安装`ocunit2junit`
```bash
gem install ocunit2junit
```
2) 单元测试命令
```bash
xcodebuild -workspace Demo.xcworkspace -scheme Demo_Tests -sdk iphonesimulator build-for-testing
xcodebuild -workspace Demo.xcworkspace -scheme Demo_Tests test -destination 'id=FC6447B8-C098-4DE2-A5AF-9CDE651507F2' 2>&1 | ocunit2junit
```
3) `Jenkins`构建配置中`构建后操作`添加`Publish Junit test report`,`测试报告(XML)`行输入`test-reports/*.xml`,即测试报告文件路径。

打包脚本：
```bash
#工程名
APP_NAME="项目名"
# 证书
CODE_SIGN_DISTRIBUTION="iPhone Distribution: xxx. (xxx)"
#info.plist路径
project_infoplist_path="./${APP_NAME}/Info.plist"
#取版本号
bundleShortVersion=$(/usr/libexec/PlistBuddy -c "print CFBundleShortVersionString" "${project_infoplist_path}")
#取build值
bundleVersion=$(/usr/libexec/PlistBuddy -c "print CFBundleVersion" "${project_infoplist_path}")
DATE="$(date +%Y%m%d_%H:%M:%S)"

#创建output目录
if [ ! -x output]; then  
　　mkdir output  
fi 

echo ${IPA_PATH}

TAG="${bundleShortVersion}_${bundleVersion}_$(date +%m%d%H%M%S)"

IPANAME="${APP_NAME}_${TAG}.ipa"
#要上传的ipa文件路径
IPA_PATH="output/${IPANAME}"
SCHEME="项目Scheme"

xcodebuild -workspace "${APP_NAME}.xcworkspace" -scheme "${SCHEME}" -configuration 'Release' clean
xcodebuild -workspace "${APP_NAME}.xcworkspace" -scheme "${SCHEME}" -sdk iphoneos -configuration 'Release' CODE_SIGN_IDENTITY="${CODE_SIGN_DISTRIBUTION}" SYMROOT='$(PWD)'
xcrun -sdk iphoneos PackageApplication "./Release-iphoneos/${SCHEME}.app" -o "${PWD}/${IPA_PATH}"


```
