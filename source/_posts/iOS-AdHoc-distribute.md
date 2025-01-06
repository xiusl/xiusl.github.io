---
title: iOS 分发 Ad Hoc 应用
date: 2020-09-30 16:13:34
tags:
- iOS
- Ad Hoc
caegories:
- technology
---

公司内部测试 iOS App，之前一直用蒲公英分发，但现在各种平台都要重新认证，自己写一个网页简单分发下

<!-- more -->

打包好的 App，导出 Ad Hoc 的安装包

编写需要的 plist 文件，差不多下面这个样子 `install.plist`

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>items</key>
    <array>
        <dict>
            <key>assets</key>
            <array>
                <dict>
                    <key>kind</key>
                    <string>software-package</string>
                    <key>url</key>
                    <string>https://files.example.com/ipa/like.ipa</string>
                </dict>
            </array>
            <key>metadata</key>
            <dict>
                <key>bundle-identifier</key>
                <string>com.xiusl.like</string>
                <key>bundle-version</key>
                <string>1.1.2</string>
                <key>kind</key>
                <string>software</string>
                <key>releaseNotes</key>
                <string>测试</string>
                <key>title</key>
                <string>哩嗑</string>
            </dict>
        </dict>
    </array>
</dict>
</plist>
```

再写一个简单的网页 `index.html`
```
<!DOCTYPE>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>哩嗑 App</title>
</head>

<body>
    <div style="font-size:20px;text-align:center;margin-top:100px;">
        <a title="iPhone" href="itms-services://?action=download-manifest&url=https://image.sleen.top/ipa/install.plist">哩嗑内测下载</a>
        <p style="font-size:14px;color:#999;">Note: 内测包安装的设备需要在 Apple Developer 中注册</p>
    </div>
</body>
</html>
```

最后把 `install.plist`，`like.ipa`，`index.html`，上传到测试人员可以访问的地方，这里直接放在了七牛对象存储里，访问 `index.html` 就能安装喽。

