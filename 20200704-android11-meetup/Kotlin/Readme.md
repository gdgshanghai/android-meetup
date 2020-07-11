- [上海 20200704 GDG Kotlin Coroutines](#上海-20200704-gdg-kotlin-coroutines)
  - [AsyncTask 的废弃](#asynctask-的废弃)
  - [Kotlin 协程](#kotlin-协程)
  - [协程的特点](#协程的特点)
    - [轻量高效](#轻量高效)
    - [简单好用](#简单好用)
  - [探究协程魔法](#探究协程魔法)
  - [协程作用域](#协程作用域)
    - [**Dispatcher**](#dispatcher)
    - [**Job**](#job)
  - [调用自定义耗时任务](#调用自定义耗时任务)
  - [协程是结构化的](#协程是结构化的)
  - [协程取消的注意点](#协程取消的注意点)
  - [处理协程的异常](#处理协程的异常)
    - [传统方式](#传统方式)
    - [全局捕获方式](#全局捕获方式)
  - [协程失败时的行为](#协程失败时的行为)
  - [总结](#总结)
  - [Q&A](#qa)


# 上海 20200704 GDG Kotlin Coroutines

本文记录了在 Android 11 上海 Meetup GDG 上，郭霖关于 Kotlin Coroutines 的分享内容。 
## AsyncTask 的废弃

![AsyncTask](/20200704-android11-meetup/Kotlin/Readme.assets/1.png)

在 Android 11 中 `AsyncTask` 已经被废弃。 AsyncTask 可以说是伴随了众多 Android 开发者的成长，AsyncTask 是在 Android 1.5 中被引入的，现在却从 Android 11 当中废弃了。为什么在最新的 Android 11 将 AsyncTask 这个类废弃了？这是因为在 Android 11 中，有了一种更加方便的异步任务处理方式，也就是我们今天要介绍的主题 Kotlin 协程。 

## Kotlin 协程

Kotlin 协程已经成为了谷歌官方推荐的异步任务处理方式。协程的英文单词是 *Coroutines*，Coroutines 这个单词实际上是一个组合单词，它是由 *Co* + *routines* 组合而成的。*Co* 在这里指的是 cooperation (协作)，*routines* 在英文当中表达的意思是叫例行日程。利用协作的方式去帮助我们完成例行日程，就是协程的含义。

当把 *routines* 映射到编程语言当中时，就可以理解为编程语言中的函数。可以将它理解成是一种协作式的函数调用模型。以上就是协程比较广义的概念，不过协程不是 Kotlin 独有的，很多编程语言上都会有协程，在不同的编程语言上，协程的实现都有所差异。 

## 协程的特点

### 轻量高效 

> One can think of a coroutine as a light-weight thread. Like threads, coroutines can run in parallel, wait for each other and communicate. The biggest difference is that coroutines are very cheap, almost free: we can create thousands of them, and pay very little in terms of performance. True threads, on the other hand, are expensive to start and keep around. A thousand threads can be a serious challenge for a modern machine. 

上面是 Kotlin 官方文档上的一段描述，从中可以看出，协程是非常轻量的，而轻量通常就意味着高效。但是在客户端开发中，基本没有需要创建成千上万线程的并发场景，最多就是在加载很多图片时多开几个线程。所以官方提到创建成千上万协程的场景，客户端开发一般无法遇到。 

### 简单好用

如果轻量高效不足以成为使用协程的理由，那么简单好用则足以。

**用同步的方式编写异步代码** 

协程允许开发者使用同步的方式来编写异步代码，接下来的例子将会体现如何使用同步的方式来编写异步代码。

![img](/20200704-android11-meetup/Kotlin/Readme.assets/2.png)

在平时的开发中，如果提到异步，一般都会联想到网络请求，因为网络请求都是要在异步中进行的，而提到网络请求，就无法绕开一个开源库 - Retrofit。Retrofit 是由 Square 公司开发的一款非常出色的开源网络库。它改变了网络请求的传统编程思维。没有 Retrofit 之前发起一个网络请求，需要借助 HttpURLConnection ，然后要封装请求的地址，还要设置请求类型，设置请求的连接超时、请求的读取超时，直到发起网络请求之后，还要通过流的方式去读取网络的数据响应，等把流读取完了之后再解析流中的内容，最后再想办法，把数据按照某种格式进行解析，一系列的步骤非常复杂。有了 Retrofit 之后，它完全改变了这种传统的网络请求方式。现在编写网络请求，只要使用一种类似调用函数的方式来进行网络请求和交互。

以下代码就是 Retrofit 的标准用法：
```kotlin
// GET http://example.com/get_user?user_id=userId
interface ApiService {
    @GET("get_user")    
    fun getUser(@Query("user_id") userId: String): Call<User> 
}
```

首先定义请求的接口函数，具体使用的时候：取动态代理对象，通过代理对象调用定义的接口函数，就可以发起这样一条网络请求了。
```kotlin
fun getAndShowUser() {
    val apiService = getApiService()
    apiService.getUser(userId).enqueue(object : Callback<User> {
        override fun onFailure(call: Call<User>, t: Throwable) {            
            // handle failure        
        }              
override fun onResponse(call: Call<User>, response: Response<User>) {
        val user = response.body()            
        showUser(user)        
        }    
    }) 
}
```
Retrofit 在处理网络请求的时候，大部分的场景下都是极其优雅的，但是唯独在一个场景下显得不那么优雅。

例子代码中 getUser 函数的返回值是一个 `Call` 类型的对象，在 `Call` 类型的对象基础上再调用`enqueue` 函数将请求加入到的 Retrofit 的请求队列当中，然后 Retrofit 就会开启一个 IO 线程，在异步的情况下去进行网络请求。 

因为这个请求是异步发起的，不能直接在它的下一行代码就得到网络请求的响应结果，所以在这里就需要借助**回调**来实现。在 `enqueue` 函数当中，借助了 `Callback` 的匿名实现类去接收网络响应的结果。在 `Callback` 的匿名实现类当中，重写了两个方法，`onFailure` 方法表示网络请求响应失败时或者其他原因失败时的回调， `onResponse` 方法表示请求成功之后的回调，在其中调用 `response.body` 方法就可以拿到服务器响应的 User 对象。从服务器响应到 User 的过程是 Retrofit 自动解析的。最后调用 `showUser` 函数，将服务器响应的数据展示到界面上。

以上就是使用 Retrofit 的传统编程方式，上面提到过 Retrofit 最大的突破点在于帮助我们以一种调用函数的方式来发起网络请求，但是当你编写了长长的一段回调之后，你会发现这个方式就已经不再像是简单调用函数了。

在过去没有办法解决异步回调问题，是因为异步发起的任何请求任务时，必须要通过回调才能接收响应，但是现在，Retrofit 从 2.6.0 版本开始内置了对协程的支持。如果你的项目当中使用的是 Retrofit 2.6.0 以上版本，那么在 `ApiService` 接口声明当中，可以在函数的前面加上 `suspend` 关键字（`suspend` 关键字是 Kotlin 语言协程当中内置的一个关键字），加上 suspend 关键字之后，`getUser` 函数的返回值就不需要再声明 `Call` 类型，而是可以直接声明成 `User` 类型。 

```kotlin
interface ApiService {
    @GET("get_user")
    suspend fun getUser(@Query("user_id") userId: String): User
}
```

 一个函数如果被声明为 suspend，这个函数就变成了挂起函数。一个挂起函数，它只能在另外一个挂起函数或者是在一个协程作用域当中才能调用。所以这里 `getAndShowUser` 函数不能直接调用 `getUser` 函数。而是需要给 `getAndShowUser` 函数加上 suspend 关键词将它也变成挂起函数。

```kotlin
suspend fun getAndShowUser() {
    val apiService = getApiService()
    ...
}
```

增加了 suspend 关键字之后，就能对回调的部分进行简化。 

```kotlin
suspend fun getAndShowUser() {
    val apiService = getApiService()
    val user = apiService.getUser(userId)
    showUser(user)
}
```

刚才很长的一段回调代码，经过简化之后，现在就只剩下简单的两三行代码。第一行代码没有变动，还是获取动态代理对象，第二行代码现在是直接调用了 `getUser` 函数，不需要再去借助回调的方式获取服务器的响应，而是直接可以得到响应的 User 对象。

经过这样的优化之后，网络请求真正变成了一种类似于函数请求的方式，就好像是在调用一个函数获取它的返回值一样简单。 

## 探究协程魔法

一个普通函数通常支持两种操作：*call and return。*因为每个函数都支持调用，同时也支持获取它的返回值。当在一个函数上声明了 `suspend` 关键字之后，这个函数就会支持两种额外的操作：*suspend and resume*。*suspend* 是指函数可以在一个特定的时间点被挂起（就好像函数突然停止了运行），然后它会被存储到某一个状态，暂停它的运行，*resume* 指的是可以恢复之前挂起函数的状态，让它从当时被挂起的地方，继续向下执行。

![img](/20200704-android11-meetup/Kotlin/Readme.assets/3.png)

上面的例子当中，当调用了 `apiService.getUser` 函数的时候，Retrofit 会开启一个 IO 线程，在 IO 线程中进行网络请求，同时它也会将当前 `getAndShowUser` 函数的协程给挂起，当前 `getAndShowUser` 协程就不会再继续运行了，而是停留在了 `apiService.getUser` 这一行代码。当前的协程被挂起之后，其他的协程就会得到机会去运行。如果某个线程下没有任何协程正在运行，那么这个线程就可以继续运行其他代码，也就是说当前 `getAndShowUser` 协程被挂起之后，主线程就可以得到正常运行。所以不管 Retrofit 网络请求花费了多长的时间，都完全不会阻塞主线程，主线程可以正常响应任何的点击、触碰等事件。 

当网络请求结束之后，Retrofit 会将之前挂起的协程恢复，恢复到刚才 `apiService.getUser` 被挂起的位置。得到服务器响应的数据之后，将它复制到 user 对象上，然后代码就可以恢复正常的继续下一步运行。再继续调用 `showUser` 函数，将数据显示到界面上。

上面提到过：一个挂起函数只能在另外一个挂起函数或者在一个协程作用域当中去调用，那么现在如果将 `getAndShowUser` 函数声明成了一个挂起函数，该怎样调用它呢？可以再定一个挂起函数去调用它，但是这样的话就会陷入无限循环，所以总需要有一个入口能去调用挂起函数。

## 协程作用域

协程作用域就是调用挂起函数的入口。在 Kotlin 当中创建协程主要有两种方式，分别是 `launch` 和 `async` 两个函数，`launch` 是较通用的一种方式，它这个理念更类似于是一种叫 *fire and* *forgot* 的方式，我创建了你，然后你就去运行吧，之后就不管你了，有点像创建线程的方式。然后 `async` 函数则不同了，`async` 函数和 `launch` 函数一样，也会创建一个协程，但是它会有一个 Deferred 类型的返回值。

![img](/20200704-android11-meetup/Kotlin/Readme.assets/4.png)


调用 `async` 函数代码会执行，之后可以调用 `deferred.await` 函数来获取函数执行的返回结果。async 会开启协程去执行代码块里的代码，同时代码块最后一行代码会作为返回值返回，可以调用 `await` 函数来去获取返回的返回值。如果调用 `await` 函数的时候，协程还没有运行完，调用 `await` 函数的协程就会被挂起，一直等到 async 函数执行结束之后， `await` 函数才会重新被恢复。

主要可以通过这两个函数可以创建一个协程，但是这两个函数都不可以直接调用，而是要在协程作用域当中才能调用，同时在挂起函数中也不可以调用，所以还有一个核心的问题要解决，就是如何创建一个协程作用域，创建协程作用域的方式有很多种，但是这里不推荐其他那些使用方式，因为建议的方式就只有一种： `CoroutineScop`。除了这个函数之外，如果你比较了解协程，还会知道有 v*iewModelScop* 的或者 l*ifecycleScop* 这些方式，也可以创建协程作用域，这些方式是推荐使用的，但是他们都是在 `CoroutineScop`的基础上做的封装。

```kotlin
val scope = CoroutineScope(Dispatchers.Main + Job())
```

`CoroutineScope` 函数的参数列表中有一个叫 CoroutineContext 的参数，可以将 CoroutineContext 简单理解成是一种 Set 集合，又因为 CoroutineContext 里面重载了加号运算符，所以多个 CoroutineContext 元素之间可以使用加号来连接。

上面的示例代码中，`Dispatchers.Main` 和 `Job` 其实都是 CoroutineContext 对象。 

### **Dispatcher**

Dispatcher 用于告知协程，应该在哪个线程当中去运行。`Dispatchers.Main` 就是在 Android 的主线程当中运行，除了 `Main` 之外，我们还可以指定：
- `Dispatchers.Default` 开启低并发的子线程，去执行一些计算密集型的操作
- `Dispatchers.IO` 开启高并发的子线程，然后去进行一些阻塞密集型操作 

### **Job**

`Job` 是作为协程身份唯一标识的存在，每一个协程内部都会有一个唯一的标识。通过 Job 对可以控制协程的生命周期，比如：
- 可以判断协程是否正在运行
- 可以判断协程是否已经被取消
- 可以判断协程是否运行结束 

得到一个  CoroutineScope 对象之后，接下来就可以调用 `scope.launch` 函数去创建一个协程，创建了一个协程之后， `launch` 会赋予一个协程的作用域，有了协程作用域，就可以在作用域里去调用刚才的 `getAndShowUser` 函数。

```kotlin
val scope = CoroutineScope(Dispatchers.Main + Job())
scope.launch {
    getAndShowUser()
}

suspend fun getAndShowUser() {    
    val apiService = getApiService()    
    val user = apiSerivce.getUser(userId)    
    showUser(user)
}
```

除了使用 Retrofit 之外，其他耗时任务也可以通过协程来简化。

## 调用自定义耗时任务 

下面的示例，定义一个 `getUserFromDb` 函数，从数据库中获取数据。由于数据库的读取操作也是一个耗时操作，所以需要将它放到一个子线程当中去运行，否则它会阻塞当前的主线程，影响到界面的交互。

```kotlin
suspend fun getUserFromDb(): User {    
    return withContext(Dispatchers.IO) {        
        val user = // load user from db        
        user    
    }
}

suspend fun getAndShowUser() {    
    val user = getUserFromDb()    
    showUser(user)
}
```

这里借助了 `withContext` 函数来实现， `withContext` 也是协程当中内置的一个非常常用的函数。在 withContext 参数当中指定了 `Dispatchers.IO`，`withContext` 函数代码块当中的代码就都会在子线程当中运行，代码块中就可以去执行任意的耗时操作，然后当 withContext 运行结束之后，它的最后一行代码会作为返回值返回，然后拿到返回的 User 对象，并且直接去 return withContext 函数，这样 `getUserFromDB` 函数就可以获取到从数据库读取的 User 对象。

在这个示例中，虽然有异步的操作，但是没有异步的回调，都是使用类似于编写同步代码的方式来实现的功能。

*目前为止，协程的基本用法已经了解完了，接下来继续探究协程更多的特性。* 

## 协程是结构化的 

上面有提到 `launch` 函数是需要在协程的作用域当中才能调用，同时 `launch` 函数它又会创建一个协程的作用域，那是不是意味着可以在 `launch` 当中继续调用 `launch`，然后再继续调用 `launch` 函数呢？

```kotlin
val scope = CoroutineScope(Dispatchers.Main + Job())
scope.launch {
    launch {
        launch {

        }
    }
    launch {

    }
}
```

上面这段示例代码是合法的，协程支持嵌套调用，但是协程的嵌套调用和线程又不一样，协程的嵌套调用是有父子结构的，也就是说这里顶层的 `scope.launch` 函数首先创建了一个协程，然后在它的协程下面又创建了两个子协程，这两个子协程是有父协程概念的。在第一个子协程当中又创建了一个子协程，这是它的父协程，就是刚才创建的第一个子协程。而反观线程，他们之间是没有任何父子关系的，不管在线程当中开启了多少个线程，都是一个个独立的子线程，跟创建它的外层线程之间没有任何的关联。

协程的这种特性又被称作为结构化并发，这种结构化并发的特性让整个协程变得非常**利于管理**。

比如现在有一个 Activity 界面，要发起网络请求或者执行很多耗时的逻辑操作。然后在这些逻辑操作还没有完成的情况下，Activity 被用户关闭，那么现在进行的一些逻辑操作其实没有必要再进行下去的。因为进行了也没有办法将结果反馈到界面上，所以这种情况下最佳的解决方案是将这些正在运行的逻辑全部都取消掉。

在使用线程时，如果你创建了很多个线程，想要管理他们是非常困难，因为需要追踪到每一个线程，然后想办法将他们一一取消，而协程的结构化并发特性使得管理协程变得非常简单：只需要调用最顶层协程的 `cancel` 函数，就可以将它下面的所有子协程一起取消掉，非常方便。

调用顶层的 `scope.cancel` 函数来去取消所有的协程，下面的绿框里表示的就是能取消的协程的范围。

![img](/20200704-android11-meetup/Kotlin/Readme.assets/5.png)

如果现在只想要取消部分的子协程怎么办？注意：每一个 `launch` 函数它都会返回一个 Job 对象，Job 是一个协程的唯一标识，所以 `launch` 函数返回的 Job 对象其实就是用于标示它的唯一 ID。有了 Job 对象之后，你就可以在任何地方调用 `job.cancel` 函数，只会取消掉当前协程以及它自己下面的所有子协程，而它外面的父协程，还有它的兄弟协程不会受到影响。

![img](/20200704-android11-meetup/Kotlin/Readme.assets/6.png)

## 协程取消的注意点

如果你认为协程因为有结构化并发的特性，只要调用一下 `scope.cancel` 函数，就可以让协程帮我们自动完成所有协程的取消，那你就大错特错 --- 因为协程的取消是需要协作完成。通过一个具体的示例来进行解释，下面定义了一个 `doSomethingHeavy` 函数：

```kotlin
suspend fun doSomethingHeavy() {
    // logic before withContext
    withContext(Dispatchers.IO) {
        // do heavy logic
    }
    // logic after withContext
}
```

在这个函数当中先执行了一些逻辑，有注释标识。然后在调用完这一段逻辑之后，开始执行 `withContext`。如果当我在执行 *logic before withContext* 的时候，当前的协程被取消掉了，`withContext` 函数在执行之前会检查协程是不是还在运运行状态，如果发现当前协程已经取消了，`withContext` 代码块当中的代码就不会得到执行，并且 `withContext` 之后的 *logic after withContext* 也不会执行。

但是如果我们 *logic before withContext* 运行完了，已经进入到 `withContext` 代码块里面，这个时候我们的协程被取消了，那么 `withContext` 代码块中的代码一定会全部执行完，我们的 `cancel` 函数是不会影响到 `withContext` 代码块中代码执行的。然后 `withContext` 这个函数会帮我们在协程结束之后再做一次检查，看看当前的协程是不是还在正常运行，如果不在正常运行，那么 *logic after withContext* 这部分的代码就不会继续执行。这个是因为 `withContext` 帮我们做这样的代码检查，所以它才能够比较好的帮我们完成协程取消的操作。如果现在执行的代码并没有调用 `withContext`，或者是在 `withContext` 函数的代码块当中循环去执行一段非常耗时的逻辑操作，那我们的取消是没有办法正常实现的，需要代码块中的代码全部执行完，才有办法取消。

```kotlin
suspend fun doSomethingHeavy() {
    // logic before withContext
    withContext(Dispatchers.IO) {
        for (file in files) {
            // do heavy logic
        }
    }
    // logic after withContext
}
```

考虑下面的示例代码，在`withContext` 的代码块中循环遍历文件。假设在执行到第二次的循环时，当前协程被取消，希望后面的循环不要再继续运行了，但是实际上并不会，只有整个耗时的循环，每一个 file 遍历执行完之后，协程才能得到取消。所以为什么说协程的取消是需要协作是完成的，我们需要清楚它的取消机制，并且在每次执行耗时逻辑之前都来做一次协程是否还处于运行状态的检查。

```kotlin
suspend fun doSomethingHeavy() {
    // logic before withContext
    withContext(Dispatchers.IO) {
        for (file in files) {
            ensureActive()
            // do heavy logic
        }
    }
    // logic after withContext
}
```

示例中使用了 `ensureActive` 函数来帮我们检查协程是否还处于运行状态。 

## 处理协程的异常

### 传统方式

try-catch 是传统的异常处理方式。如果在一个 `launch` 函数当中创建了一个协程，然后去使用 try-catch 来去捕获协程当中的异常，这种方式和平时编写代码使用 try-catch 的方式是完全一样的，可以捕获到协程里面逻辑出现的异常。

```kotlin
launch {
    try {
        // do something
        throw Exception("unhandled exception")
    } catch (e: Exception) {
        // caught exception
    }
}
```

现在将这种捕获的方式来稍微换一种写法，把 try-catch 移到 `launch` 函数的外面，然后捕获协程当中的异常，这种方式一定是捕获不到的。

```kotlin
try {
    launch {
        // do something
        throw Exception("unhandled exception")
    }
} catch (e: Exception) {
    // caught exception
}
```

因为 `launch` 函数它并不是调用它之后，整个代码就会阻塞在这里不动，而是会继续向下执行的。当继续向下执行之后，很快就会超出 try-catch 的作用域，而超出作用域之后，如果 launch 内部的代码块 throw 一个 Exception，try-catch 就不可能捕获。

如果再换一种更加特殊的场景，使用 `async` 函数，然后在里面 throw 一个 Exception，并且 `deferred.await` 这个方法是在 try-catch 作用域当中的，那么这种方式能够捕获到协程的异常吗？不一定，有些情况的话能够捕获到，而有些捕获不到。

```kotlin
try {
    val deferred = async {
        // do something
        throw Exception("unhandled exception")
    }
    deferred.await()
} catch (e: Exception) {
    // caught exception
}
```

总结一下协程的异常处理：协程内部的异常通过传统的 try-catch 方式捕获没有问题，但是永远不要去做跨协程的异常捕获。 

### 全局捕获方式 

除了传统的 try-catch, 协程还提供了一种全局捕获异常的方式：*CoroutineExceptionHandler*。

```kotlin
val handler = CoroutineExceptionHandler { coroutineContext, throwable ->
    // caught exception
}
```

调用 CoroutineExceptionHandler 函数，并且给它传递一个 lambda 表达式，然后在 lambda 中接收异常的返回信息，lambda 中的 throwable 参数就是具体抛出的异常。可以将 CoroutineExceptionHandler 应用到 CoroutineScrop 函数当中，因为 CoroutineExceptionHandler 实际上也是一个 CoroutineContext，所以它可以用加号进行连接。

```kotlin
val scope = CoroutineScope(Dispatchers.Main + Job() + handler)
```

比如说使用 ViewModelScope 时，是没有 CoroutineScope 编写权限的，这种情况下可以在调用 launch 函数时加入 handler。

```kotlin
viewModelScope.launch(handler) {}
```

如果调用 scope.launch 其中的一个子协程，在子协程 launch 时并传入了 handler，能捕获到吗？这种情况下是捕获不到的，CorountineExceptionHandler 只能放到顶层协程当中，所以在子协程当中不要使用它。

```kotlin
scope.launch {
    launch(handler) {
    }
}
```

## 协程失败时的行为

接下来我们再来讨论一下协程失败的行为。什么是失败？--- 就是当程序出现了未捕获的异常时就叫做失败。但是如果你没有使用，刚才介绍 CorountineExceptionHandler 将异常捕获，那么你的程序就会崩溃，崩溃的情况下也就谈不上失败。所以前提是要将异常捕获住，然后才能谈协程失败的行为。那么当一个协程失败的话，它会做出什么样的反应？

![](/20200704-android11-meetup/Kotlin/Readme.assets/7.png)


首先失败事件会冒泡到上一层，也就是 Parent 的这层协程当中，然后 Parent 层会先将自己的子协程全部 cancel 掉，接着再将自己 cancel 掉，最后再将这个事件继续冒泡到上一层。可以简单理解：假如一个协程失败的话，它的整个协程栈，所有的协程全都会被取消。

那么取消会造成什么样的后果呢？一个协程一旦它被取消之后，它就无法再次 launch，所以这对程序来说可能是一个比较大的影响，避免全部取消的方法就是使用 SupervisorJob 。SupervisorJob 和之前提到的 Job 类似，只不过它有一个额外的行为：如果一个子协程失败时，SupervisorJob 不会对子协程或者它自己做任何其他的处理，你自己失败就可以了，这个是 SupervisorJob 它的作用。接下来我们来看一下它的用法，很简单，其实就是将我们刚才 Job 的部分替换成 SupervisorJob。

![](/20200704-android11-meetup/Kotlin/Readme.assets/8.png)

```kotlin
val scope = CoroutineScope(Dispatchers.Main + SupervisorJob() + handler)
scope.launch {
    throw Exception("coroutine failed")
}
scope.launch {
  
}
```

然后我再跟大家演示一段程序，现在的话我换成了 SupervisorJob  之后，我们使用 CoroutineScope launch 两个协程，那么我在第一个协程里面去 throw 了一个 Exception，那么第二个协程会受到影响吗？--- 答案是不会的。

```kotlin
val scope = CoroutineScope(Dispatchers.Main + SupervisorJob() + handler)
scope.launch {
    launch {
        throw Exception("coroutine failed") 
    }
    launch {
    }
}
```

现在如果我在 scope.launch 里面 launch 一个子协程，然后在它里面又去 launch 两个子协程。这个时候，第一个子协程里去 throw 了一个 Exception，那么第二个子协程会受到影响吗？--- 会的。因为我们就是内部的两个子协程，它的父协程的 Job 类型已经变成了 Job，而不是 SupervisorJob。那么这种情况的话，如果我们想要让他互相之间不会受影响，就要借助一个的supervisorScope  函数来去包裹一下这两个子协程。这样他们的 Job 类型就变成了 SupervisorJob。

## 总结

今天我们首先分享了协程的定义是什么？它是一种协作式的函数调用方式。接着我们又讲了为什么协程可以简化我们的代码，因为它允许我们使用同步的方式来编写一部分代码。后来我们介绍了协程基本用法，包括 launch async 函数，还有 withContext 函数，SupervisorJob 等等，我今天讲的这些都是协程的基本用法，当然也是最常用的用法。然后我们又介绍了协程的取消，其中覆盖了协程结构化并发特性，为什么它可以很好的管理。最后我们又介绍了处理异常的方式，包括传统的 try-catch 异常的处理方式和借助 CoroutineExceptionHandler 的处理方式。还有协程失败时，我们怎样去更改失败时的相应行为。

## Q&A

**为什么挂起函数需在协程作用域里调用？suspend 作用是为了切线程吗?**

我觉得这个问题答案是自然而然的，因为挂起函数它的特性，支持了普通函数不支持的两个特点：挂起和恢复。我可能在这里没法展开来讲，因为它具体的实现方式通过一个叫状态机的方式来实现，但是如果我们从使用层面来理解，你可以就将它理解成代码可以在这一行突然就终止，不再执行了，然后可以在稍后的情况下去恢复继续执行它。而这个特性，在一个普通的函数当中是你绝对不可以想象的。我们在从来没有协程之前，你能相信一个函数可以执行在某一行代码里，然后就突然暂停了吗？这是完全不可能的，而协程的话它是因为内部利用了很多比较巧妙的实现，然后完成了这样一个功能，所以它限定于你要使用它的话，就必须得要在我的协程世界当中才能使用，而在普通的线程世界当中是使用不了的。那么 suspend 关键字就可以理解为，我已经进入到协程的世界了，而刚才我可能讲的快忽略到了一点，刚才的 launch 函数和  async 函数，我们可以将它理解成是线程世界通往协程世界的一个桥梁。那么在他的外部你可以理解成，我现在是在线程世界当中，而调用的 launch 函数进入他的代码块当中，我现在已经进入到协程的世界当中，那么就可以利用协程的一些特性，包括挂起和恢复。

**目前项目主要使用 Java 开发，现在准备使用 Kotlin 以及协程。是否需要把之前的网络请求或者其他的耗时任务都修改成挂起函数，有什么比较推荐的改造方式吗？**

我觉得做这样的优化，你一定要意识到一个问题，做这样的优化对你自己是有好处，你不是在帮助别人，其实是在帮助你自己，因为会让你的代码以后变得更好阅读更好维护也更好扩展。 所以说如果你问我有没有必要要去做这样的事情，我认为非常是有必要的，不是在为任何人做，是在为你自己而做。至于具体的有没有什么更好的方式，我觉得我今天演讲的方式就已经挺好了。参照我今天演讲的方式，然后去对你自己的代码做优化，做重构机制。

**Kotlin 中协程是否可以取代所有 Thread 的操作？**

其实很多人可能会比较在意协程和线程之间的关系，那么协程从它本质上的实现上来讲，它是一个用于管理线程的框架，只是它让我们可以不需要再和线程去直接打交道，而是在协程这个层面去进行异步的操作就可以。而协程它这种实现或者它的本质就决定了它不是为了要替代线程而存在的，他是为了要封装线程而存在的。所以之前也有人会问我协程和线程池之间的关系，他们之间是有一定的联系的，因为协程它本身也是一个线程框架，更准确的说法应该是协程是一个用于管理线程的框架，那么如果再准确一点的话，你可以把它理解成是一种高效且方便的用于线程管理的框架，所以它不是为了替代线程而存在的。

**协程怎样处理异常?**

关于异常处理方式，我刚才在这个主题里已经讲了，其实很大一块内容，但是时间的原因我PPT 点得非常快，这个我要回答的就是一种普遍的方式，协程该怎样处理异常，也就是刚才我里讲述的这种内容。那么对于 Retrofit 而言的话，其实使用刚才这种方式是可以通用解决的，但是 Retrofit 也帮我们提供了一些它内置的异常的解决方案。我不知道我这样不通过代码演示直接口讲，大家能不能理解？ 刚才的我们将 getUser 的返回值声明成了 User 类型对吧？实际上除了我声明为 User 类型之外，Retrofit 还支持让我们声明一种 Response 类型，然后在里面声明一个泛型的类型。当声明成了 Response 这种类型之后，当你的网络请求出现异常之后是不会崩溃的，他会将这个就是异常的情况封装到这个 Response 里面，然后我们可以通过 Response 的一些状态位来判断我当前的请求是否是成功的，if 成功的话我可以进行怎样的处理，else 的话我进行另外的处理，这是 Retrofit 帮我们内置的一种方式。 当然如果你不想使用内置的方式，然后直接将它返回值声明成 User  也是是没问题的，然后使用我们刚才演讲里介绍的这种处理方式一样可以解决问题的。