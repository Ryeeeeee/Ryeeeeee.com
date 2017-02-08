title: Android 中为自定义 View 的属性设置默认样式
date: 2016-11-22 00:52
tags: Android
---

我们自定义控件的时候，最常见的需要重写构造函数是 

```
public View(Context context) {}
public View(Context context, AttributeSet attrs) {}
```

因为第一个是在 Java 代码中创建 View 时会调用，第二是在 XML 布局文件中引用 View 时，会被调用。这两个构造函数都是系统会调用的方法。但是，还剩两个构造函数是需要我们自行调用的，系统并不会调用，当我们需要为自定义 View 设置默认的属性的时候，就需要用到了。


```
public View(Context context, AttributeSet attrs, int defStyleAttr) {}
public View(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {}
```
 

这是一段典型的一段自定义 View 的构造函数。

```
public class AttrTestTextView extends TextView {
    public AttrTestTextView(Context context) {
        super(context);
    }

    public AttrTestTextView(Context context, AttributeSet attrs) {
        this(context, attrs, R.attr.defaultStyleAttr);
    }

    public AttrTestTextView(Context context, AttributeSet attrs, int defStyleAttr) {
        this(context, attrs, defStyleAttr, R.style.DefaultStyleRes);
    }

    @TargetApi(Build.VERSION_CODES.LOLLIPOP)
    public AttrTestTextView(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {
        super(context, attrs, defStyleAttr, defStyleRes);

        TypedArray typedArray = context.obtainStyledAttributes(attrs, R.styleable.AttrTestTextView,
                defStyleAttr, defStyleRes);

        ...

        typedArray.recycle();
    }
}
```

可以看出来，属性的获取主要由 `context.obtainStyledAttributes` 决定，而最后两个参数`defStyleAttr `和`defStyleRes `决定了最终默认的属性样式是怎么的。

为了弄清楚默认样式的运作规律，我们先为这个自定义 View 创建 5 个自定义属性，和一个单独的属性，`attrs.xml` 文件如下：

```
<resources>
    <declare-styleable name="AttrTestTextView">
        <attr name="firstAttr" format="string" />
        <attr name="secondAttr" format="string" />
        <attr name="thirdAttr" format="string" />
        <attr name="fourthAttr" format="string" />
        <attr name="fifthAttr" format="string" />
    </declare-styleable>

    <attr name="defaultStyleAttr" format="reference" />
</resources
```

设置 `5` 个属性的原因是 Android 的 View 属性覆盖的优先级有 `5` 处，分别是：
1. 布局文件中的直接指定的属性
2. 布局文件中的 style 中指定的属性
3. 我们自己定义的 Style 
4. Theme 中直接指定的 Style 引用
5. Theme 中直接指定的属性

他们之间的覆盖的优先级是：

```
1 > 2 > 3 / 4 > 5
```

为了验证我们的结论，首先在Application 或 Activity 的 Theme 中直接设置这5个属性，并且为上方定义的 `defaultStyleAttr` 属性设置一个style的引用，值为下方定义的 `@style/DefaultStyleInAppTheme`, 而这个属性将作为自定义View构造函数中的第三个值，即该自定义 View 默认的 style。`DefaultStyleRes` 则作为自定义 View 的构造函数中的第四个参数，而 `LayoutStyle` 则在布局中直接引用， `styles.xml` 文件如下：

```
<resources>

    <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <item name="firstAttr">firstAttr from theme attr</item>
        <item name="secondAttr">secondAttr from theme attr</item>
        <item name="thirdAttr">thirdAttr from theme attr</item>
        <item name="fourthAttr">fourthAttr from theme attr</item>
        <item name="fifthAttr">fifthAttr from theme attr</item>

        <item name="defaultStyleAttr">@style/DefaultStyleInAppTheme</item>
    </style>

    <style name="DefaultStyleInAppTheme">
        <item name="firstAttr">firstAttr from theme style</item>
        <item name="secondAttr">secondAttr from theme style</item>
        <item name="thirdAttr">thirdAttr from theme style</item>
    </style>

    <style name="DefaultStyleRes">
        <item name="firstAttr">firstAttr from style res</item>
        <item name="secondAttr">secondAttr from style res</item>
        <item name="thirdAttr">thirdAttr from style res</item>
        <item name="fourthAttr">fourthAttr from style res</item>
    </style>

    <style name="LayoutStyle">
        <item name="firstAttr">firstAttr from layout style</item>
        <item name="secondAttr">secondAttr from layout style</item>
    </style>

</resources
```

