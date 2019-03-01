翻译：[https://labs.ribot.co.uk/android-application-architecture-8b6e34acda65](https://labs.ribot.co.uk/android-application-architecture-8b6e34acda65)

我们的架构之旅从标准的Activities + AsyncTasks到一种由支持RxJava的基于MVP结构的。

Android开发的生态系统发展非常迅速。新的开发工具、新的SDK包以及新的博客文章，这些每周都在发生着。如果你去度假一个月，那么当你回来时，将会有一个新版本的SDK出现，同时可能有一个新版本的Google Play服务。

在过去的三年中，我和我的团队（ribot）一直在做Android App的开发。这期间，构建Android App的架构技术一直在不断的发展着。本文通过我们在架构技术的工作和学习过程中的经验、教训和摸索过程来介绍App架构技术的发展过程。

# 过去

2012年，我们的代码库主要遵循Android的基本结构。我们没有使用任何网络库，AsyncTasks仍然使我们的朋友。下图展示了当时的架构组织。![](/assets/Initial architecture.png)代码可以分为两层：数据层从REST APIs和数据库中读/写数据；UI层的责任是将数据展示到UI上。APIProvider提供了使Activities和Fragments与REST APIs交互的方法，这些方法有：使用URLConnection和AsyncTasks在独立的线程中来执行网络操作并将结果回调给Activities。类似的，CacheProviderti提供了在SharedPreferences或SQLite数据库读/写数据的方法，它同样使用了回调将结果返回给Activitties。

# 问题

这一结构的主要问题在于View层承担了过多的责任。一个常见的情景，某个应用需要加载一个博客帖子列表，然后将这个列表存储到SQLite数据库中，同时将这些数据展示到ListView中，这一系列活动需要执行下面的操作：

1. 访问APIProvider中的_loadPosts\(callback\)；_
2. 等待APIProvider的成功回调并访问CacheProvider中的_savePosts\(callback\)；_
3. 等待CacheProvider的成功回调后将数据展示到ListView；
4. 单独处理APIProvider和CacheProvider中潜在的错误回调。

这是一个非常简单的例子。在实际情况中，REST API返回的数据可能不是View层直接需要的。因此，在显示数据之前，Activity可能需要以某种方式转换或过滤数据。另一种常见情况，loadPostst\(\)方法获取需要从其他位置获取的参数，例如Play Services SDK提供的电子邮件地址。 SDK可能会使用回调异步返回电子邮件，这意味着我们现在有三个级别的嵌套回调。如果我们不断增加复杂性，这种设计将导致我们称为“回调地狱”的结果。

最终，

* Activities、Fragments变得异常庞大且难以维护；
* 过多的嵌套回调使得代码非常丑陋且难以理解，同时更改代码或添加信得功能非常困难；
* 单元测试很难实现，甚至不可能实现。因为很多业务逻辑都在Activities/Fragments内实现，这些业务想实现单元测试是困难的。

# 由RxJava驱动的新框架

两年的时间，我们都在维护上面的结构。这段时间里，我们也做过一些改善措施，来缓解上面提到的几个问题。比如，我们添加了一些Helper类，将一些业务从Activities/Fragments里剥离，在APIProvider内，我们使用Volley。即使有着一系列改变，我们的代码也没有解决测试不友好的问题，而且“嵌套地狱”的问题仍然在发生。

直到2014年接触了RxJava后这一情况才有所缓解。通过在几个示例项目上使用RxJava重构，我们意识到这才是解决“回调地狱”问题的终极解决方案（如果你不熟悉**反应式编程**，可以阅读这篇文章：[https://gist.github.com/staltz/868e7e9bc2a7b8c1f754](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754）。)）。简而言之，RxJava允许通过异步流来管理数据，同时提供了许多对流的操作，比如转换\(transfer\)、过滤\(filter\)和比较\(compare\)。

考虑到前几年开发过程中的痛苦，我们开始考虑新的App架构该怎么设计。所以我们提出了下面的设计结构。  
![](/assets/RxJava-driven architecture.png)与第一个设计图类似，这一结构也有分为数据层和视图层。数据层包括DataManager和一系列Helper类。视图层由一系列Android组件组成：Activities、Fragments、ViewGroup，等等。每一个Helper类都有具体的职责，并且实现起来都很简洁。例如，大多数项目都有访问REST API的Helper，从数据库读取数据或与第三方SDK交互。不同的应用程序将有不同数量的Helpers，但最常见的是：

* PreferencesHelper：从SharedPreferences读/写数据；
* DatabaseHelper：访问SQLite数据库；
* Retrofit services：执行对REST APIs的调用。我们开始使用Retrofit来替换Volley，是因为它提供了对RxJava

的支持，同时它的使用也非常友好。

大多数Helper类中提供的开放方法返回的都是RxJava Observable。**DataManager**是架构的核心。它广泛的使用RxJava提供的combine、filter和transfer操作来处理Helper类中返回的数据。**DataManager**的目的是为UI层（Activitties/Fragments）提供不在需要再次转换的数据来减少UI层的工作量。

TheDataManageris是建筑的大脑。它广泛使用RxJava运算符来组合，过滤和转换从辅助类中检索的数据。 DataManager的目的是通过提供准备显示且通常不需要任何转换的数据来减少活动和片段必须完成的工作量。下面的代码展示了**DataManager**方法的示例。这些代做了下面的事情：

1. 访问Retrofit service，通过REST API请求博客帖子列表；
2. 借助DatabaseHelper，将帖子内容保存在本地数据库中达到缓存的目的；
3. 过滤出今天的帖子，因为这才是UI层想要展示的。

```
public Observable<Post> loadTodayPosts() {
        return mRetrofitService.loadPosts()
                .concatMap(new Func1<List<Post>, Observable<Post>>() {
                    @Override
                    public Observable<Post> call(List<Post> apiPosts) {
                        return mDatabaseHelper.savePosts(apiPosts);
                    }
                })
                .filter(new Func1<Post, Boolean>() {
                    @Override
                    public Boolean call(Post post) {
                        return isToday(post.date);
                    }
                });
}
```

View层的组件（Activities/Fragments）通过访问loadTodayPosts（）方法并且订阅得到返回值的Observable事件。订阅完成后，Observable发出的不同帖子可以直接添加到适配器，以便在RecyclerView或类似Ui上显示。

该架构的最后一个元素是**Event Bus**。Event Bus允许我们广播数据层的事件，以便View层中的多个组件可以订阅这些事件。例如，在DataManager中的signOut\(\)方法中可以在Observable完成时发布一event，多个Activities可以订阅这一事件，在注销事件发生后更改UI状态为注销。以便订阅此事件的多个活动可以更改其UI以显示已注销状态。

# 为什么这一结构更好?

* RxJava Observables和运算符不需要嵌套回调。

![](/assets/callback hell.png)

* DataManager接管了以前属于视图层的职责。这使得Activies、Fragments更加轻量级；
* 将Activities、Fragments中的代码移到DataManger/Helper意味着单元测试变得容易；
* 将各类的职责清楚地分离，并将DataManager作为与数据层交互的唯一点，使得这种架构测试更加友好。Helper类或DataManager特更易模拟。

#### 我们还有什么问题吗？? {#27ce}

* 对于大型或复杂的项目，DataManager可能会变得过于臃肿且难以维护；

* 尽管View层的组件变得轻量级，但是依然需要有处理大量管理RxJava订阅，分析错误，等等的逻辑。

#### 集成MVP

在过去的一年中，MVP或MVVM等几种架构模式在Android社区中越来越受欢迎。在对这些模式的示例项目和文章探索之后，我们发现MVP可以为我们现有的方法带来非常有价值的改进。因为我们当前的架构分为两层（View和Data），所以添加MVP感觉很自然。我们只需添加一个新的**Presenter**层，并将部分代码从View层移动到**Presenter**。

![](/assets/MVP-based architecture.png)Data层保持原样，但是现在它称作Model层，这与MVP模式保持一致。

**Presenter**负责向**Model**请求数据并在获取到结果后调用U的方法。**Presenter**订阅了Datamanager数据返回的Obervable。因此，它必须处理调度程序（[schedulers](http://reactivex.io/documentation/scheduler.html)）和订阅等事件（[subscriptions](http://reactivex.io/RxJava/javadoc/rx/Subscription.html)）。此外，如果需要，他们可以分析错误代码或对数据流执行额外的操作。例如，如果我们需要过滤一些数据并且这个过滤器不可能在其他地方重复使用，那么在Presenter而不是DataManager中实现它可能更有意义。

下面的示例，是一个Presenter中的方法，它订阅了上一节提到的dataManager.loadTodayPosts\(\)返回的Observable。

```
public void loadTodayPosts() {
    mMvpView.showProgressIndicator(true);
    mSubscription = mDataManager.loadTodayPosts().toList()
            .observeOn(AndroidSchedulers.mainThread())
            .subscribeOn(Schedulers.io())
            .subscribe(new Subscriber<List<Post>>() {
                @Override
                public void onCompleted() {
                    mMvpView.showProgressIndicator(false);
                }

                @Override
                public void onError(Throwable e) {
                    mMvpView.showProgressIndicator(false);
                    mMvpView.showError();
                }

                @Override
                public void onNext(List<Post> postsList) {
                    mMvpView.showPosts(postsList);
                }
            });
    }
```

mMvpView是Presenter在处理的View组件。通常，MVP的View是指Activity、Fragment或ViewGroup的实例。



与以前的体系结构一样，视图层包含标准框架组件，如ViewGroups，Fragments或Activities。主要区别在于这些组件不直接订阅Observables。它们实现了一个MvpView接口，并提供了一些简明的方法，如showError（）或showProgressIndicator（）。视图组件还负责处理用户交互（例如单击事件）并通过在演示者中调用正确的方法来相应地执行操作。例如，如果我们有一个加载帖子列表的按钮，我们的Activity将从onClick监听器调用presenter.loadTodayPosts（）。

Like the previous architecture, the**view layer**contains standard framework components like ViewGroups, Fragments or Activities. The main difference is that these components don’t subscribe directly to Observables. They instead implement an MvpView interface and provide a list of**concise**methods such as_showError\(\)\_or\_showProgressIndicator\(\)_. The view components are also in charge of handling user interactions such as click events and act accordingly by calling the right method in the presenter. For example, if we have a button that loads the list of posts, our Activity would call\_presenter.loadTodayPosts\(\)\_from the onClick listener.

If you want to see a full working sample of this MVP-based architecture, you can check out our

[Android Boilerplate project on GitHub](https://github.com/ribot/android-boilerplate)

. You can also read more about it in the

[ribot’s architecture guidelines](https://github.com/ribot/android-guidelines/blob/master/architecture_guidelines/android_architecture.md)

#### 为什么这一设计更好? {#90ec}

* Activities and Fragments become very lightweight. Their only responsibilities are to set up/update the UI and handle user events. Therefore, they become easier to maintain.

* We can now easily write unit tests for the presenters by mocking the view layer. Before, this code was part of the view layer so we couldn’t unit test it. The whole architecture becomes very test-friendly.

* If the data manager is becoming bloated, we can mitigate this problem by moving some code to the presenters.

#### 我们还有什么问题? {#a51d}

* Having a single data manager can still be an issue when the codebase becomes very large and complex. We haven’t reached the point where this is a real problem but we are aware that it could happen.

值得一提的是，这不是完美的架构。事实上，认为有一种独特而完美的东西可以永远解决你所有的问题，是天真的。 Android生态系统将继续快速发展，我们必须通过探索、阅读和试验来拥抱这些变化，以便能够找到更好的方法来持续构建优秀的Android应用程序。

