title: 一个我看了就想马上用起来的 Gitflow
date: 2015-07-21 22:28:20
tags: Git
---
前几天在网上看到一篇关于 Git 分支模型的推荐，作者亲身实践并告诉我们这是一个成功的模型。当时，自己正苦于项目混乱的分支管理，当我看完这篇文章的时候，心中豁然开朗，相见恨晚大概就是这样的感觉吧。

> 原文链接：http://nvie.com/posts/a-successful-git-branching-model/

我不觉得我会描述得比原文更好，所以这里只是做个总结，我还是推荐大家去看原文。

![分支模型](http://upload-images.jianshu.io/upload_images/228805-2e46113a86c5c159.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

* **master**: 主分支，也是对外的默认分支，当前分支上的所有版本都是可以直接发布的，并且需要使用 tag 记录所有版本信息。
* **develop**: 开发分支，所有的开发任务都应在此分支或从此分支 checkout 的分支中进行，且不会被其他分支合并，即使是 master 分支。
* **feature**: 特性开发分支，当程序需要开发一个新的特性的时候，应从 develop checkout 一个特性分支，在特性开发完毕之后需要被合并到 develop 分支，且远端不可存在此分支。
* **release**: 发布分支，当程序近乎可发布状态的时候，从 develop 分支 checkout ，并且在此分支处理一些例如版本号修改等各发布相关的修改。在此分支切不可开发新功能，但可以进行一些发布必要的小修补。完成后需要被合并到 master 分支和 develop 分支。
* **hotfix**: 紧急修复分支，这个分支可能是你最不想看到的分支。因为当一个新的版本发布了之后，可能会面临一些比较重大的 bug，而又等不到下一个版本开发完成之后再进行修复，此时就需要从 master 分支 checkout 一个 hotfix 分支出来，在这个分支进行修复，完成之后需要被 master 和 develop 分支合并。


