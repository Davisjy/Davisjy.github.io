---
title: WKWebview的bug
date: 2019-04-10 14:17:52
tags: wkwebview
categories: iOS
---

### 背景
用webview打开一个h5，比如长按图片时弹出系统alertVC，平时没有发现什么问题，但如果此时你webview控制器是present呈现的话，会发现奇怪的bug。alertVC__dismiss__时，控制器也会奇怪的跟着dismiss。当时看到这个情况真的是震惊了，打断点会发现调用两次dismiss(网上查到的很多是多次，不知道是如何复现的)，查看调用堆栈觉得是__WKActionSheet__搞的鬼。<!--more-->

### 解决过程
#### 问题确定
> 首先确定的是，这是一个bug，官方也已经进行修复，然而在Xcode10.2，iOS12还会出现问题，不造为什么(手动懵逼)。

> 2018-03-12  Wenson Hsieh  <wenson_hsieh@apple.com>
        REGRESSION(r211643): Dismissing WKActionSheet should not also dismiss its presenting view controller
        https://bugs.webkit.org/show_bug.cgi?id=183549
        <rdar://problem/34960698>
        Reviewed by Andy Estes.
        Fixes the bug by dismissing the presented view controller (i.e. the action sheet or the view controller being
        presented during rotation) rather than the presenting view controller.
        Test: ActionSheetTests.DismissingActionSheetShouldNotDismissPresentingViewController
        * UIProcess/ios/WKActionSheet.mm:
        (-[WKActionSheet doneWithSheet:]):


#### 解决方案
1. push这个包裹webview的vc
2. 子类继承UINavigationController并包裹vc，重写dismissViewControllerAnimated方法

```
override func dismissViewControllerAnimated(flag: Bool, completion: (() -> Void)?) {
    if (self.presentedViewController != nil) {
        super.dismissViewControllerAnimated(flag, completion: completion)
    }
}
```

### 参考
<https://stackoverflow.com/questions/49856616/wkwebview-action-sheet-dismisses-the-presenting-view-controller-after-being-dism>

<https://stackoverflow.com/questions/37380333/modal-view-closes-when-selecting-an-image-in-wkwebview-ios#new-answer>

<https://webkit.googlesource.com/WebKit/+/master/Source/WebKit/ChangeLog>
