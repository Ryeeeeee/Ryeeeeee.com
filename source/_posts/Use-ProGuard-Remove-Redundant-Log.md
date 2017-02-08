title: 使用 ProGuard 移除无用输出语句
date: 2015-07-18 02:14
tags: Android
---
首先，来看一段这样的代码。

```
if (LogUtil.sEnable) {
	Log.d(TAG, "some output");
}
```

从接触 Android 开始，这样的代码我在各种不同的项目里都见过。这样做有什么不对的地方吗？并没有。可是有更好的解决方式吗？

**有**。

上述代码将应用的 Log 输出用一个开关控制，如果是 Debug 版本则开启输出。反之如果是 Released 版本，则关闭输出。

新的解决办法就是通过 `ProGuard` 这个工具。在所有 Android 应用中，都有一个配置文件 Proguard。通常，我们都使用这个文件混淆我们的代码。但是官方文档是这样说明的：

> The ProGuard tool shrinks, optimizes, and obfuscates your code by removing unused code and renaming classes, fields, and methods with semantically obscure names. The result is a smaller sized .apk file that is more difficult to reverse engineer. 

里面提到一点，说 `ProGuard` 可以移除无用的代码，在查阅了官方文档之后会发现，`-assumenosideeffects` 选项会在版本构建的时候就将指定的源码移除。

```
-assumenosideeffects class android.util.Log {
    public static *** d(...);
    public static *** i(...);
    public static *** v(...);
}
```

示例代码演示了在 Released 版本中，debug, info, verbose 三个等级的 Log 输出将被删除。通过自定义，不仅可以在 Log 控制输出等级，还能减小安装包的体积，简直一石二鸟！

## IMPORTANT
-------
需要注意的是，必须不能有关闭优化的配置`-dontoptimize`，否则上述方法将无效。
