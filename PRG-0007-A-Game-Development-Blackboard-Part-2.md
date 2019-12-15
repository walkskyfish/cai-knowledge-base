# Game Development Blackboard - Part 2

## 2019-12-12 星期四

### ECS

![](media/15761253452577.jpg)

![](media/15761253912907.jpg)

![](media/15761257357160.jpg)

![](media/15761256109411.jpg)

![](media/15761257484830.jpg)

![](media/15761259592638.jpg)

![](media/Screen%20Shot%202019-12-12%20at%2001.04.09.png)

![](media/15761263118685.jpg)

![](media/15761273293224.jpg)

![](media/15761279211616.jpg)

* World System
* Tuple 组件元组，可以调用 Sibling() 函数获取同一个元组内的组件；Tuple 使用 struct 结构体来定义
* EntityAdmin 是一个 World（System），会调用所有 System 的 Update()
* 每个 System 都会执行一些工作；不在固定的 Tuple 元组组件集合上执行操作，而是选择了一些基础组件来遍历，然后再由相应的行为去调用其他兄弟组件
* 不同的 System 作为不同的观察者，可以从不同角度/方式去处理同一个 Component；根据主体视角区分所有 Behaviour，这样来描述一个 Game Object 游戏对象的全部行为会更容易
* Singleton Component 单例组件，单一的匿名实体，可以通过 EntityAdmin 直接访问；单例组件的使用十分普遍
* Utility 是定义共享行为的 System；如果你想在多处调用一个 Utility 函数，那么这个函数应该依赖很少的组件，而且不应该带副作用或很少副作用；如果一个 Utility 函数依赖很多组件，那就试着限制调用点的数量
* 好的单例组件可以通过「推迟」（Deferment）来解决系统间耦合的问题。存储行为所需的状态，然后把副作用延后到当前帧里更好的时机再执行
* Entity 如果拥有行为所需的 Tuple 组件元组，它就会是这个行为的主体
* 通过 Tuple 元组可以知道什么 System 可以访问什么状态
* 定义 Tuple 元组时，可以把组件标记上「只读」属性，这意味着，即使有多个 System 都操作该组件，但都是只读的，可以并行处理
* ECS 的使用准则：

    - Component 没有函数
    - System 没有状态
    - 共享代码要放在 Utility 中
    - Component 中复杂的副作用要通过队列的方式推迟处理，尤其是 Singleton Component
    - System 不能调用其他 System

* 仍然有大量代码不符合这个规范，它们是复杂度和维护工作的主要来源

### Overwatch 中的 ECS 架构

