翻译：[https://labs.ribot.co.uk/android-application-architecture-8b6e34acda65](https://labs.ribot.co.uk/android-application-architecture-8b6e34acda65)

我们的架构之旅从标准的Activities + AsyncTasks到一种由支持RxJava的基于MVP结构的。

Android开发的生态系统发展非常迅速。新的开发工具、新的SDK包以及新的博客文章，这些每周都在发生着。如果你去度假一个月，那么当你回来时，将会有一个新版本的SDK出现，同时可能有一个新版本的Google Play服务。

在过去的三年中，我和我的团队（ribot）一直在做Android App的开发。这期间，构建Android App的架构技术一直在不断的发展着。本文通过我们在架构技术的工作和学习过程中的经验、教训和摸索过程来介绍App架构技术的发展过程。

# 过去

2012年，我们的代码库主要遵循Android的基本结构。我们没有使用任何网络库，AsyncTasks仍然使我们的朋友。下图展示了当时的架构组织。![](/assets/Initial architecture.png)代码可以分为两层：数据层从REST APIs和数据库中读/写数据；UI层的责任是将数据展示到UI上。APIProvider提供了使Activities和Fragments与REST APIs交互的方法，这些方法有：使用URLConnection和AsyncTasks在独立的线程中来执行网络操作并将结果回调给Activities。类似的，CacheProviderti提供了在SharedPreferences或SQLite数据库读/写数据的方法，它同样使用了回调将结果返回给Activitties。

