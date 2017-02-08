title: Mac OSX 下手动更新 Android Studio
date: 2015-12-18 21:10
tags: Android
---
不知道为什么，我的 Android Studio 在翻墙状态还是总是无法自动更新，在屡次的重新下载几百兆的新包之后我终于忍无可忍，决定自己手动更新。最新版本测试可用（1.5.1 更新至 2.0）。

其实，知道规则之后，手动更新也是挺简单的。

首先，我们可以手动查询这个网址，这个网址列出了每个版本可用的 patch。在浏览器中打开它。
[https://dl.google.com/android/studio/patches/updates.xml](https://dl.google.com/android/studio/patches/updates.xml) 

这里截取最开头的一段内容。 
```
<products>
<product name="Android Studio">
<code>AI</code>
<channel feedback="https://code.google.com/p/android/issues/entry?template=Android+Studio+bug" id="AI-2-eap" majorVersion="2" name="Android Studio updates"status="eap" url="http://tools.android.com/recent">
<build number="143.2489090" version="2.0 Preview 4">
<message>
<![CDATA[
<html> A new Android Studio 2.0 Preview 4 is available in the canary channel.<p/> Canary builds are the bleeding edge, released about weekly. While these builds do get tested,<br/> they are still subject to bugs, as we want people to see what's new as soon as possible.<br/><p/> For slightly more predictable builds, use <b>Settings | Updates</b> and select the <b>Dev Channel</b>.</html>
]]>

</message>

<button download="true" name="Download" url="http://tools.android.com/download/studio/canary/latest/"/>
<button name="Release Notes" url="http://tools.android.com/recent"/>
<patch from="141.2422023" size="86"/>
<!-- 1.5.0.4 -->
<patch from="143.2443734" size="17"/>
<!-- 2.0.0.0 -->
<patch from="141.2456560" size="86"/>
<!-- 1.5.1.0 -->
<patch from="143.2461418" size="17"/>
<!-- 2.0.0.1 -->
<patch from="143.2469411" size="16"/>
<!-- 2.0.0.2 -->
<patch from="143.2479369" size="16"/>
<!-- 2.0.0.3 -->
</build>
</channel>
...
```

从这个 xml 文档我们可以知道，当前最新的版本号为 `143.2489090`, 即 `2.0 Preview 4`。

支持直接升级到这个版本低版本为  `141.2422023`, `143.2443734`,`141.2456560`, `143.2461418`, `143.2469411`, `143.2479369`。

同理，文档的后面只是列出其他版本的信息。有了这个信息，我们就可以拼出 patch 文件的下载路径了。（如果跨太多版本，就要需要通过下载多个jar包，先升级到中间版本，在升级到最新版本）

```
MacOS: AI-$FROM-$TO-patch-mac.jar
```

其中，`$FROM` 是更新前的版本， `$TO`是要更新到的版本。

所以，`141.2456560` 升级至 `143.2489090` 的 patch 下载地址为：
```
https://dl.google.com/android/studio/patches/AI-141.2456560-143.2489090-patch-mac.jar
```

下载完成之后，使用终端进入到 Android Studio 的 Bundle 中，输入如下命令。

```
java -classpath ./AI-141.2456560-143.2489090-patch-mac.jar com.intellij.updater.Runner install Contents/
```

**注意：很多老的博客中，命令不是以 `Content/`结尾，而是`.` , 经过测试，在较新的版本中并不适用，可能出现如下的错误信息**。

![](http://upload-images.jianshu.io/upload_images/228805-00b457bc7cfe3a7c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最后，重启 Android Studio 就是新版本了。

**一点点思考：**
1. 其实从安装过程就可以看出，其实更新程序和补丁内容包含在了下载的这个 jar 包里了。相比之下，想起自己之前做工具的时候，自动更新的代码是包含在前一个版本发布出去，这样不仅灵活性更低，兼容性不好，而且当发布出去的版本如果更新存在隐藏的bug, 那么自动更新功能可能就废了。涨姿势了。

2. 对于 patch 的制作，Google 也给出了很好的示范，每个最新版本只制作支持前6个版本的补丁。这样不仅可以避免跨多个版本更新，由于太多的增量补丁连续更新的出错概率，也不会因为为每个旧版本都制作直接升级到最新版本的补丁，而导致后期补丁数量爆炸。

最后，反编译 jar 包验证我的猜想。
![C6577155-78C8-4EEA-8342-00A3A621395F.png](http://upload-images.jianshu.io/upload_images/228805-ecdbd25e0ae0e31f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
