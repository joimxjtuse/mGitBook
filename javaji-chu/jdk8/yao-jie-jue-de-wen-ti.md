一. **Lambda所提供的新语言特性**

1）Lambda表达式；

2）方法引用；

3）扩展的目标类型和类型推断；

4）接口中的默认和静态方法。

二. **Lambda语言的背景**

2.1 **函数式语言 vs. 面向对象语言**。  
无论是面向对象语言还是函数式语言中，都是可以动态的封装程序的行为：面向对象语言提供了带有方法的对象，而函数式采用函数来封装行为这样的联系并不明显。以Java语言为例，Java语言中有一些对象仅仅是对一个函数的封装。Java API中定义了一种接口（回调接口），它希望调用者提供一个该接口的实例。例如：

```
public interface ActionListener{
    void actionPerformed(ActionEvent event);
}
```

这里并不需要专门定义一个类来实现`ActionListener`，因为它只会在调用处被使用一次。用户一般会使用匿名类型把行为内联（inline）：

```
button.addActionListener(new ActionListener() {
  public void actionPerformed(ActionEvent e) {
    ui.dazzle(e.getModifiers());
  }
});
```

很多库都在依赖于上面的模式。对于并行 API 更是如此，因为我们需要把待执行的代码提供给并行 API。并行编程是一个非常值得研究的领域，因为在这里摩尔定律得到了重生：**尽管我们没有更快的 CPU，但是我们有更多的 CPU**。而串行 API 就只能使用有限的计算能力。

随着回调模式和函数式编程风格的日益流行，我们需要在Java中提供一种尽可能轻量级的将**代码封装为数据**（Model code as data）的方法。匿名内部类并不是一个好的选择，因为：

1 语法冗长；

2 匿名类中的字段和this可能会引起混淆；

3 类型载入和实例常见语义不灵活；

4 无法使用非final修饰的局部变量；

5 无法对控制流进行抽象。

这些问题，在Jdk8中大部分都解决了：

* 通过提供更简洁的语法和局部作用域规则，Java SE 8 彻底解决了问题 1 和问题 2；
* 通过提供更加灵活而且便于优化的表达式语义，Java SE 8 绕开了问题 
* 通过允许编译器推断变量的“常量性”（finality），Java SE 8 减轻了问题 4 带来的困扰

不过，Java SE 8 的目标并非解决所有上述问题。因此非final修饰的局部变量（问题 4）和非局部控制流（问题 5）并不在 Java SE 8的范畴之内。

2.2 **函数式接口**

上面提到的ActionListener接口只有一个方法（Runnable, Comparator等回调接口，都有这样的特征），我们将只有一个方法的接口称为函数式接口。

参考资料：

[http://cr.openjdk.java.net/~briangoetz/lambda/lambda-state-final.html](http://cr.openjdk.java.net/~briangoetz/lambda/lambda-state-final.html)

[http://cr.openjdk.java.net/~briangoetz/lambda/lambda-libraries-final.html](http://cr.openjdk.java.net/~briangoetz/lambda/lambda-libraries-final.html)

[http://cr.openjdk.java.net/~briangoetz/lambda/lambda-translation.html](http://cr.openjdk.java.net/~briangoetz/lambda/lambda-translation.html)