* [浅谈《守望先锋》中的 ECS 架构 - 云风的 BLOG](https://blog.codingnow.com/2017/06/overwatch_ecs.html)

> ECS 的 E ，也就是 Entity ，可以说就是传统引擎中的 Game Object 。但在这个系统下，它仅仅是 C/Component 的组合。**它的意义在于生命期管理**，这里是用 32bit ID 而不是指针来表示的，另外附着了渲染用到的资源 ID 。因为仅负责生命期管理，而不设计调用其上的方法，用整数 ID 更健壮。整数 ID 更容易指代一个无效的对象，而指针就很难做到。

> 游戏的业务循环就是在调用很多不同的系统，每个系统自己遍历自己感兴趣的对象，只有预定义的组件部分可以被子系统感知到，这样每个系统就能具备很强的内聚性。注意、这和传统的面向对象或是 Actor 模型是截然不同的。OO 或 Actor 强调的是对象自身处理自身的业务，然后框架去管理对象的集合，负责用消息驱动它们。**而在 ECS 中，每个系统关注的是不同的对象集合，它处理的对象中有共性的切片。**

> ECS 的设计就是为了管理复杂度，它提供的指导方案就是 Component 是纯数据组合，没有任何操作这个数据的方法；而 System 是纯方法组合，它自己没有内部状态。**它要么做成无副作用的纯函数，根据它所能见到的对象 Component 组合计算出某种结果；要么用来更新特定 Component 的状态。System 之间也不需要相互调用（减少耦合），是由游戏世界（外部框架）来驱动若干 System 的。**如果满足了这些前提条件，每个 System 都可以独立开发，它只需要遍历给框架提供给它的组件集合，做出正确的处理，更新组件状态就够了。编写 Gameplay 的人更像是在用胶水粘合这些 System ，他只要清楚每个 System 到底做了什么，操作本身对哪些 Component 造成了影响，正确的书写 System 的更新次序就可以了。一个 System 对大多数 Component 是只读的，只对少量 Component 是会改写的，这个可以预先定义清楚，有了这个知识，一是容易管理复杂度，二是给并行处理留下了优化空间。

> 如果产生状态改变这种副作用的行为必须存在时，又在很多 System 中都会触发，那么为了减少调用的地方，就需要把真正产生副作用的点集中在一处了。这个技巧就是**推迟行为的发生时机。就是把行为发生时需要的状态保存起来，放在队列里，由一个单独的 System 在独立的环节集中处理它们。**

### Entitas

* [Entitas-CSharp - GitHub](https://github.com/sschmid/Entitas-CSharp)
* [Entity System Architecture with Unity - Unite Europe 2015](https://www.slideshare.net/sschmid/uniteeurope-2015)
* [ECS architecture with Unity by example - Unite Europe 2016](https://www.slideshare.net/sschmid/uniteeurope-2016)
* [Clean, fast and simple with Entitas and Unity - Unite Melbourne 2016](https://www.slideshare.net/sschmid/unite-melbourne-2016-clean-fast-and-simple-with-entitas-and-unity)

* [Tutorials - Entitas-CSharp](https://github.com/sschmid/Entitas-CSharp/wiki/Tutorials)
* [FAQ - Entitas-CSharp](https://github.com/sschmid/Entitas-CSharp/wiki/FAQ)
* [The Basics - Entitas-CSharp](https://github.com/sschmid/Entitas-CSharp/wiki/The-Basics)
* [Attributes - Entitas-CSharp](https://github.com/sschmid/Entitas-CSharp/wiki/Attributes)

* [Entitas-Shmup - GitHub](https://github.com/sschmid/Entitas-Shmup)

### Use Entitas

* [How I build games with Entitas? - GitHub](https://github.com/sschmid/Entitas-CSharp/wiki/How-I-build-games-with-Entitas-%28FNGGames%29)

![](media/15762125313958.jpg)

> **Data**: The game state. Data such as health, inventory, experience, enemy type, ai state, movement speed etc. In Entitas these data live in Components.
> **Logic**: The rules for how the data can be transformed. PlaceInInventory(), BuildItem(), FireWeapon() etc. In Entitas these are Systems.
> **View**: The code responsible for displaying the game state to the player, rendering, animation, audio, ui widget etc. In my examples these will be MonoBehaviours living on GameObjects.
> **Services**: Outside sources and sinks for information e.g. Pathfinding, Leaderboards, Anti-Cheat, Social, Physics, even the game engine itself.
> **Input**: Outside input to the simulation, usually via limited access to parts of the game logic e.g. controller / keyboard / mouse input, network input.

* Abstraction：抽象
* Interface：接口
* Inversion of control：依赖倒置
* View Layer Abstraction：视图层抽象
* Event：事件

## 2019-12-02 星期一

### Building apps for Android

* [Getting started with Android development - Unity Manual](https://docs.unity3d.com/2018.4/Documentation/Manual/android-GettingStarted.html)
* [Android environment setup - Unity Manual](https://docs.unity3d.com/2018.4/Documentation/Manual/android-sdksetup.html)
* [Troubleshooting Android development - Unity Manual](https://docs.unity3d.com/2018.4/Documentation/Manual/TroubleShootingAndroid.html)

* **Unity Android 发布环境设置**

    - 在 Unity Hub 为相应版本的 Unity 添加 Android Build Support 模块
    - 下载并安装 [Android Studio](https://developer.android.com/studio)
    - 在 Android Studio 中下载并安装 Android SDK Tools 26.1.1、Android SDK Platform-Tools、Android SDK Build-Tools、SDK Platform
    - 在 Unity 编辑器菜单 Preferences -> External Tools 的 NDK 项，点击 Download 下载相应版本的 NDK（Unity 2018.4.12f1 对应的是 android-ndk-r16b）

### Android 开发调试环境设置

**adb 工具所在目录：**

`sdk/platform-tools/adb.exe`

**monitor 工具所在目录：**

`sdk/tools/monitor.bat`

### 自定义 Android 闪屏

* [Customizing an Android Splash Screen - Unity Manual](https://docs.unity3d.com/2018.4/Documentation/Manual/AndroidMobileCustomizeSplashScreen.html)

## 2019-11-25 星期一

### 2D 背景图片循环滚动

* 方案一：控制 `Renderer.material.mainTextureOffset`，实现纹理的滚动效果
* 方案二：将无缝 Sprite 首尾相接，通过控制 GameObject 位置，实现滚动效果

* [Unity 2D 背景滚动 - CSDN](https://blog.csdn.net/wuhaishengxxx/article/details/53033095)
* [背景循环滚动 - 知乎专栏](https://zhuanlan.zhihu.com/p/53913978)

## 2019-11-21 星期四

### 2D 动画

* [Unity 2D 动画系统教程 - bilibli](https://www.bilibili.com/video/av38214700?p=1)
* [Introduction to 2D Animation - Unity Docs](https://docs.unity3d.com/Packages/com.unity.2d.animation@2.2/manual/index.html)
* [PSD Importer - Unity Docs](https://docs.unity3d.com/Packages/com.unity.2d.psdimporter@1.2/manual/index.html)
* [2D Inverse Kinematics（IK）- Unity Docs](https://docs.unity3d.com/Packages/com.unity.2d.ik@1.2/manual/index.html)

## 2019-11-14 星期四

### Object Management

* [Object Management - Catlike Coding](https://catlikecoding.com/unity/tutorials/object-management/)
* [Multiple Scenes - Catlike Coding](https://catlikecoding.com/unity/tutorials/object-management/multiple-scenes/)

### Unity City Builder Asset

* [Mobile Touch Camera - Unity Asset Store](https://assetstore.unity.com/packages/tools/camera/mobile-touch-camera-43960)
* [City Adventure](https://www.beffio.com/city-adventure)
* [RPG Medieval Kingdom Kit](https://www.beffio.com/medieval-kingdom)

## 2019-10-21 星期一

### C# 预编译符号

* 在 Visual Studio `项目 -> 属性 -> 生成 -> 常规`，`条件编译符号`中填入预编译符号，即可让该预编译符号在整个项目中生效。

### 在 Visual Studio 修改项目的程序集名称

* 在 Visual Studio 中，修改了项目名称之后，其相应的程序集名称并没有自动改变，需手动在`项目 -> 属性 -> 应用程序`，修改`程序集名称`和`默认命名空间`。

### .NET 程序集之间相互调用

同一个解决方案下的不同项目之间，或者说不同程序集之间，虽然可以通过引用，让处于依赖层次上层的程序集可以调用到下层的程序集，但由于禁止循环依赖，处于依赖层次下层的程序集，是无法直接调用上层程序集的。

以下两种方法，可以一定程度上实现程序集之间的相互调用。
以上层程序集 A 依赖于下层程序集 B，但下层程序集 B 需要调用上层程序集 A 中的 Model.ClassA 为例：

* 接口与反射

    - 在下层程序集 B 中定义接口 InterfaceB，在上层程序集 A 中，让 Model.ClassA 实现该接口
    - 在下层程序集 B 中，使用反射机制，通过检索程序域中程序集 A，获取 Model.ClassA 对象类型，然后创建该对象实例
    - 在下层程序集 B 中，就可以使用 InterfaceB 引用来持有上层程序集 A 中 Model.ClassA 的对象实例，并通过接口类中定义的接口方法来主动调用对象中的具体实现方法

```csharp
System.Reflection.Assembly[] assemblies = AppDomain.CurrentDomain.GetAssemblies();
foreach (System.Reflection.Assembly assembly in assemblies)
```

* 委托（事件系统）

    - 让上层程序集 A 和下层程序集 B 均引用/依赖于另一个程序集 C，在程序集 C 中实现通用的事件系统并定义事件类型
    - 在上层程序集 A 中订阅/监听事件，该事件是由下层程序集 B 分发/发送的
    - 以此方式即可实现下层程序集 B 调用上层程序集 A 中的具体方法实现

### 动态加载程序集

* [How do I get all assemblies in the solution?](https://www.codeproject.com/Questions/1089179/How-do-I-get-all-assemblies-in-the-solution)
* [Dynamically pre-load assemblies in a ASP.NET Core(or any C#) project](https://dotnetstories.com/blog/Dynamically-pre-load-assemblies-in-a-ASPNET-Core-or-any-C-project-en-7155735300)

```csharp
// Put the right path to the assembly you are trying to load here
string path = @"..\..\ConsoleApplication11\bin\Debug\ConsoleApplication11.exe"; 
```

### 《提问的智慧》

* [How to ask questions the smart way? - GitHub](https://github.com/ryanhanwu/How-To-Ask-Questions-The-Smart-Way/blob/master/README-zh_CN.md)

## 2019-10-14 星期一

### C# 命名规范

* [Naming Guidelines - Capitalization Styles - Microsoft Docs](https://docs.microsoft.com/en-us/previous-versions/dotnet/netframework-1.1/x2dbyw72%28v%3dvs.71%29)

**Pascal case (帕斯卡命名法/大驼峰命名法) :**

The first letter in the identifier and the first letter of each subsequent concatenated word are capitalized. You can use Pascal case for identifiers of three or more characters.
For example: `BackColor`.

**Camel case (驼峰命名法/小驼峰命名法) :**

The first letter of an identifier is lowercase and the first letter of each subsequent concatenated word is capitalized.
For example: `backColor`.
 
**Uppercase**

All letters in the identifier are capitalized. Use this convention only for identifiers that consist of two or fewer letters.
For example: `System.IO` and `System.Web.UI`.

## 2019-10-10 星期四

### C# 委托与匿名函数

* [我所理解的委托和匿名函数 - UWA](https://blog.uwa4d.com/archives/2072.html)

> 因为刚才讲到，委托是一种类型，而函数并不是类型，所以通常情况下，如果没有为两种类型提供转型操作，编译器会报出类型不匹配的错误。所以，既然我们这种写法能够通过编译器，那么说明编译器为我们实现了转型操作，比如隐式用于转型的构造函数。（知识点：隐式显示转型构造函数）。
> 如果委托的函数非常简单，或者基本上没有重复利用的可能，那么可以直接把函数体实现写在委托的生成处，这样你就不用费劲心思去思考该怎样给函数取一个容易的名字了，名字压根就不重要了，函数体那几句代码更易于理解代码在干什么，这就是匿名函数，也就是叫做 lambda 表达式的东西。
> 所以要正确使用匿名函数，必须清楚以下几点：

```csharp
// 用ILSpy反编译出来的代码：
```

```csharp
// 函数功能：打印从0~9这几个数字
        // 假如满足几率很小

// ====== 反编译后的代码：

void Error1() {

// ====== 修正后的正确代码：

void Error1() {
```

* [GCFreeClosure - GitHub](https://github.com/lujian101/GCFreeClosure)

### Unity 编辑器工具制作

* [Unity 手游开发札记（从零开始搭建手游开发的工具集）- 知乎](https://zhuanlan.zhihu.com/p/24557713)
* [Unity 工具类系列教程（配置化和规范教程）- 知乎](https://zhuanlan.zhihu.com/p/30042447)
* [Unity 工具类系列教程（代码自动化生成）- 知乎](https://zhuanlan.zhihu.com/p/30716595)
* [Unity 工具类系列教程（对象池）- 知乎](https://zhuanlan.zhihu.com/p/30575559)

### UGUI 使用

* [UGUI 系列教程（UGUI 基础与界面拼接）- 知乎专栏](https://zhuanlan.zhihu.com/p/28905447)
* [UGUI 系列教程（监听事件，完成解谜）- 知乎专栏](https://zhuanlan.zhihu.com/p/28906086)
* [UGUI 系列教程（OSU 动态界面制作）- 知乎专栏](https://zhuanlan.zhihu.com/p/28906293)
* [UGUI 系列教程（OSU Battle）- 知乎专栏](https://zhuanlan.zhihu.com/p/28906798)

## 2019-10-08 星期二

### Unity I18N

* [Unity I18N 小探 - 知乎](https://zhuanlan.zhihu.com/p/81159633)
* [I2 Localization - Unity Asset Store](https://assetstore.unity.com/packages/tools/localization/i2-localization-14884)

* [游戏国际化的一些建议 - 硬盘在歌唱](http://disksing.com/game-i18n)

> * 不要为每个语言版本建立单独分支
> * 尽量不要针对不同地区写特殊代码
> * 源代码中不能出现汉字或直接用于显示的字符串
> * 服务器代码或配置中不能出现汉字或直接用于显示的字符串
> * 尽量不要使用带文字的图片
> * 设计结构化的字典配置，减少冗余
> * 不用使用 %d、%s 等标识文本中的变量
> * 注意多义词
> * 留意 UI 中文字的长度问题

### Unity 项目资源目录与命名规范

* [Mastering Unity Project Folder Structure. Level 0 - Folders required for version control systems - DEV](http://developers.nravo.com/mastering-unity-project-folder-structure-level-1-reserved-folders/#.XZ3XOI4zYUF)
* [Mastering Unity Project Folder Structure. Level 1 - Reserved Folders - DEV](http://developers.nravo.com/mastering-unity-project-folder-structure-level-1-reserved-folders/#.XZ3XOI4zYUF)
* [Mastering Unity Project Folder Structure. Level 2 - Assets Organization](http://developers.nravo.com/mastering-unity-project-folder-structure-level-2-assets-organization/#.XZ3XQY4zYUE)
* [Best Practices - Folder Structure - Unity Forum](https://forum.unity.com/threads/best-practices-folder-structure.65381/)
* [Unity 工程目录规范 - Moses's Note](https://mrsoong.com/posts/2018-04-01-3743db2c.html)
* [Unity 项目目录结构与命名规则 - 腾讯云](https://cloud.tencent.com/developer/article/1141251)
* [自动化规范 Unity 资源的实践 - UWA](https://edu.uwa4d.com/course-intro/0/121)

### Unity 项目资源制作规范

* [Unity 美术资源制作规范 - Joyimp](http://www.joyimp.com/?post=161)
* [3D 模型与动画，美术资源的规范 - 技术人生](http://www.luzexi.com/2018/08/03/Unity3D%E9%AB%98%E7%BA%A7%E7%BC%96%E7%A8%8B%E4%B9%8B%E8%BF%9B%E9%98%B6%E4%B8%BB%E7%A8%8B-3D%E6%A8%A1%E5%9E%8B%E4%B8%8E%E5%8A%A8%E7%94%BB1.html)
* [Unity 美术资源规范整理 - 程序园](http://www.voidcn.com/article/p-qjjvpian-bcw.html)
* [Unity3D 美术资源规范 - 腾讯游戏学院](https://gameinstitute.qq.com/community/detail/128388)
* [美术资源标准（纹理篇） - Unity Connect](https://connect.unity.com/p/mei-zhu-zi-yuan-biao-zhun-wen-li-pian)

### Unity 项目编码规范

* [Unity 之命名规范（一）- 阿里云](https://yq.aliyun.com/articles/666181/)
* [Unity 之命名规范（二）- 阿里云](https://yq.aliyun.com/articles/666180?spm=a2c4e.11153940.0.0.609c684f0u4nXw)

-------

