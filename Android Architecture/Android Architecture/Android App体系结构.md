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
4. 单独处理APIProvider和CacheProvider中潜在的错误回调；

这是一个非常简单的例子。在实际情况中，REST API返回的数据可能不是View层直接需要的。因此，在显示数据之前，Activity可能需要以某种方式转换或过滤数据。另一种常见情况，loadPostst\(\)方法获取需要从其他位置获取的参数，例如Play Services SDK提供的电子邮件地址。 SDK可能会使用回调异步返回电子邮件，这意味着我们现在有三个级别的嵌套回调。如果我们不断增加复杂性，这种设计将导致我们称为“回调地狱”的结果。

最终，

* Activities、Fragments变得异常庞大且难以维护；
* 过多的嵌套回调使得代码非常丑陋且难以理解，同时更改代码或添加信得功能非常困难；
* 单元测试很难实现，甚至不可能实现。因为很多业务逻辑都在Activities/Fragments内实现，这些业务想实现单元测试是困难的。

# 由RxJava驱动的新框架

两年的时间，我们都在维护上面的结构。这段时间里，我们也做过一些改善措施，来缓解上面提到的几个问题。比如，

We followed the previous approach for about two years. During that time, we made several improvements that slightly mitigated the problems described above. For example, we added several helper classes to reduce the code in Activities and Fragments and we started using[Volley](http://developer.android.com/training/volley/index.html)in the APIProvider. Despite these changes, our application code wasn’t yet test-friendly and the\_callback hell\_issue was still happening too often.

It wasn’t until 2014 when we started reading about[RxJava](http://reactivex.io/). After trying it on a few sample projects, we realised that this could finally be the solution to the nested callback problem. If you are not familiar with reactive programming you can read[this introduction](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754). In short,RxJava allows you to manage data via asynchronous streams and gives you many[operators](http://reactivex.io/documentation/operators.html)that you can apply to the stream in order to transform, filter or combine the data.

Taking into account the pains we experienced in previous years, we started to think about how the architecture of a new app would look. So we came up with this.  
![](/assets/RxJava-driven architecture.png)Similar to the first approach, this architecture can be separated into a data and view layer. The**data layer**contains the DataManager and a set of helpers. The**view layer**is formed by Android framework components like Fragments, Activities, ViewGroups, etc.

**Helper classes**\(third column on diagram\) have very specific responsibilities and implement them in a concise manner. For example, most projects have helpers for accessing REST APIs, reading data from databases or interacting with third party SDKs. Different applications will have a different number of helpers but the most common ones are:

* PreferencesHelper: reads and saves data in SharedPreferences.

* DatabaseHelper: handles accessing SQLite databases.

* [Retrofit](https://github.com/square/retrofit)  
  services: perform calls to REST APIs. We started using Retrofit instead of Volley because it provides support for RxJava. It’s also nicer to use.

Most of the public methods inside helper classes will return RxJava Observables.

The**DataManager**is the brain of the architecture. It extensively uses RxJava operators to combine, filter and transform data retrieved from helper classes. The aim of the DataManager is to reduce the amount of work that Activities and Fragments have to do by providing data that is ready to display and won’t usually need any transformation.

The code below shows what a DataManager method would look like. This sample method works as follows：

1. Call the Retrofit service to load a list of blog posts from a REST API
2. Save the posts in a local database for caching purposes using the DatabaseHelper.
3. Filter the blog posts written today because those are the only ones the view layer wants to display.

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

Components in the**view layer**such as Activities or Fragments would simply call this method and subscribe to the returned Observable. Once the subscription finishes, the different Posts emitted by the Observable can be directly added to an Adapter in order to be displayed on a RecyclerView or similar.

The last element of this architecture is the**event bus**. The event bus allows us to broadcast events that happen in the data layer, so that multiple components in the view layer can subscribe to these events. For example, a\_signOut\(\)\_method in the DataManager can post an event when the Observable completes so that multiple Activities that are subscribed to this event can change their UI to show a signed out state.

# Why was this approach better?

* RxJava Observables and operators remove the need for having nested callbacks.

![](/assets/callback hell.png)

* The DataManager takes over responsibilities that were previously part of the view layer. Hence, it makes Activities and Fragments more lightweight.
* Moving code from Activities and Fragments to the DataManager and helpers means that writing unit tests becomes easier.
* Clear separation of responsibilities and having the DataManager as the only point of interaction with the data layer, makes this architecture**test-friendly**Helper classes or the DataManager can be easily mocked.

#### What problems did we still have? {#27ce}

* For large and very complex projects the DataManager can become too bloated and difficult to maintain.

* Although view layer components such as Activities and Fragments became more lightweight, they still have to handle a considerable amount of logic around managing RxJava subscriptions, analysing errors, etc.

# Integrating Model View Presenter

In the past year, several architectural patterns such as MVP or MVVM have been gaining popularity within the Android community. After exploring these patterns on a sample project and article, we found that MVP could bring very valuable improvements to our existing aproach. Because our current architecture was divided in two layers \(view and data\), adding MVP felt natural. We simply had to add a new layer of presenters and move part of the code from the view to presenters.

![](/assets/MVP-based architecture.png)The data layer remains as it was but it’s now called**model**to be more consistent with the name of the pattern.

**Presenters**are in charge of loading data from the model and calling the right method in the view when the result is ready. They subscribe to Observables returned by the data manager. Therefore, they have to handle things like[schedulers](http://reactivex.io/documentation/scheduler.html)and[subscriptions](http://reactivex.io/RxJava/javadoc/rx/Subscription.html). Moreover, they can analyse error codes or apply extra operations to the data stream if needed. For example, if we need to filter some data and this same filter is not likely to be reused anywhere else, it may make more sense to implement it in the presenter rather than in the data manager.

Below you can see what a public method in the presenter would look like. This code subscribes to the Observable returned by the\_dataManager.loadTodayPosts\(\)\_method we defined in the previous section.

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

The mMvpView is the view component that this presenter is assisting. Usually the MVP view is an instance of an Activity, Fragment or ViewGroup.

Like the previous architecture, the**view layer**contains standard framework components like ViewGroups, Fragments or Activities. The main difference is that these components don’t subscribe directly to Observables. They instead implement an MvpView interface and provide a list of**concise**methods such as_showError\(\)\_or\_showProgressIndicator\(\)_. The view components are also in charge of handling user interactions such as click events and act accordingly by calling the right method in the presenter. For example, if we have a button that loads the list of posts, our Activity would call\_presenter.loadTodayPosts\(\)\_from the onClick listener.

If you want to see a full working sample of this MVP-based architecture, you can check out our

[Android Boilerplate project on GitHub](https://github.com/ribot/android-boilerplate)

. You can also read more about it in the

[ribot’s architecture guidelines](https://github.com/ribot/android-guidelines/blob/master/architecture_guidelines/android_architecture.md)

#### Why is this approach better? {#90ec}

* Activities and Fragments become very lightweight. Their only responsibilities are to set up/update the UI and handle user events. Therefore, they become easier to maintain.

* We can now easily write unit tests for the presenters by mocking the view layer. Before, this code was part of the view layer so we couldn’t unit test it. The whole architecture becomes very test-friendly.

* If the data manager is becoming bloated, we can mitigate this problem by moving some code to the presenters.

#### What problems do we still have? {#a51d}

* Having a single data manager can still be an issue when the codebase becomes very large and complex. We haven’t reached the point where this is a real problem but we are aware that it could happen.

It’s important to mention that this is not the perfect architecture. In fact, it’d be naive to think there is a unique and perfect one that will solve all your problems forever. The Android ecosystem will keep evolving at a fast pace and we have to keep up by exploring, reading and experimenting so that we can find better ways to continue building excellent Android apps.

I hope you enjoyed this article and you found it useful. If so, don’t forget to click the**recommend**button. Also, I’d love to hear your thoughts about our latest approach.

[      
](https://twitter.com/ivacf)

