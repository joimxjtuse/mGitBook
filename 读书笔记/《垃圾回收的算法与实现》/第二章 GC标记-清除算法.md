```
mark_sweep(){
    mark_phase();
    sweep_phase();
}
```

标记-清除算法优化：

* 多个空闲链表

利用分块大小不同的空闲链表，即创建只连接大分块的空闲空间和只连接小分块的空闲链表。只按照mutator所申请的分块大小选择空闲链表，就能在短时间内找到符合条件的分块。

通常，会给分块大小设定一个上限，分块如果大于等于这个大小，就全部采用一个空闲链表处理。【分配非常大的分块的情况是极为罕见的】

* BIBOP（Big Bag Of Pages）法

将大小相近的对象整理成固定大小的块进行管理。

* 位图标记法
* 延迟清除法（**Android的内存分配：四次分配策略**）

在分配时调用 lazy\_sweep\(\)函数，进行清除操作。

* 延迟清除法不是一下遍历整个堆，它只在分配时执行必要的遍历，所以可以压缩因清除操作而导致的mutator的暂停时间。