然后我们自定义一个 TextView， 分别获取这 5 个属性的值，并且显示出来。

```
public class AttrTestTextView extends TextView {
    public AttrTestTextView(Context context) {
        super(context);
    }

    public AttrTestTextView(Context context, AttributeSet attrs) {
        this(context, attrs, R.attr.defaultStyleAttr);
    }

    public AttrTestTextView(Context context, AttributeSet attrs, int defStyleAttr) {
        this(context, attrs, defStyleAttr, R.style.DefaultStyleRes);
    }

    @TargetApi(Build.VERSION_CODES.LOLLIPOP)
    public AttrTestTextView(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {
        super(context, attrs, defStyleAttr, defStyleRes);

        TypedArray typedArray = context.obtainStyledAttributes(attrs, R.styleable.AttrTestTextView,
                defStyleAttr, defStyleRes);

        StringBuilder sb = new StringBuilder();
        sb.append(typedArray.getString(R.styleable.AttrTestTextView_firstAttr)).append("\n");
        sb.append(typedArray.getString(R.styleable.AttrTestTextView_secondAttr)).append("\n");
        sb.append(typedArray.getString(R.styleable.AttrTestTextView_thirdAttr)).append("\n");
        sb.append(typedArray.getString(R.styleable.AttrTestTextView_fourthAttr)).append("\n");
        sb.append(typedArray.getString(R.styleable.AttrTestTextView_fifthAttr));

        setText(sb);

        typedArray.recycle();
    }
}
```

我们知道，属性的获取最终是根据 `obtainStyledAttributes()` 方法获得，他的最后两个参数也就是构造行数的最后两个参数，这两个参数都是设置自定义 View 的默认样式，那么，他们作用的优先级是怎样的？

```
* @param defStyleRes A resource identifier of a style resource that
*        supplies default values for the view, used only if
*        defStyleAttr is 0 or can not be found in the theme. Can be 0
*        to not look for defaults.
```

根据 API 文档的可以看出，当 `defStyleAttr` 参数被设置为 `0`， 或者在 Application 和 Activity 的 Theme 中找不到 `defStyleAttr` 属性的赋值（例如 styles.xml 中的 `<item name="defaultStyleAttr">@style/DefaultStyleInAppTheme</item>` 这行代码被删除），那么，这个 View 的默认样式就是 `defStyleRes` 参数所引用的属性样式。可以看出这两个参数是冲突的，而且冲突级别是整个 style, 即 `defStyleAttr` 参数的 style 能找到情况下就忽略了整个 `defStyleRes` 的属性集。

然后，在布局文件中添加自定义的 View，分别设置了直接的属性，`firstAttr` 和 设置了 `styles.xml` 中定义的 style。
```
<RelativeLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:id="@+id/activity_main"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

    <com.ryeeeeee.attrtest.AttrTestTextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            app:firstAttr="firstAttr from layout attr"
            style="@style/LayoutStyle"/>

</RelativeLayout>
```

运行结果如下图，可以看到 `defStyleRes` 所引用的 style 中的属性完全被忽略了，因为 `styles.xml` 中能找到 `defStyleAttr` 这个属性的赋值：

![WechatIMG1.png](http://upload-images.jianshu.io/upload_images/228805-bb1770d5ab10ade3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

现在将 `styles.xml` 中能找到 `defStyleAttr` 这个属性的赋值删除，或者将 `defStyleAttr` 参数使用 `0` 赋值, 运行结果如下，可以看到 `defStyleRes` 所引用的style 才显示出来：

![WechatIMG2.png](http://upload-images.jianshu.io/upload_images/228805-d265e54c01e2e25d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

实验结果证明，属性层级的覆盖优先级是 `1 > 2 > 3 / 4 > 5`，和上方结论一致。需要注意的是，在 style 层级的优先级 `defStyleAttr` 参数所引用的 style 的优先级是高于 `defStyleRes` 的。











