翻译：[https://labs.ribot.co.uk/android-application-architecture-8b6e34acda65](https://labs.ribot.co.uk/android-application-architecture-8b6e34acda65)

我们的架构之旅从标准的Activities + AsyncTasks到一种由支持RxJava的基于MVP结构的。

Android开发的生态系统发展非常迅速。新的开发工具、新的SDK包以及新的博客文章，这些每周都在发生着。如果你去度假一个月，那么当你回来时，将会有一个新版本的SDK出现，同时可能有一个新版本的Google Play服务。

在过去的三年中，我和我的团队（ribot）一直在做Android App的开发。这期间，构建Android App的架构技术一直在不断的发展着。本文通过我们在架构技术的工作和学习过程中的经验、教训和摸索过程来介绍App架构技术的发展过程。

# 过去

2012年，我们的代码库主要遵循Android的基本结构。我们没有使用任何网络库，AsyncTasks仍然使我们的朋友。下图展示了当时的架构组织。![](/assets/Initial architecture.png)代码可以分为两层：数据层从REST APIs和数据库中读/写数据；UI层的责任是将数据展示到UI上。APIProvider提供了使Activities和Fragments与REST APIs交互的方法，这些方法有：使用URLConnection和AsyncTasks在独立的线程中来执行网络操作并将结果回调给Activities。类似的，CacheProviderti提供了在SharedPreferences或SQLite数据库读/写数据的方法，它同样使用了回调将结果返回给Activitties。

# 问题

The main issue with this approach was that the View layer had too many responsibilities. Imagine a simple common scenario where the application has to load a list of blog posts, cache them in a SQLite database and finally display them on a ListView. The Activity would have to do the following:

这一结构的主要问题在于View层承担了过多的责任。一个常见的情景，某个应用需要加载一个博客帖子列表，然后将这个列表存储到SQLite数据库中，同时将这些数据展示到ListView中，这一系列活动需要执行下面的操作：

1. 访问APIProvider中的_loadPosts\(callback\)；_
2. 等待APIProvider的成功回调并访问CacheProvider中的_savePosts\(callback\)；_
3. 等待CacheProvider的成功回调后将数据展示到ListView；
4. 单独处理APIProvider和CacheProvider中潜在的错误回调；



