title: 解决 Mac OSX 无法识别 Android 设备
date: 2015-03-04 16:53
tags: Android
---
在 Mac 上开发 Android 需要真机调试的时候，却发现 Android 设备居然无法识别，这真是急的要跳起来。无论“抽插”多少次数据线，结果还都一样。

<!-- more -->

不要着急，下面就附上解决方案。

1. 第一步
找到自己设备的 ID。
About This Mac(关于本机) -> System Report(系统报告) -> USB-> 选择你所连接的Android 设备 --> Vendor ID (供应商ID)。可见，我的 Nexus 5 的 Vendor ID 为 `0x18d1` 。
![Vendor ID](http://upload-images.jianshu.io/upload_images/228805-f1c222e154b10e47.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

2. 第二步
在终端执行如下命令，命令中的 `YOUR_VENDOR_ID` 用你的 Android 设备 Vendor ID 代替, 这里可以代入`0x18d1` ：
```
echo YOUR_VENDOR_ID >> ~/.android/adb_usb.ini
```

3. 第三步
在终端中执行如下命令，发现已经可以识别到设备了。
```
adb start-server
adb devices
```
结果如下：

![识别到设备](http://upload-images.jianshu.io/upload_images/228805-397ba126cb84871c.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)
