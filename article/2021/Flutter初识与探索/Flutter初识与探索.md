# Flutter初识与探索

## 1.技术简介

### 1.1 移动开发技术进化历程

| 技术类型            | UI渲染方式      | 性能 | 开发效率        | 动态化     | 框架代表       |
| ------------------- | --------------- | ---- | --------------- | ---------- | -------------- |
| H5+原生             | WebView渲染     | 一般 | 高              | 支持       | Cordova、Ionic |
| JavaScript+原生渲染 | 原生控件渲染    | 好   | 中              | 支持       | RN、Weex       |
| 自绘UI+原生         | 调用系统API渲染 | 好   | Flutter高, QT低 | 默认不支持 | QT、Flutter    |

备注：

（1）开发语言主要指UI的开发语言。
（2）开发效率，是指整个开发周期的效率，包括编码时间、调试时间、以及排错、兼容时间。
（3）动态化主要指是否支持动态下发代码和是否支持热更新。
值得注意的是Flutter的Release包默认是使用Dart AOT模式编译的，所以不支持动态化，但Dart还有JIT或snapshot运行方式，这些模式都是支持动态化的。

### 1.2 初识Flutter

#### 1.2.1 Flutter简介

Flutter是谷歌的移动UI框架，可以快速在iOS和Android上构建高质量的原生用户界面。 Flutter可以与现有的代码一起工作。在全世界，Flutter正在被越来越多的开发者和组织使用，并且Flutter是完全免费、开源的。

Flutter主打跨平台、高保真、高性能。开发者可以通过 Dart语言开发 App，一套代码同时运行在 iOS 和 Android平台。 Flutter提供了丰富的组件、接口，开发者可以很快地为 Flutter添加 native扩展。同时 Flutter还使用 Native引擎渲染视图，这为用户提供良好的体验。

**跨平台**

Flutter使用自己的高性能渲染引擎来绘制widget。这样不仅可以保证在Android和iOS上UI的一致性，而且也可以避免对原生控件依赖而带来的限制及高昂的维护成本。

目前Flutter默认支持iOS、Android、Fuchsia（Google新的自研操作系统）三个移动平台。但Flutter也可支持Web开发（Flutter for web）和PC开发，此次分享的示例和介绍主要是基于iOS和Android平台的。

**高性能**

Flutter高性能主要靠两点来保证：

首先，Flutter APP采用Dart语言开发。Dart在 JIT（即时编译）模式下，速度与 JavaScript基本持平。但是 Dart支持 AOT，当以 AOT模式运行时，JavaScript便远远追不上了。

其次，Flutter使用自己的渲染引擎来绘制UI，布局数据等由Dart语言直接控制，所以在布局过程中不需要像RN那样要在JavaScript和Native之间通信，这在一些滑动和拖动的场景下具有明显优势，因为在滑动和拖动过程往往都会引起布局发生变化，所以JavaScript需要和Native之间不停的同步布局信息，这和在浏览器中要JavaScript频繁操作DOM所带来的问题是相同的，都会带来比较可观的性能开销。

**Dart**语言开发

Flutter为什么选择了 Dart而不是 JavaScript？

（静态编译与动态解释：静态编译的程序在执行前全部被翻译为机器码，通常将这种类型称为**AOT** （Ahead of time）即 “提前编译”；而解释执行的则是一句一句边翻译边运行，通常将这种类型称为**JIT**（Just-in-time）即“即时编译”）

（**1**）**开发效率高**：

**基于JIT的快速开发周期**：Flutter在开发阶段采用，采用JIT模式，这样就避免了每次改动都要进行编译，极大的节省了开发时间；

**基于AOT的发布包**: Flutter在发布时可以通过AOT生成高效的ARM代码以保证应用性能。而JavaScript则不具有这个能力。

**（2）高性能**

Flutter旨在提供流畅、高保真的的UI体验。为了实现这一点，Flutter中需要能够在每个动画帧中运行大量的代码。这意味着需要一种既能提供高性能的语言，而不会出现会丢帧的周期性暂停，而Dart支持AOT，在这一点上可以做的比JavaScript更好。

**（3）类型安全**

Dart是类型安全的语言，支持静态类型检测，所以可以在编译前发现一些类型的错误，并排除潜在问题。与之不同的，JavaScript是一个弱类型语言，也因此前端社区出现了很多给JavaScript代码添加静态类型检测的扩展语言和工具，如：微软的TypeScript以及Facebook的Flow。

#### 1.2.2 Flutter框架结构

