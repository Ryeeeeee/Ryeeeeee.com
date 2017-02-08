title: Android WebView 中 addJavascriptInterface 接口无效问题
date: 2015-03-11 11:47
tags: Android
---

说点题外话吧，从 Cocos2d-x 技术支持部门调到 Cocos Play 部门已经不知觉半年了。半年中，大概一天都是当成两天来用，完全没有时间做点自己的事情。最近正统的 Andorid 代码已经很少写了，倒是 C++, Python 写的飞起，踩过的坑，流过的类也是不计其数。今天难得可以抽点时间写写 Android 代码，没想到又是一个坑，好吧，既然让我遇到了，就顺便记录下来吧。

<!-- more -->

Java 与 Javascript 交互
-------
最近遇到这样一个需求，Android 程序中需要使用 WebView 来加载一个网页，而这个网页会根据请求的参数正确与否返回不同的内容。当请求参数正确，则返回正常显示的 HTML 页面。如果请求参数错误，直接返回 json。

所以，我们必须通过 WebView 来对页面内容做出区分，如果是正常的页面内容则显示，反之，则解析其中的 json。我们知道，WebView 提供了一个接口，可以让我们注入 Java 对象到页面中，这样，页面中的 javascript 就能直接访问 Java 对象的接口，从而实现 Java 和 Javascript 的交互。

首先必须启用 WebView 中的 Javascript 支持。

```
webView.getSettings().setJavaScriptEnabled(true);
``` 

注入 Java 对象到 WebView 中。

```
webView.addJavascriptInterface(new JavascriptHandler(), "handler");
```

Java 对象定义如下，需要特别注意的是，在 JELLY_BEAN_MR1 之后，只有 public 且添加了 `@JavascriptInterface` 注解的方法才能被调用。这也是为了安全考虑。毕竟页面可以直接操作 Native Application 有点太可怕了。（没加注解，导致反射无效，查了好久也是醉了）

```
class JavascriptHandler {
    @JavascriptInterface
    public void getContent(String htmlContent){
        Log.i(TAG, "html content:" + htmlContent);
}

```

最后，在 WebView 载入完成后的回调方法中调用如下方法就能获得网页的内容了。

```
 @Override
public void onPageFinished(WebView view, String url) {
    view.loadUrl("javascript:window.handler.getContent(document.body.innerHTML);");
    super.onPageFinished(view, url);
}
```

IMPORTANT
-------
* Javascript 和 Java 的交互是在子线程中的，必须注意线程安全
* Java 对象中的成员方法不再能直接访问
* For applications targeted to API level LOLLIPOP and above, methods of injected Java objects are enumerable from JavaScript. （目前还不知道官方文档中的这句话代表了什么）
* 特别需要注意 Android 各个版本对 WebView 的要求导致的。越高的版本对 WebView 的限制更加严格。

References
-------
[http://developer.android.com/reference/android/webkit/WebView.html](http://developer.android.com/reference/android/webkit/WebView.html)
