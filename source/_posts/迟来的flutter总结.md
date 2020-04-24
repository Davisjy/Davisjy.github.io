---
title: 迟来的Flutter总结
date: 2019-06-13 10:32:35
tags: Flutter初体验
categories: Flutter
---

### 前言
本篇是个人在公司分享Flutter的内容，在此搬到博客上<!--more-->

### Flutter介绍

> Flutter是Google发布的一个用于创建跨平台、高性能移动应用的框架。Flutter和QT mobile一样，都没有使用原生控件，相反都实现了一个自绘引擎，使用自身的布局、绘制系统。那么，我们会担心，QT mobile面对的问题Flutter是否也一样，Flutter会不会步入QT mobile后尘，成为另一个烈士？要回到这个问题，我们先来看看Flutter诞生过程：

> 2017 年 Google I/O 大会上，Google 首次推出了一款新的用于创建跨平台、高性能的移动应用框架——[Flutter](https://github.com/flutter/flutter)。
2018年2月，Flutter发布了第一个Beta版本，同年五月， 在2018年Google I/O 大会上，Flutter 更新到了 beta 3 版本。
2018年6月，Flutter发布了首个预览版本，这意味着 Flutter 进入了正式版（1.0）发布前的最后阶段。
观其发展，在2018年5月份，Flutter 进入了 GitHub stars 排行榜前 100 名，已有 27k star。而今天(2019年6月10日)，已经有66K+的Star。经历了短短2年多的时间，Flutter 生态系统得以快速增长，由此可见，Flutter在开发者中受到了热烈的欢迎，其未来发展值得期待！

#### 生态及社区
从Github上来看，目前Flutter活跃用户正在高速增长。从Stackoverflow上提问来看，Flutter社区现在已经很庞大。Flutter的文档、资源也越来越丰富，开发过程中遇到的很多问题都可以在Stackoverflow或其github issue中找到答案。在掘金上，Flutter 标签下有 2300 多篇文章；闲鱼团队也在主推 Flutter。关注社区的活跃度其实就是想知道会不会在短期内挂掉，从目前的状况看，我觉得可能性比较小，而它被另一个同类产品 PK 掉的可能性则更小，因此值得投入时间去了解它。

#### 技术支持
现在Google正在大力推广Flutter，Flutter的作者中很多人都是来自Chromium团队，并且github上活跃度很高。另一个角度，从今年上半年Flutter频繁的版本发布也可以看出Google对Flutter的投入的资源不小，所以在官方技术支持这方面，大可不必担心。

#### 开发效率
Flutter的热重载可帮助开发者快速地进行测试、构建UI、添加功能并更快地修复错误。在iOS和Android模拟器或真机上可以实现毫秒级热重载，并且不会丢失状态。

#### 小结
个人认为技术发展到一定程度就会衍生出新的方向，比如后端高并发，分布式，云计算，集群等。对于移动端发展到最后估计只有大前端这一个方向，一统三端指日可待，当然这仅仅是表层业务逻辑上的统一。

-

### 第一节--对整个Wigets有个大概认识

#### 概念

在前面的介绍中，我们知道Flutter中几乎所有的对象都是一个Widget，与原生开发中“控件”不同的是，Flutter中的widget的概念更广泛，它不仅可以表示UI元素，也可以表示一些功能性的组件如：用于手势检测的 GestureDetector widget、用于应用主题数据传递的Theme等等。而原生开发中的控件通常只是指UI元素。在后面的内容中，我们在描述UI元素时，我们可能会用到“控件”、“组件”这样的概念，心里需要知道他们就是widget，只是在不同场景的不同表述而已。由于Flutter主要就是用于构建用户界面的，所以，在大多数时候，可以认为widget就是一个控件，不必纠结于概念。

#### Widget与Element
在Flutter中，Widget的功能是“描述一个UI元素的配置数据”，它就是说，Widget其实并不是表示最终绘制在设备屏幕上的显示元素，而只是显示元素的一个配置数据。实际上，Flutter中真正代表屏幕上显示元素的类是Element，也就是说Widget只是描述Element的一个配置，且一个Widget可以对应多个Element，这是因为同一个Widget对象可以被添加到UI树的不同部分，而真正渲染时，UI树的每一个Element节点都会对应一个Widget对象。总结一下：

> 1. Widget实际上就是Element的配置数据，Widget树实际上是一个配置树，而真正的UI渲染树是由Element构成；不过，由于Element是通过Widget生成，所以它们之间有对应关系，所以在大多数场景，我们可以宽泛地认为Widget树就是指UI控件树或UI渲染树
2. 一个Widget对象可以对应多个Element对象。这很好理解，根据同一份配置（Widget），可以创建多个实例（Element）

##### 源码
```
@immutable
abstract class Widget extends DiagnosticableTree {
  const Widget({ this.key });
  final Key key;

  @protected
  Element createElement();

  @override
  String toStringShort() {
    return key == null ? '$runtimeType' : '$runtimeType-$key';
  }

  @override
  void debugFillProperties(DiagnosticPropertiesBuilder properties) {
    super.debugFillProperties(properties);
    properties.defaultDiagnosticsTreeStyle = DiagnosticsTreeStyle.dense;
  }

  static bool canUpdate(Widget oldWidget, Widget newWidget) {
    return oldWidget.runtimeType == newWidget.runtimeType
        && oldWidget.key == newWidget.key;
  }
}
```

- Widget类继承自DiagnosticableTree，DiagnosticableTree即“诊断树”，主要作用是提供调试信息。
- Key: 这个key属性类似于React/Vue中的key，主要的作用是决定是否在下一次build时复用旧的widget，决定的条件在canUpdate()方法中。
- createElement()：正如前文所述“一个Widget可以对应多个Element”；Flutter Framework在构建UI树时，会先调用此方法生成对应节点的Element对象。此方法是Flutter Framework隐式调用的，在我们开发过程中基本不会调用到。
- debugFillProperties(...) 复写父类的方法，主要是设置诊断树的一些特性。
- canUpdate(...)是一个静态方法，它主要用于在Widget树重新build时复用旧的widget，其实具体来说，应该是：是否用新的Widget对象去更新旧UI树上所对应的Element对象的配置；通过其源码我们可以看到，只要newWidget与oldWidget的runtimeType和key同时相等时就会用newWidget去更新Element对象的配置，否则就会创建新的Element。

#### Widgets
介绍一下Flutter中常用的一些基础widget，由于大多数widget的属性都比较多，分享会为了不占用太多时间，不会像API文档一样所有属性都介绍，关于属性详细的信息可下去自行参考Flutter SDK文档。

##### 基础Widget
- 文本、字体样式 __Text__
- 按钮 __RaisedButton__ __FlatButton__ __IconButton__ 
- 图片和Icon __Image__ __Icon__
- 单选框和复选框 __Switch__ __Checkbox__
- 输入框和表单 __TextField__

##### 布局类Widget
- 线性布局 __Row__ __Column__
- 弹性布局 __Flex__
- 流式布局 __Wrap__ __Flow__
- 层叠布局 __Stack__ __Positioned__

##### 容器类Widget
- __Padding__
- 布局限制类容器 __ConstrainedBox__ __SizeBox__
- 装饰容器 __DecoratedBox__
- 变换 __Transform__
- __Container__ 容器
- __Scaffold__ __TabBar__ __底部导航__

##### 可滚动Widget
- __SingleChildScrollView__
- __ListView__
- __GridView__
- __CustomScrollView__
- 滚动监听及控制 __ScrollController__

-

### 第二节--State生命周期

- initState：当Widget第一次插入到Widget树时会被调用，对于每一个State对象，Flutter framework只会调用一次该回调，所以，通常在该回调中做一些一次性的操作，如状态初始化、订阅子树的事件通知等。不能在该回调中调用BuildContext.inheritFromWidgetOfExactType（该方法用于在Widget树上获取离当前widget最近的一个父级InheritFromWidget，原因是在初始化完成后，Widget树中的InheritFromWidget也可能会发生变化，所以正确的做法应该在在build（）方法或didChangeDependencies()中调用它。
- didChangeDependencies()：当State对象的依赖发生变化时会被调用；例如：在之前build() 中包含了一个InheritedWidget，然后在之后的build() 中InheritedWidget发生了变化，那么此时InheritedWidget的子widget的didChangeDependencies()回调都会被调用。典型的场景是当系统语言Locale或应用主题改变时，Flutter framework会通知widget调用此回调。
- build()：此回调读者现在应该已经相当熟悉了，它主要是用于构建Widget子树的，会在如下场景被调用：

	1. 在调用initState()之后。
	2. 在调用didUpdateWidget()之后。
	3. 在调用setState()之后。
	4. 在调用didChangeDependencies()之后。
	5. 在State对象从树中一个位置移除后（会调用deactivate）又重新插入到树的其它位置之后。

- reassemble()：此回调是专门为了开发调试而提供的，在热重载(hot reload)时会被调用，此回调在Release模式下永远不会被调用。
- didUpdateWidget()：在widget重新构建时，Flutter framework会调用Widget.canUpdate来检测Widget树中同一位置的新旧节点，然后决定是否需要更新，如果Widget.canUpdate返回true则会调用此回调。正如之前所述，Widget.canUpdate会在新旧widget的key和runtimeType同时相等时会返回true，也就是说在在新旧widget的key和runtimeType同时相等时didUpdateWidget()就会被调用。
- deactivate()：当State对象从树中被移除时，会调用此回调。在一些场景下，Flutter framework会将State对象重新插到树中，如包含此State对象的子树在树的一个位置移动到另一个位置时（可以通过GlobalKey来实现）。如果移除后没有重新插入到树中则紧接着会调用dispose()方法。
- dispose()：当State对象从树中被永久移除时调用；通常在此回调中释放资源。

#### 示例
通过一个demo来演示一下State的生命周期。

```
class CounterWidget extends StatefulWidget {
  const CounterWidget({
    Key key,
    this.initValue: 0
  });

  final int initValue;

  @override
  _CounterWidgetState createState() => new _CounterWidgetState();
}
```

```
class _CounterWidgetState extends State<CounterWidget> {  
  int _counter;

  @override
  void initState() {
    super.initState();
    //初始化状态  
    _counter=widget.initValue;
    print("initState");
  }

  @override
  Widget build(BuildContext context) {
    print("build");
    return Scaffold(
      body: Center(
        child: FlatButton(
          child: Text('$_counter'),
          //点击后计数器自增
          onPressed:()=>setState(()=> ++_counter,
          ),
        ),
      ),
    );
  }

  @override
  void didUpdateWidget(CounterWidget oldWidget) {
    super.didUpdateWidget(oldWidget);
    print("didUpdateWidget");
  }

  @override
  void deactivate() {
    super.deactivate();
    print("deactive");
  }

  @override
  void dispose() {
    super.dispose();
    print("dispose");
  }

  @override
  void reassemble() {
    super.reassemble();
    print("reassemble");
  }

  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    print("didChangeDependencies");
  }

}
```

-

### 第三节--学习路线

#### 背景
作为前端程序员，能够一眼看到自己写出来的效果是非常有成就感并且会提升兴趣，所以在学习完一系列基础widget就可以搭界面了。

##### 17个demo
在此找到了[JSPang](https://juejin.im/user/5875dfc7a22b9d0058a96d06)的博客，本质是录视频供初学者学习，在此千万不要看视频，浪费时间，不如找个开源项目自己动手敲一遍。


#### 最后说一下我的学习路线
> 首先[官方文档](https://flutterchina.club)是 __一定__ 要过一遍的，发现Dart是新语言，不要慌。据了解，掌握JS/Java/Swift/Kotlin这些语言的开发者基本无压力，当然也有个别语法不一样，我也找到真丶深红骑士的Flutter学习之[Dart语法特性](https://juejin.im/post/5c44727df265da611c274087)供大家快速掌握Dart语言。内容太多，可在需要时自行查阅。

天真的以为能够动手写项目了，还不够， __重点__ 学习一系列状态管理

1. [Scoped Model](https://juejin.im/post/5b97fa0d5188255c5546dcf8)
2. [Redux](https://juejin.im/post/5ba26c086fb9a05ce57697da)
3. [BLoC](https://juejin.im/post/5bb6f344f265da0aa664d68a)
4. [RxDart](https://juejin.im/post/5bcea438e51d4536c65d2232)

如果这四篇博客看完，对状态管理就有了基本认识，后续rx系列高级语法可以自行深究

-

### 第四节--杂七杂八的总结

#### IDE选择
Android studio 和 VSCode各有千秋，之前用过哪个就用哪个。

#### 个人对大前端方向的看法
无论是之前的 `RN`\ `Weex`，还是如今的`Flutter`，都是衍生品亦或说是替代品，技术驱动学习，不然总会被淘汰。

项目不在多，基础是根基，深入源码才是硬道理。

最后感谢社区，感谢开源精神。

#### 致谢
[Flutter中文网](https://book.flutterchina.club/chapter14/render_object.html) 鼎力推荐⭐️⭐️⭐️⭐️⭐️

[Flutter 跨平台移动应用开发实战](https://flutter-app-in-action.netlify.com/#/get-start)

[GSYFlutterBook实战详解系列](https://guoshuyu.cn/home/wx/)

[Flutter | 深入浅出Key](https://juejin.im/post/5ca2152f6fb9a05e1a7a9a26)

[咸鱼技术](https://www.jianshu.com/u/cf5c0e4b1111)

[Flutter中文社区](https://flutter-io.cn)
