---
title: 'HashMap, SparseArray, ArrayMap 三者的区别'
date: 2016-12-30 14:40:40
tags: Android
---

## HashMap

位于 JDK 中，作为开发中最常见的数据结构而广泛应用于各种场景中，但是效率和内存消耗却有欠佳，主要原因如下：

1. 因为它的 Key/Value 不支持 Java 中的基础类型，如果需要以 int 之类的基础类型作为 Key, 那么基础类型及其封装对象之间的转化将导致效率下降
2. 为了维护其内部链地址法的实现，减少 Key 的 Hash 值的冲突，HashMap 额外分配了 25% 的数组空间，而且数组的大小空间必须是 2 的指数
3. 大数据量执行效率高


## SparseArray

Android SDK 中提供类，适用于 Key/Value 为基础类型时

1. 优化了基础类型与其对应的封装类型之间的转化过程，执行效率高
2. 内部维护两个数组，一个数组用于存储 Key, 另一个数组用于存储 Value, 没有计算 Hash 的过程，使用二分查找 Key
3. 相对于 HashMap 内存消耗更少
4. 当内容很大时，效率会低于 HashMap, 因为查找需要使用二分查找，而且在数组中的添加、删除操作效率低下
5. 相对于 HashMap, 在几百这样的数量级，效率差异在 50% 以内
6. 对于元素的删除操作有优化，不会减少立即删除，而是将其标记为删除，但是储存空间保留，用于相同 Key 的重复利用

## ArrayMap

Android KitKat (API Level 19) 才引入 Android 平台，不过 Support 包已经支持了。

1. 内部维护两个数组，一个数组用于存储 Key 的 Hash 值， 另一个数组用户维存储 Key 及 Value，存储结构如下：

  ```
  mHashes[index] = hash;
  mArray[index<<1] = key;
  mArray[(index<<1)+1] = value;
  ``` 
2. 使用二分法查找Hash, 效率高
3. 内存消耗少于 HashMap
4. 当容量增长时，只需要拷贝数组中的内容即可，不需要重新建立 Hash Map
5. 当内容很大时，效率会低于 HashMap, 因为查找需要使用二分查找，而且在数组中的添加、删除操作效率低下
6. 当有内容减少时，内部没有 shrink 的机制，必须由开发者来处理
7. 如果不需要 Java 容器 API 的情况下(ArrayMap 实现了 Map 接口)， 可以使用 SimpleArrayMap 代替 ArrayMap