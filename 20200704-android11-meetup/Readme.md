- [What’s new in Android](#whats-new-in-android)
  - [嘉宾分享](#嘉宾分享)
    - [**Window Insets-布局的监听器**](#window-insets-布局的监听器)
    - [**聊天**](#聊天)
    - [**Bubbles**](#bubbles)
    - [**隐私**](#隐私)
    - [**5G**](#5g)
    - [**生物识别**](#生物识别)
    - [**NNAPI v1.3**](#nnapi-v13)
    - [**开发工具**](#开发工具)
    - [**Nullablity 注解**](#nullablity-注解)
    - [**崩溃原因报告**](#崩溃原因报告)
    - [**GWP-ASan 调试工具**](#gwp-asan-调试工具)
    - [**ADB 增量 APK 安装**](#adb-增量-apk-安装)
    - [**行为变更开关**](#行为变更开关)
    - [**现代 API**](#现代-api)
    - [**最新的组件**](#最新的组件)
    - [**paging 3.0 支持Kotlin coroutine**](#paging-30-支持kotlin-coroutine)
    - [**Hilt:基于Dagger的DI方案**](#hilt基于dagger的di方案)
    - [**Jetpack Compose**](#jetpack-compose)
    - [**工具-Android Studio**](#工具-android-studio)
      - [4.0稳定版：MotionLayout+Motion Editor](#40稳定版motionlayoutmotion-editor)
      - [4.0稳定版：Layout Inspector](#40稳定版layout-inspector)
      - [4.0 Beta：Database Inspector (Room, SQLite)](#40-betadatabase-inspector-room-sqlite)
      - [4.2：Canary](#42canary)
  - [**Q&A环节**](#qa环节)
  - [关于GDG](#关于gdg)
 

# What’s new in Android

**编辑：钟辉||校对：小赞||排版：Hayley**



在我们还没来得及反应的时候，2020年上半年就已经飞速过去了，虽然时间易逝，但是GDG活动不停，7月份第一场精彩活动已经在上一周圆满落幕。心心念念的线下活动如约而至，小伙伴们也纷至沓来。





<video src="./source/highlight.mp4"></video>



本期活动，来自 Android 团队的老师们带来了一场关于Android 11干货满满的分享，到场的小伙伴均表示收获颇丰，错过活动的小伙伴们也不要懊悔哦，贴心的小编在此还给大家整理出了嘉宾的分享内容以及现场Q&A环节的内容，望大家温故知新，及时复习哈~



**[点击此处可查看直播回放](https://www.bilibili.com/video/BV1Na4y1e7hL)**



## 嘉宾分享

大家好，我叫钟辉，我是Android团队开发成员，今天很感谢上海GDG组织者邀请我来参加的这次meet up活动。如果大家每一年有看过我们 Google I/O 活动的话，应该知道我们每一年都有类似的Android演讲，最近一年我们有什么更新呢？


### **Window Insets-布局的监听器**

![Window_Insets](.\source\Window_Insets.webp)

我们在设计布局的时候，布局文件中的某些元素，有时候被导航栏或者是状态栏遮住，有时候也会被弹出的键盘覆盖。在 Android 11 版本里面增加了新的 WindowInsets 回调函数`setWindowInsetsAnimatorCallback`，使用该回调函数可以更方便的处理 WindowInset 不同的状态。

我们来看下 WindowInsets 常用场景

1. 给 WindowInsets 设置监听，检测键盘的可见性。可以方便的根据软键盘状态调整布局文件里面的内容

![Window_Insets](.\source\Window_Insets2.webp)    

2. 监听键盘动画事件

![Window_Insets](.\source\Window_Insets3.webp)

3. 主动触发键盘动画

![Window_Insets](.\source\Window_Insets4.webp)

 

Sample链接地址：

https://github.com/android/user-interface-samples/tree/master/WindowInsetsAnimation

### **聊天**

![chat](.\source\chat.webp)

上图中通知栏可以看到一个聊天的场景，我们把人作为设计的中心，每个聊天对象的头像的 icon 都很明显。这是如何实现的呢，我们来看一下代码。

第一步先建立一个 Person 的类，一定要把这个 person 设置 `setLongLived(true)`

![chat](.\source\chat2.webp)

第二步使用 `ShortcutManagerCompat.pushDynamicShortcut(shortcutInfo)`

![chat](.\source\chat3.webp)

最后是设置 `MessagingStyle`

![chat](.\source\chat4.webp)    

这样就完成了创建一个包含 ShortcutInfo 的 Notification

### **Bubbles**

下图演示了 bubbles 的功能，有收缩和展开两种状态。

![bubbles](.\source\bubbles.webp)

在 Android 10 的时候，使用 Bubble 需要打开开发者模式，现在 Android 11 可以正常使用了。

 ![bubbles](.\source\bubbles2.webp)

Bubble 其实是一个基于 notification 的 API 做的扩展，他能在系统的任何一个界面被开启。

如果你是做一个聊天的软件的话，怎么样把你的聊天的互动变成一个 Bubbles 呢？我们来看几行代码。

第一步，首先你在 manifest 中声明一个 Activity，用户点了 bubbles 以后会开启这个 Activity

![bubbles](.\source\bubbles3.webp)

第二步，添加启动 Activity 代码

![bubbles](.\source\bubbles4.webp)

第三步，在 Notification 中添加 BubbleMetadata

![bubbles](.\source\bubbles5.webp) 

最后，创建和 Metadata 绑定的 Notification

![bubbles](.\source\bubbles6.webp)

Sample 链接地址：

https://github.com/android/user-interface-samples/tree/master/People



### **隐私**

隐私是 Android 11 的其中一个重点，Android 11 给用户带来了更佳的数据保护，主要体现在5个方面

![privacy](.\source\privacy.webp)

### **5G**

5G 的渗透率在全世界来说都不断提升了，每一个厂商每一个运营商都有他们相关的计划。

在 Android11 里面也有相应对开发者的支持，希望大家可以共同的探索出新的主题场景。

在 API 方面提供了相关功能：

- 可以了解用户的网络是不是要付费
- 估算用户网络带宽速度

我们来看下相关代码

![5G](.\source\5g.webp)

### **生物识别**

现在很多手机都有不同的生物识别功能，比如，3D人脸、虹膜等等，不同的方法是基于不同的底层技术和硬件设备的支持。从安全的角度，不同的技术有强弱的分别。通常来说3D人脸识别都是比2D的更加安全。

Android 11 里面提供标准的方法帮助开发者判断用户所设置的生物识别的安全级别。总共有3个级别，一个强，一个弱，还有是用户使用了数字或者 PIN 来设置解锁密码。

相关代码如下：

![bio](.\source\bio.webp)    

### **NNAPI v1.3**

神经网络API

![NN](.\source\nn.webp)    

在Android 11里面，提升 GPU，DSP ，NPU 的性能，如果懂这个领域的同学，肯定会想到不同的方法去利用这些新的操作，还有新的不同的方法去把自己的模型运作得更顺畅。

### **开发工具**

adb 使用 Wi-Fi 来连接设备，调试程序。目前来说我们稳定版本的 Android Studio 没有完善的支持，在 Canary 版本有相对完善的支持。

下图就是系统提供的设置的窗口，如果有 pixel 手机的话，可以尝试一下。

![devtool](.\source\devtool.webp)

### **Nullablity 注解**

在 Android 11 的 SDK 里面，为了方便开发者使用，调用不同的函数，还有提高 Java 语言和 Kotlin 的互用性，在 SDK 里面加上了一些新的 Nullability 注解。在这版本才加上的注解会命名为 `RecentlyNullable` 或 `RecentlyNonNull`。为了减少对应用兼容性的影响，违反了这类注解只会导致构建时的警告，不会影响构建。当下一版本发布时就会把这些 Recently* 注解迁移到 `@Nullable` 或 `@NonNull`。违反的代码就会导致构建错误。

![nullability](.\source\nullability.webp)    

### **崩溃原因报告**

在 Android 11 里面有一些新的 API ，可以帮助开发者收到有关应用闪退的错误报告了。有时候你用你的App闪退的时候，你会觉得无从入手，因为闪退的情况太多了，其中尤其是 ANR 是特别难抓的。在安卓11里面我们提供了一个新的API `App exit info`

![crash](.\source\crash.webp)    

其实它的功能很简单，在 App 闪退的时候，Android 系统里面会把闪退的原因写入到缓存里面去，后面用户再启动 APP 的时候，就可以再读取这个缓存里面有什么内容，从而可以帮助开发者追踪不同的问题的日志。

### **GWP-ASan 调试工具**

Native安卓上面写C++的一个工具，它可以帮助开发者检测在内存里面发现到的一些问题。

![gwp](.\source\gwp.webp)

如果想用这个功能的话，就可以在 manifest 里面的 Application 节点或者 activity 节点指定，代码如下图：

![gwp](.\source\gwp2.webp)

### **ADB 增量 APK 安装**

我们在 Android 11 里面增强了 adb 的速度，最大支持10倍加速

![apk](.\source\apk.webp) 

具体 adb 命令如下：

![apk](.\source\apk2.webp) 

### **行为变更开关**

Android 11 系统有一个新的系统的 UI，如下图，可以给开发者提供不同功能的开关，在做调试的时候，如果你只是想针对某一个功能做调试，就可以到这个页面，单独的开启或者关闭某个功能，从而帮助开发者定位问题。

![active](.\source\active.webp)

也可以使用 `adb command` 来开启相关的开关。

![active](.\source\active2.webp)

### **现代 API**

![modern](.\source\modern.webp) 

### **最新的组件**

![jetpack](.\source\jetpack.webp)

### **paging 3.0 支持Kotlin coroutine**

![jetpack](.\source\jetpack2.webp) 

### **Hilt:基于Dagger的DI方案**

Hilt尽量的帮助开发者减少dagger的样板代码。

![jetpack](.\source\jetpack3.webp) 

### **Jetpack Compose**

暂时还在预览版，还在添加一些新的功能，不建议在生产环境中使用

 ![jetpack](.\source\jetpack4.webp) 

### **工具-Android Studio**

####  4.0稳定版：MotionLayout+Motion Editor

![as](.\source\as.webp) 

#### 4.0稳定版：Layout Inspector

对布局提供一个3d的可视化工具

![as](.\source\as2.webp)

#### 4.0 Beta：Database Inspector (Room, SQLite)

直接读取数据库中的数据，方便开发过程中进行数据库方面的调试

![as](.\source\as3.webp)
 
#### 4.2：Canary

![as](.\source\as4.webp) 

![as](.\source\as5.webp) 


今天的分享就到到此为止，谢谢大家。

## **Q&A环节** 

**木木 问：**

**安卓10将私有目录下文件的可执行权限禁掉了，对于终端类和服务器类app产生了很大影响，后续有什么解决方案吗？这类应用还有未来吗？**

陈卓 答：

其实这个问题我之前也不是太了解，我刚才在网上去查了一下，发现确实有和开发者提这个问题，而且他们在谷歌的公开的 Bugzilla 里面去提交了一个 bug，这个 bug 也是有相应的工程师去做了回复。

总的来说，这个是由于我们在安卓10的时候做了一些 Linux 上面的一些保护，其实这个问题就是这么设计的，总体来说我们觉得可执行的代码只应该是从你的 Apk 里面来进行加载和执行。

如果你要用那个方法去随意的执行一些可行代码的话，这个是我们所不鼓励的，而且有可能在将来的安卓版本中，你现在的做法也是会被禁止的。在 Google Play 我们也是有相应的政策，就是不允许去动态的下载和加载可执行的代码。

因为这个是一个安全性挺大的影响，所以有了这么一个新的确定性。其实有一个我看到可以暂时解决的方法，如果你的可行性代码确实是包含在 Apk 里面的话，那么你可以把它放在你的 Apk 的在 nativelib 目录里面，然后在 mainfest 写上 `extractnativelibs = true`，这样的话这些可执行文件会被解压到 data/app 那个目录里面，在这个目录里可执行的文件是可以用去把它加载起来。

总的原则就是你的所有的可执行代码都应该在你的Apk里面，这是为了安全性的考虑。



**马萍 问：**

**权限变更对用户隐私有什么影响？**



陈卓 答：

其实我的理解是这样的，就是说我们在做安卓11的一些行为的变化和设计的时候，用户隐私是一个非常大的主题，围绕着这个主题我们决定要做的一些对的事情，然后在这些用户隐私方面做了一些相应的变化之后，我们再去做一些权限上的调整和增加，来去配合用户隐私方面的一些行为变更。比如说我们在安卓10和11里面引入了新的 Scoped Storage 之后，我们就相应的对之前的一些跟存储相关的权限。

比如说 `READ_EXTERNAL_STORAGE`，本来这个权限是可以读取整个外部存储，目前这个权限调整成可读取 MediaStore 里面的东西。对于一些存储管理的那种应用，那么我们就相应的增加了一个叫 `MANAGE_EXTERNAL_STORAGE` 权限。

相应的比如说对应用包可见性，为了允许那种像第三方的用市场这样的，它真的需要手机里面装了哪些应用，我们也相应的增加了 `QUERY_ALL_PACKAGES` 权限。



**枯燥 问：**

** Jetpack 有一个正在开发的 UI 工具库 Compose，关于它我想问的两个问题：它和 Flutter 原理之间有什么区别，和以后会不会颠覆常规的安卓开发模式。**



钟辉 答：

 Compose 这块现在 developer preview 的一个阶段。这个问题的其中一个部分是从编程的模型来说，如果有用过 flutter 的同学知道，这两套的UI组件都是基于同一套的反应式的编程模型，还有类似 further 的，比如 React Native 等等的那些UI框架也是用类似的编程模型的，那么我们在设计 Compose 的过程，我们在市面上看哪些编程模型比较受欢迎，哪一些技术上比较牛，所以我们把类似的编程模型的理念放进去。

然后另外一个问题，是说日后在安卓里面做 xml 布局，或者是类似的一些 library，比如 data binding 等等会不会受到影响？

是会的，因为如果有尝试过 Compose 或者是 Flutter 的同学应该知道，它不是基于一个类似 layout 到 xml的一个机制，他没有用这个机制。Compose 是需要你写 kotlin 代码定义布局。

目前来说我们对的 Compose 的发展跟 Google 对 Flutter 的发展两者是并行的。Flutter 优势在于跨平台的支持，Compose 的优势是在安卓原生的开发，以后当 Compose 进入稳定期的，应该很方便的通过 kotlin 来调用不同的 Compose 特性。

但是目前来说，我并不建议大家开始使用 Compose 在自己的 Production App 里面。

compose release 的时间点目前来说的最快的时间点是下一年。



**Petterp 问:**

**Android10 之后对于设备唯一码怎么确定呢，现在的方案都是自己根据设备信息生成，有重复的几率**



陈卓 答：

其实我们从安卓10开始已经越来越限制设备Id的使用，其他的初衷也是一个隐私性的考虑，就是说我们希望应用不要去通过长时期的去追踪这个用户，来达到侵犯隐私的可能性。所以其实我们在最近的几个Android版本中，特别是对于那种跟硬件相关的不可重置的ID都做了非常严格的限制，基本上现在第三方应用都是没有办法去拿到这样的id，因为这种id一旦拿到以后，用户也没有办法去重制了，你就可以一直追踪这个用户。我们在安卓的开发者的官方的文档上面，其实是专门有一篇文档是说在使用identify方面的一些最佳实践，大家可以去看一下，总体来说我们还是鼓励开发者来想一想自己的应用，为什么一定要去追踪一台设备，而不是说要追踪到我自己的用户就可以了。

如果大家真的有这种特别需要追踪设备的这种需求的话，其实也欢迎大家来跟我们去沟通和探讨，我们来一起想一下，就这个需求是不是一个真正的必须的需求。总体来看，根据这两三年我们引入这些唯一代码的限制，基本上我们没有看到，真的非常强的说一定要去追踪设备的这样的一个特别合理的需求。



**VERTUONLY 问：**

**对热更新机制是否会进行限制**



陈卓 答：

其实这个问题基本上我们在每年的活动里都会有人问到？

其实是这样的了，热更新其实是对可执行代码的热更新是违反Google Player的政策的，所以在Google Player不可以做任何对于可执行代码的热更新，但Google Player是允许你对资源或者对一些Assets来做热更新。

但是我们也非常了解在国内的环境下，因为国内的这种分发渠道是比较碎片化的，所以热更新是一个很多开发者都常用的解决方案，所以我们在安卓的系统平台的级别是没有对热更新做任何限制的。

我们其实也是跟国内的一些常见的热更新的厂商都保持紧密的联系，因为大家在做热更新的时候会常常用到一些私有API，因为它是私有的，所以我们会在经常在新的安卓版本中，在没有通知开发者的情况下，就给它做大幅的修改，所以会导致很多热身新的方案在新版本的安卓中就会被break。

我们会尽早的告诉国内热更新的厂商，我们安卓的新版中哪些私有API方面的变化，来敦促他们尽早的去做适配，这样给使用热更新的应用更多的时候来做适配。而且我们也会和热更新的厂商去做一些合作，他们对系统平台级别有什么样的需求，我们就有可能会开放出一些新的公开的SDK接口，来让这些热更新厂商做到更好的适配性。

比如说我们在去年的时候，在安卓10中，我们引入了一个新的接口，这个接口能够让一个应用在刚起来的时候就去设置一个自定义的类加载器。

还有我们在今年Android 11里面有一套新的对于动态加载资源的API叫做 resources loader。我们也有一些热更新厂商都已经开始在试用resources loader，而且效果还不错，也欢迎大家如果对于热更新特别是资源方面的热更新，有什么需求的话，可以去研究一下我们新的API。


> 本文所提及的所有链接
> [bubbles：https://github.com/android/user-interface-samples/tree/master/People](https://github.com/android/user-interface-samples/tree/master/People)
>
> [Window Insets：https://github.com/android/user-interface-samples/tree/master/WindowInsetsAnimation](https://github.com/android/user-interface-samples/tree/master/WindowInsetsAnimation)
>
> [paging：https://github.com/googlecodelabs/android-paging](https://github.com/googlecodelabs/android-paging)
>
> [Hilt：https://github.com/googlecodelabs/android-hilt](https://github.com/googlecodelabs/android-hilt)



## 关于GDG

Google Developer Groups 谷歌开发者社区，是谷歌开发者部门发起的全球项目，面向对 Google 和开源技术感兴趣的人群而存在的公益性开发者社区。GDG Shanghai 创立于 2009 年，是全球 GDG 社区中最活跃和知名的技术社区之一，每年举办 30 – 50 场大大小小的科技活动，每年影响十几万以上海为中心辐射长三角地带的开发者及科技从业人员。

![gdg](.\source\gdg.jpg)  

社区中的各位组织者均是来自各个行业有着本职工作的互联网从业者，我们需要更多新鲜血液的加入！如果你对谷歌技术感兴趣，业余时间可调配，认同社区的价值观，愿意为社区做出贡献，欢迎加入我们成为社区志愿者！



**志愿者加入方式：关注上海 GDG 公众号：GDG_Shanghai，回复：志愿者。**

**社区成员加入方式：请发邮件至以下邮箱**

 **gdg-shanghai+subscribe@googlegroups.com**