Flutter官方提供的Flutter框架图：

![](https://files.mdnice.com/user/22004/c7af5dca-b3e4-4118-8a09-7e76b9091c9e.png)

**Flutter** **Framework**

这是一个纯 Dart实现的 SDK：

- 底下两层（Foundation和Animation、Painting、Gestures）在Google的一些视频中被合并为一个dart UI层，对应的是Flutter中的`dart:ui`包，它是Flutter引擎暴露的底层UI库，提供动画、手势及绘制能力。
- Rendering层，这一层是一个抽象的布局层，它依赖于dart UI层，Rendering层会构建一个UI树，当UI树有变化时，会计算出有变化的部分，然后更新UI树，最终将UI树绘制到屏幕上(这个过程类似于React中的虚拟DOM)。Rendering层可以说是Flutter UI框架最核心的部分，它除了确定每个UI元素的位置、大小之外还要进行坐标变换、绘制(调用底层dart:ui)。
- Widgets层是Flutter提供的的一套基础组件库，在基础组件库之上，Flutter还提供了 Material 和Cupertino两种视觉风格的组件库。而**Flutter开发的大多数场景，只是和这两层打交道**。

**Flutter** **Engine**

这是一个纯 C++实现的 SDK，其中包括了 Skia引擎、Dart运行时、文字排版引擎等。在代码调用 `dart:ui`库时，调用最终会走到Engine层，然后实现真正的绘制逻辑。

### 1.3 搭建Flutter开发环境

#### 1.3.1 安装Flutter

使用镜像： 参考https://flutter.io/community/china 以获得有关镜像服务器的最新动态

Windows和macOS下Flutter SDK的安装：

（1）系统要求：需满足对应平台的最低需求

（2）获取Flutter SDK：

https://flutter.dev/docs/development/tools/sdk/releases#windows

更新环境变量：在Windows系统自带命令行运行flutter命令
运行flutter doctor命令： 查看是否还需要安装其它依赖

（3）Android设置

**坑点：https://developer.android.com/studio/index.html ==>Service Unavailable 官网提供**
**https://developer.android.google.cn/studio/**

参考：https://blog.csdn.net/dhhyx/article/details/108277116

Flutter依赖于Android Studio的全量安装。Android Studio不仅可以管理Android 平台依赖、SDK版本等，而且它也是Flutter开发推荐的IDE之一（也可以使用其它编辑器或IDE）

#### 1.3.2 IDE配置

理论上可以使用任何文本编辑器与命令行工具来构建Flutter应用程序。 不过，Flutter官方建议使用Android Studio和VS Code之一以获得更好的开发体验。Flutter官方提供了这两款编辑器插件，通过IDE和插件可获得代码补全、语法高亮、widget编辑辅助、运行和调试支持等功能，可以帮助我们极大的提高开发效率。

**坑点：安装FLutter和Dart插件**

在Android Studio中安装插件时，打开插件首选项 (macOS：**Preferences>Plugins**, Windows：**File>Settings>Plugins**)，始终加载不出来Flutter和Dart插件，此时去IDEA官网Developer Tools>All Plugins中搜索插件，选择versions>Compatibility with Android Studio 下载版本和自己安装Android Studio版本对应的版本install到Android Studio中即可(不是直接下载最新版)

![](https://files.mdnice.com/user/22004/399217c4-7d3f-41db-abb1-c6b6f0ecd560.png)
![](https://files.mdnice.com/user/22004/2b855a94-d379-4d56-9ba3-d0d843e5e2a3.png)



创建Flutter应用
运行应用程序

#### 1.3.3 连接设备运行Flutter应用

连接Android模拟器：https://developer.android.com/studio/run/managing-avds.html

**坑点**：**[Android Studio]AVD Manager Unable to locate adb**

https://blog.csdn.net/wenjun_xiao/article/details/107198615

**1.3.4** **遇到问题**？

如果在过程中遇到问题，可以先去flutter官网查看一下安装方式是否发生变化，或者在网上搜索一下解决方案。

### 1.4 Dart语言简介

Dart 编程语言主页https://dart.cn/
dart在线编辑器https://dartpad.cn/

Dart是谷歌的皇太子，更有Flutter等大臣辅佐。继位正统,拳打JS,脚踢Java是分分钟的事情. 所以大家都应该“奉旨”学习Dart.

Dart的设计同时借鉴了Java和JavaScript。Dart在静态语法方面和Java非常相似，如类型定义、函数声明、泛型等，而在动态特性方面又和JavaScript很像，如函数式特性、异步支持等。Dart也具有一些其它具有表现力的语法，如可选命名参数、`..`（级联运算符）和`?.`（条件成员访问运算符）以及`??`（判空赋值运算符）

Dart目前真正的不足是**生态**，但是，随着Flutter的逐渐火热，会回过头来反推Dart生态加速发展。

## 2.应用示例

用Android Studio和VS Code创建的Flutter应用模板默认是一个简单的计数器示例。



![](https://files.mdnice.com/user/22004/e0f94ee9-5d45-47f3-a8e0-8d4357394925.png)

该计数器示例中，每点击一次右下角带“+”号的悬浮按钮，屏幕中央的数字就会加1。

主要Dart代码是在 **lib/main.dart** 文件中:

```dart
import 'package:flutter/material.dart'; //导入包 导入了Material UI组件库

void main() => runApp(new MyApp()); //应用入口:main函数中调用了runApp 方法，它的功能是启动Flutter应用。runApp它接受一个Widget参数，在本示例中它是一个MyApp对象，MyApp()是Flutter应用的根组件。

//应用结构
class MyApp extends StatelessWidget {  //MyApp类代表Flutter应用，它继承了 StatelessWidget类，这也就意味着应用本身也是一个widget。
  @override
  Widget build(BuildContext context) { //Flutter在构建页面时，会调用组件的build方法，widget的主要工作是提供一个build()方法来描述如何构建UI界面（通常是通过组合、拼装其它基础widget）
    return new MaterialApp( //MaterialApp 是Material库中提供的Flutter APP框架，通过它可以设置应用的名称、主题、语言、首页及路由列表等。MaterialApp也是一个widget。
      title: 'Flutter Demo', //应用名称
      theme: new ThemeData(
        primarySwatch: Colors.blue, //蓝色主题 
      ),
      home: new MyHomePage(title: 'Flutter Demo Home Page'), //应用首页路由
    );
  }
}

//MyHomePage 是Flutter应用的首页，它继承自StatefulWidget类，表示它是一个有状态的组件（Stateful widget）
class MyHomePage extends StatefulWidget {
  MyHomePage({Key key, this.title}) : super(key: key);
  final String title;

  @override
  _MyHomePageState createState() => new _MyHomePageState();
}

//_MyHomePageState类是MyHomePage类对应的状态类
class _MyHomePageState extends State<MyHomePage> {
  int _counter = 0;

  void _incrementCounter() {
      //当按钮点击时，会调用此函数，该函数的作用是先自增_counter，然后调用setState 方法。setState方法的作用是通知Flutter框架，有状态发生了改变，Flutter框架收到通知后，会执行build方法来根据新的状态重新构建界面。
    setState(() {
      _counter++;
    });
  }

  @override
  Widget build(BuildContext context) { //构建UI界面的逻辑在build方法中
    return new Scaffold( //Scaffold 是 Material 库中提供的页面脚手架，它提供了默认的导航栏、标题和包含主屏幕widget树的body属性
      appBar: new AppBar(
        title: new Text(widget.title),
      ),
      body: new Center( //body的组件树中包含了一个Center 组件，Center 可以将其子组件树对齐到屏幕中心
        child: new Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            new Text(
              'You have pushed the button this many times:',
            ),
            new Text(
              '$_counter',
              style: Theme.of(context).textTheme.headline4,
            ),
          ],
        ),
      ),
      floatingActionButton: new FloatingActionButton( //floatingActionButton是页面右下角的带“+”的悬浮按钮，它的onPressed属性接受一个回调函数，代表它被点击后的处理器
        onPressed: _incrementCounter,
        tooltip: 'Increment',
        child: new Icon(Icons.add),
      ), // This trailing comma makes auto-formatting nicer for build methods.
    );
  }
}
```



整个计数器执行流程串起来：当右下角的`floatingActionButton`按钮被点击之后，会调用`_incrementCounter`方法。在`_incrementCounter`方法中，首先会自增`_counter`计数器（状态），然后`setState`会通知Flutter框架状态发生变化，接着，Flutter框架会调用`build`方法以新的状态重新构建UI，最终显示在设备屏幕上。

## 3.路由管理

### 3.1 简单示例

示例：

创建一个新路由，命名“NewRoute”

```dart
class NewRoute extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("New route"),
      ),
      body: Center(
        child: Text("This is new route"),
      ),
    );
  }
}

```

在`_MyHomePageState.build`方法中的`Column`的子widget中添加一个按钮（`FlatButton`）

```dart
Column(
      mainAxisAlignment: MainAxisAlignment.center,
      children: <Widget>[
      ... //省略无关代码
      FlatButton(
         child: Text("open new route"),
         textColor: Colors.blue,
         onPressed: () {
          //导航到新路由   
          Navigator.push( context, //Navigator是一个路由管理的组件，它提供了打开和退出路由页方法。
           // MaterialPageRoute继承自PageRoute类，PageRoute类是一个抽象类，表示占有整个屏幕空间的一个模态路由页面，它还定义了路由构建及切换时过渡动画的相关接口及属性。
           MaterialPageRoute(builder: (context) { //builder 是一个WidgetBuilder类型的回调函数，它的作用是构建路由页面的具体内容，返回值是一个widget。我们通常要实现此回调，返回新路由的实例。
              return NewRoute();
           }));
          },
         ),
       ],
 )
```

![](https://files.mdnice.com/user/22004/d116661b-f9c6-4632-bd39-74923e867d02.png)

![](https://files.mdnice.com/user/22004/e53c91d6-a682-4d78-b8d6-8d6ce73d7a9b.png)

### 3.2 路由传值

路由跳转时我们需要带一些参数
创建一个`TipRoute`路由，它接受一个提示文本参数，负责将传入它的文本显示在页面上，另外`TipRoute`中我们添加一个“返回”按钮，点击后在返回上一个路由的同时会带上一个返回参数

`TipRoute`实现代码：

```dart
class TipRoute extends StatelessWidget {
  TipRoute({
    Key key,
    @required this.text,  // 接收一个text参数
  }) : super(key: key);
  final String text;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("提示"),
      ),
      body: Padding(
        padding: EdgeInsets.all(18),
        child: Center(
          child: Column(
            children: <Widget>[
              Text(text),
              RaisedButton(
                onPressed: () => Navigator.pop(context, "我是返回值"),
                child: Text("返回"),
              )
            ],
          ),
        ),
      ),
    );
  }
}
```

打开新路由`TipRoute`的代码：

```dart
class RouterTestRoute extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Center(
      child: RaisedButton(
        onPressed: () async {
          // 打开`TipRoute`，并等待返回结果
          var result = await Navigator.push(
            context,
            MaterialPageRoute(
              builder: (context) {
                return TipRoute(
                  // 路由参数
                  text: "我是提示xxxx",
                );
              },
            ),
          );
          //输出`TipRoute`路由返回结果
          print("路由返回值: $result");
        },
        child: Text("打开提示页"),
      ),
    );
  }
}
```

![](https://files.mdnice.com/user/22004/e5a7a38e-a483-47b0-9d86-09bab4795d0a.png)

![](https://files.mdnice.com/user/22004/ac34400c-924e-49d2-bb82-7b10c1a3357f.png)

提示文案“我是提示xxxx”是通过`TipRoute`的`text`参数传递给新路由页的。我们可以通过等待`Navigator.push(…)`返回的`Future`来获取新路由的返回数据。

在`TipRoute`页中有两种方式可以返回到上一页；第一种方式时直接点击导航栏返回箭头，第二种方式是点击页面中的“返回”按钮。这两种返回方式的区别是前者不会返回数据给上一个路由，而后者会。下面是分别点击页面中的返回按钮和导航栏返回箭头后，`RouterTestRoute`页中`print`方法在控制台输出的内容：

![](https://files.mdnice.com/user/22004/72e75403-8094-4001-bc0c-da0520a749ba.png)

### 3.3 命名路由

注册路由表

```dart
MaterialApp(
  title: 'Flutter Demo',
  theme: ThemeData(
    primarySwatch: Colors.blue,
  ),
  //注册路由表
  routes:{
   "new_page":(context) => NewRoute(),
    ... // 省略其它路由注册信息
  } ,
  home: MyHomePage(title: 'Flutter Demo Home Page'),
);
```

```dart
MaterialApp(
  title: 'Flutter Demo',
  initialRoute:"/", //名为"/"的路由作为应用的home(首页)
  theme: ThemeData(
    primarySwatch: Colors.blue,
  ),
  //注册路由表
  routes:{
   "new_page":(context) => NewRoute(),
   "/":(context) => MyHomePage(title: 'Flutter Demo Home Page'), //注册首页路由
  } 
);
```

通过路由名打开新路由页

```dart
onPressed: () {
  Navigator.pushNamed(context, "new_page");
  //Navigator.push(context,
  //  MaterialPageRoute(builder: (context) {
  //  return NewRoute();
  //}));  
},
```

## 4.基础组件介绍

### 4.1 Widget简介

#### 4.1.1 概念

在Flutter中几乎所有的对象都是一个Widget： 它不仅可以表示UI元素，也可以表示一些功能性的组件如：用于手势检测的 `GestureDetector` widget、用于APP主题数据传递的`Theme`等等

#### 4.1.2 Widget与Element

- Widget实际上就是`Element`的配置数据，Widget树实际上是一个配置树，而真正的UI渲染树是由`Element`构成；不过，由于`Element`是通过Widget生成的，所以它们之间有对应关系，在大多数场景，我们可以宽泛地认为Widget树就是指UI控件树或UI渲染树。
- 一个Widget对象可以对应多个`Element`对象。根据同一份配置（Widget），可以创建多个实例（Element）。

#### 4.1.3 Widget主要接口

Widget类的声明

```dart
@immutable
abstract class Widget extends DiagnosticableTree { //Widget类继承自DiagnosticableTree，DiagnosticableTree即“诊断树”，主要作用是提供调试信息。
  const Widget({ this.key });
  final Key key; //这个key属性主要的作用是决定是否在下一次build时复用旧的widget，决定的条件在canUpdate()方法中。
    
  @protected
  Element createElement(); //一个Widget可以对应多个Element；Flutter Framework在构建UI树时，会先调用此方法生成对应节点的Element对象。

  @override
  String toStringShort() {
    return key == null ? '$runtimeType' : '$runtimeType-$key';
  }

  @override
  void debugFillProperties(DiagnosticPropertiesBuilder properties) { //复写父类的方法，主要是设置诊断树的一些特性。
    super.debugFillProperties(properties);
    properties.defaultDiagnosticsTreeStyle = DiagnosticsTreeStyle.dense;
  }
  
    //canUpdate是一个静态方法，它主要用于在Widget树重新build时复用旧的widget，其实具体来说，应该是：是否用新的Widget对象去更新旧UI树上所对应的Element对象的配置；只要newWidget与oldWidget的runtimeType和key同时相等时就会用newWidget去更新Element对象的配置，否则就会创建新的Element。
  static bool canUpdate(Widget oldWidget, Widget newWidget) {
    return oldWidget.runtimeType == newWidget.runtimeType
        && oldWidget.key == newWidget.key;
  }
}
```

另外`Widget`类本身是一个抽象类，其中最核心的就是定义了`createElement()`接口，在Flutter开发中，我们一般都不用直接继承`Widget`类来实现一个新组件，我们通常会通过继承`StatelessWidget`或`StatefulWidget`来间接继承`Widget`类来实现。`StatelessWidget`和`StatefulWidget`都是直接继承自`Widget`类，而这两个类也正是Flutter中非常重要的两个抽象类，它们引入了两种Widget模型，

#### 4.1.4 StatelessWidget

`StatelessWidget`用于不需要维护状态的场景，它通常在`build`方法中通过嵌套其它Widget来构建UI，在构建过程中会递归的构建其嵌套的Widget。

```dart
class Echo extends StatelessWidget {
  const Echo({
    Key key,  
    @required this.text,
    this.backgroundColor:Colors.grey,
  }):super(key:key);
    
  final String text;
  final Color backgroundColor;

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Container(
        color: backgroundColor,
        child: Text(text),
      ),
    );
  }
}
```



![](https://files.mdnice.com/user/22004/f9b0cee5-4011-4706-9a97-22584ebc1d76.png)

![](https://files.mdnice.com/user/22004/4c7c1efd-9065-4d34-83ed-2905093b1675.png)

Context

`build`方法有一个`context`参数，它是`BuildContext`类的一个实例，表示当前widget在widget树中的上下文，每一个widget都会对应一个context对象（因为每一个widget都是widget树上的一个节点）。实际上，`context`是当前widget在widget树中位置中执行”相关操作“的一个句柄，比如它提供了从当前widget开始向上遍历widget树以及按照widget类型查找父级widget的方法。下面是在子树中获取父级widget的一个示例：

```dart
class ContextRoute extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("Context测试"),
      ),
      body: Container(
        child: Builder(builder: (context) {
          // 在Widget树中向上查找最近的父级`Scaffold` widget
          Scaffold scaffold = context.findAncestorWidgetOfExactType<Scaffold>();
          // 直接返回 AppBar的title， 此处实际上是Text("Context测试")
          return (scaffold.appBar as AppBar).title;
        }),
      ),
    );
  }
}
```

![](https://files.mdnice.com/user/22004/a7a364f0-14d4-4295-878e-efca910201ba.png)

![](https://files.mdnice.com/user/22004/e27ef1b5-4044-4c4d-a663-e4ee810b7aa5.png)

#### 4.1.5 StatefulWidget

和`StatelessWidget`一样，`StatefulWidget`也是继承自`Widget`类，并重写了`createElement()`方法，不同的是返回的`Element` 对象并不相同；另外`StatefulWidget`类中添加了一个新的接口`createState()`。
`StatefulWidget`的类定义:

```dart
abstract class StatefulWidget extends Widget {
  const StatefulWidget({ Key key }) : super(key: key);
    
  @override
  StatefulElement createElement() => new StatefulElement(this);
    
  @protected
  State createState();
}
```

- `StatefulElement` 间接继承自`Element`类，与StatefulWidget相对应（作为其配置数据）。`StatefulElement`中可能会多次调用`createState()`来创建状态(State)对象。
- `createState()` 用于创建和Stateful widget相关的状态，它在Stateful widget的生命周期中可能会被多次调用。例如，当一个Stateful widget同时插入到widget树的多个位置时，Flutter framework就会调用该方法为每一个位置生成一个独立的State实例，其实，本质上就是一个`StatefulElement`对应一个State实例。

#### 4.1.6 State

一个StatefulWidget类会对应一个State类，State表示与其对应的StatefulWidget要维护的状态，State中的保存的状态信息可以：

1. 在widget 构建时可以被同步读取。
2. 在widget生命周期中可以被改变，当State被改变时，可以手动调用其`setState()`方法通知Flutter framework状态发生改变，Flutter framework在收到消息后，会重新调用其`build`方法重新构建widget树，从而达到更新UI的目的。

**生命周期**

![](https://files.mdnice.com/user/22004/54d1a66b-927e-42f5-944f-7b052ee42444.png)

![](https://files.mdnice.com/user/22004/d3d3bbb3-c8c5-4ae1-9fac-144b8e5e7a75.png)

![](https://files.mdnice.com/user/22004/e795cbc5-5d18-408d-8dde-8cb672ff11a5.png)

#### 4.1.7 在Widget树中获取State对象

由于StatefulWidget的的具体逻辑都在其State中，所以很多时候，我们需要获取StatefulWidget对应的State对象来调用一些方法，比如`Scaffold`组件对应的状态类`ScaffoldState`中就定义了打开SnackBar(路由页底部提示条)的方法。我们有两种方法在子widget树中获取父级StatefulWidget的State对象。

**通过Context获取**

`context`对象有一个`findAncestorStateOfType()`方法，该方法可以从当前节点沿着widget树向上查找指定类型的StatefulWidget对应的State对象

```dart
Scaffold(
  appBar: AppBar(
    title: Text("子树中获取State对象"),
  ),
  body: Center(
    child: Builder(builder: (context) {
      return RaisedButton(
        onPressed: () {
          // 查找父级最近的Scaffold对应的ScaffoldState对象
          ScaffoldState _state = context.findAncestorStateOfType<ScaffoldState>();
          //调用ScaffoldState的showSnackBar来弹出SnackBar
          _state.showSnackBar(
            SnackBar(
              content: Text("我是SnackBar"),
            ),
          );
        },
        child: Text("显示SnackBar"),
      );
    }),
  ),
);
```

![](https://files.mdnice.com/user/22004/b080e1aa-a6ea-4ceb-b70a-c1cf8f1a832e.png)

![](https://files.mdnice.com/user/22004/03c1590c-d038-425d-9607-f45d6e209615.png)



**通过GlobalKey**

### 4.2 状态管理

管理状态的最常见的方法：

- Widget管理自己的状态。
- Widget管理子Widget状态。
- 混合管理（父Widget和子Widget都管理状态）。

#### 4.2.1 Widget管理自身状态

_TapboxAState 类:

- 管理TapboxA的状态。
- 定义`_active`：确定盒子的当前颜色的布尔值。
- 定义`_handleTap()`函数，该函数在点击该盒子时更新`_active`，并调用`setState()`更新UI。
- 实现widget的所有交互式行为。
  
![](https://files.mdnice.com/user/22004/35f6b058-c95d-4e9c-a55b-05530942a322.png)

![](https://files.mdnice.com/user/22004/7dc27bf1-ee51-4c80-8cd9-4f072ab6509e.png)

![](https://files.mdnice.com/user/22004/b21f83b7-5a47-473e-814d-d5f303efa945.png)

#### 4.2.2 父Widget管理子Widget的状态

对于父Widget来说，管理状态并告诉其子Widget何时更新通常是比较好的方式。

在以下示例中，TapboxB通过回调将其状态导出到其父组件，状态由父组件管理，因此它的父组件为`StatefulWidget`。但是由于TapboxB不管理任何状态，所以`TapboxB`为`StatelessWidget`。

`ParentWidgetState` 类:

- 为TapboxB 管理`_active`状态。
- 实现`_handleTapboxChanged()`，当盒子被点击时调用的方法。
- 当状态改变时，调用`setState()`更新UI。

TapboxB 类:

- 继承`StatelessWidget`类，因为所有状态都由其父组件处理。

- 当检测到点击时，它会通知父组件。

  ![](https://files.mdnice.com/user/22004/f2687313-a78a-41f1-9b2f-1dbca3050964.png)

  ![](https://files.mdnice.com/user/22004/e6cf2728-f74a-4057-a4c2-12a2c2667d95.png)





#### 4.2.3 混合状态管理

对于一些组件来说，混合管理的方式会非常有用。在这种情况下，组件自身管理一些内部状态，而父组件管理一些其他外部状态。

在下面TapboxC示例中，手指按下时，盒子的周围会出现一个深绿色的边框，抬起时，边框消失。点击完成后，盒子的颜色改变。 TapboxC将其`_active`状态导出到其父组件中，但在内部管理其`_highlight`状态。这个例子有两个状态对象`_ParentWidgetState`和`_TapboxCState`。

`_ParentWidgetStateC`类:

- 管理`_active` 状态。
- 实现 `_handleTapboxChanged()` ，当盒子被点击时调用。
- 当点击盒子并且`_active`状态改变时调用`setState()`更新UI。

`_TapboxCState` 对象:

- 管理`_highlight` 状态。
- `GestureDetector`监听所有tap事件。当用户点下时，它添加高亮（深绿色边框）；当用户释放时，会移除高亮。
- 当按下、抬起、或者取消点击时更新`_highlight`状态，调用`setState()`更新UI。
- 当点击时，将状态的改变传递给父组件。

![](https://files.mdnice.com/user/22004/1ecc447a-51ac-4402-9305-6927deffed08.png)



![](https://files.mdnice.com/user/22004/c7da5472-fae1-46d7-a740-754458bca4bc.png)

### 4.3 文本、按钮、Icon

![](https://files.mdnice.com/user/22004/224ffcf5-413e-4820-bab6-683a7627cd0b.png)





### 4.4 输入框及表单

#### 4.4.1 TextField

```dart
Column(
        children: <Widget>[
          TextField(
            autofocus: true,
            decoration: InputDecoration(
                labelText: "用户名",
                hintText: "用户名或邮箱",
                prefixIcon: Icon(Icons.person)
            ),
          ),
          TextField(
            decoration: InputDecoration(
                labelText: "密码",
                hintText: "您的登录密码",
                prefixIcon: Icon(Icons.lock)
            ),
            obscureText: true,
          ),
        ],
);
```



![](https://files.mdnice.com/user/22004/2a3d312d-57e8-493f-9d71-438daf4cfe6a.png)

![](https://files.mdnice.com/user/22004/8dc3a5bf-a71f-4bad-99f6-5c348f4538dd.png)

获取输入内容和监听文本变化

`onChange`回调
通过`controller`直接获取（监听）
两种方式相比，`onChanged`是专门用于监听文本变化，而`controller`的功能却多一些，除了能监听文本变化外，它还可以设置默认值、选择文本

控制焦点
焦点可以通过`FocusNode`和`FocusScopeNode`来控制，默认情况下，焦点由`FocusScope`来管理，它代表焦点控制范围，可以在这个范围内可以通过`FocusScopeNode`在输入框之间移动焦点、设置默认焦点等。我们可以通过`FocusScope.of(context)` 来获取Widget树中默认的`FocusScopeNode`

```dart
class FocusTestRoute extends StatefulWidget {
  @override
  _FocusTestRouteState createState() => new _FocusTestRouteState();
}

class _FocusTestRouteState extends State<FocusTestRoute> {
  FocusNode focusNode1 = new FocusNode();
  FocusNode focusNode2 = new FocusNode();
  FocusScopeNode focusScopeNode;

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: EdgeInsets.all(16.0),
      child: Column(
        children: <Widget>[
          TextField(
            autofocus: true, 
            focusNode: focusNode1,//关联focusNode1
            decoration: InputDecoration(
                labelText: "input1"
            ),
          ),
          TextField(
            focusNode: focusNode2,//关联focusNode2
            decoration: InputDecoration(
                labelText: "input2"
            ),
          ),
          Builder(builder: (ctx) {
            return Column(
              children: <Widget>[
                RaisedButton(
                  child: Text("移动焦点"),
                  onPressed: () {
                    //将焦点从第一个TextField移到第二个TextField
                    // 这是一种写法 FocusScope.of(context).requestFocus(focusNode2);
                    // 这是第二种写法
                    if(null == focusScopeNode){
                      focusScopeNode = FocusScope.of(context);
                    }
                    focusScopeNode.requestFocus(focusNode2);
                  },
                ),
                RaisedButton(
                  child: Text("隐藏键盘"),
                  onPressed: () {
                    // 当所有编辑框都失去焦点时键盘就会收起  
                    focusNode1.unfocus();
                    focusNode2.unfocus();
                  },
                ),
              ],
            );
          },
          ),
        ],
      ),
    );
  }

}
```

![](https://files.mdnice.com/user/22004/f0d4691c-aff4-47af-9835-26aff61974e0.png)

![](https://files.mdnice.com/user/22004/fa285c2b-06bf-4c2c-884c-6e00b4515892.png)

#### 4.4.2 表单Form

修改一下上面用户登录的示例，在提交之前校验：

1. 用户名不能为空，如果为空则提示“用户名不能为空”。

2. 密码不能小于6位，如果小于6为则提示“密码不能少于6位”。

   ```dart
   class FormTestRoute extends StatefulWidget {
     @override
     _FormTestRouteState createState() => new _FormTestRouteState();
   }
   
   class _FormTestRouteState extends State<FormTestRoute> {
     TextEditingController _unameController = new TextEditingController();
     TextEditingController _pwdController = new TextEditingController();
     GlobalKey _formKey= new GlobalKey<FormState>();
   
     @override
     Widget build(BuildContext context) {
       return Scaffold(
         appBar: AppBar(
           title:Text("Form Test"),
         ),
         body: Padding(
           padding: const EdgeInsets.symmetric(vertical: 16.0, horizontal: 24.0),
           child: Form(
             key: _formKey, //设置globalKey，用于后面获取FormState
             autovalidate: true, //开启自动校验
             child: Column(
               children: <Widget>[
                 TextFormField(
                     autofocus: true,
                     controller: _unameController,
                     decoration: InputDecoration(
                         labelText: "用户名",
                         hintText: "用户名或邮箱",
                         icon: Icon(Icons.person)
                     ),
                     // 校验用户名
                     validator: (v) {
                       return v
                           .trim()
                           .length > 0 ? null : "用户名不能为空";
                     }
   
                 ),
                 TextFormField(
                     controller: _pwdController,
                     decoration: InputDecoration(
                         labelText: "密码",
                         hintText: "您的登录密码",
                         icon: Icon(Icons.lock)
                     ),
                     obscureText: true,
                     //校验密码
                     validator: (v) {
                       return v
                           .trim()
                           .length > 5 ? null : "密码不能少于6位";
                     }
                 ),
                 // 登录按钮
                 Padding(
                   padding: const EdgeInsets.only(top: 28.0),
                   child: Row(
                     children: <Widget>[
                       Expanded(
                         child: RaisedButton(
                           padding: EdgeInsets.all(15.0),
                           child: Text("登录"),
                           color: Theme
                               .of(context)
                               .primaryColor,
                           textColor: Colors.white,
                           onPressed: () {
                             //在这里不能通过此方式获取FormState，context不对
                             //print(Form.of(context));
                               
                             // 通过_formKey.currentState 获取FormState后，
                             // 调用validate()方法校验用户名密码是否合法，校验
                             // 通过后再提交数据。 
                             if((_formKey.currentState as FormState).validate()){
                               //验证通过提交数据
                             }
                           },
                         ),
                       ),
                     ],
                   ),
                 )
               ],
             ),
           ),
         ),
       );
     }
   }
   ```

   ![](https://files.mdnice.com/user/22004/86591f44-f75a-44c1-9149-aaa09ae83b35.png)

   ![](https://files.mdnice.com/user/22004/8e781872-1cec-4183-bf11-3198d1523098.png)



## 5.参考

1. [Flutter中文网](https://flutterchina.club/)
2. [《Flutter实战》](https://book.flutterchina.club/)
