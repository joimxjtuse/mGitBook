# GC的历史

* 最初的GC是1960年McCarthy在论文中提到的**标记-清除算法**；
* 1960年，George E.Collins在论文中发布了叫作**引用计数**的GC算法；1963年，Harold McBeth提出了引用计数算法的缺点：不能回收“循环引用的变量”；
* 1963，有“人工智能之父”之称的Marvin L. Minsky在论文中发表了**复制算法**；

50年来，GC的根本没有改变，虽然发布了许多新的GC算法，但这些算法只不过是前面的**三种算法衍生出来的产物**。

# 根

可以直接或间接从全局变量中引用的对象视为活动对象。
