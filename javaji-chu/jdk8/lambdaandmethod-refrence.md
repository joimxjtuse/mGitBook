一  **Lambda 表达式基础语法**

lambda 表达式的语法由参数列表、箭头符号`->`

和函数体组成。函数体既可以是一个表达式，也可以是一个语句块：

```
(parameters) -> expression  or  (parameters) -> {statements;}
```

以下，是Java中Lambda表达式的简单例子。

```
// 1. 不需要参数，返回值.
() -> 5

// 2. 接收一个输入参数，返回其2倍的值.
x -> 2 * x

// 3. 接收2个参数（数字），返回它们的差值.
(x, y) -> x - y

// 4. 接收2个int型整数，返回它们的差和.
(int x, int y) -> x + y

// 5. 接收一个String对象，并在控制台打印 ，无返回值.
(String s) -> System.out.print(s)
```

二 **方法引用及语法**

有时候，Lambda表达式可能仅仅调用一个已存在的方法，而不做任何其它事。对于这种情况，通过一个方法名字来引用这个已存在的方法会更加清晰，Java 8的方法引用允许我们这样做。方法引用是一个更加紧凑，易读的Lambda表达式，方法引用是一个Lambda表达式，其中方法引用的操作符是双冒号"::"。

```
// 1. 静态方法引用
   Person :: compareByName
// 2. 特定实例的方法引用
   person :: getAge 
// 3. 任意对象（属于同一个类）的实例方法引用
   Person :: getAge
// 4. 构造方法引用
   Person :: new
```

三 **基本Lambda表达式举例**

让我们通过一些基本的例子来了解下Lambda表达式是怎么影响我们的日常编码的。

**1 遍历操作**

假设有一个玩家列表，为了遍历这个列表，在JDK8之前，我们可能使用for-each循环来实现.

```
String[] atp = {"Rafael Nadal", "Novak Djokovic",  
       "Stanislas Wawrinka",  
       "David Ferrer","Roger Federer",  
       "Andy Murray","Tomas Berdych",  
       "Juan Martin Del Potro"};
List<String> players =  Arrays.asList(atp);

// jdk8之前
for(String player: players){
    System.out.println(player + "; "); 
}

//使用Lambda表达式来操作.
players.forEach(player -> System.out.println(player + "; "))

// 使用方法引用来表达
players.forEach(System.out:: println );
```

从上面的代码可以看到，Lambda表达式可以将我们的遍历代码缩减为一行。

**2 匿名类·**

当使用Runnable来执行一个线程时。

```
//jdk8 之前
new Thread(new Runnable(){

        @Override
        public void run(){ 
                System.out.println("Hello world !");
        }
}).start();

Runnable runnable1 = new Runnable(){
        @Override
        public void run(){
                System.out.println("Hello world !");
        }
};

//使用Labmda表达式
new Thread(() -> System.out.println("Hello world !")).start();

Runnable runnable2 = () -> System.out.println("Hello world !");
```

**3 使用Lambda对集合进行排序**

Java中，Comparator用来对集合进行排序。下面的例子中是一串玩家的名字列表，我们基于玩家的名，姓，名字长度以及最后一个字母来排序。

```
String[] players = {"Rafael Nadal", "Novak Djokovic",   
    "Stanislas Wawrinka", "David Ferrer",  
    "Roger Federer", "Andy Murray",  
    "Tomas Berdych", "Juan Martin Del Potro",  
    "Richard Gasquet", "John Isner"};
```

```
根据玩家的名字来排序

//使用匿名内部类：
Arrays.sort(players,
    new Comparator<String>(){
        @Override
        public int compare(String s1, String s2) {  
            return (s1.compareTo(s2));  
        }  
    }
);

// Lambda：
Comparator<String> c = (String s1, String s2) -> s1.compareTo(s2);
Arrays.sort(players, c);
// 或
Arrays.sort(players,(String s1, String s2) -> s1.compareTo(s2));
```

**4 将Lambda与Stream结合使用**

Stream 是对集合的封装，通常与Lambda一起使用。它们支持许多种操作，例如**map, filter, limit, sorted, count, min, max, sum, collect**等等。同时，Stream使用了懒计算，它并不会访问集合中的所有数据，当遇到getFirst\(\)这样的方法后会结束链式语法。Stream提供了并行流，可以大大提高计算速度。

参考: [https://github.com/joimxjtuse/FuncTest/blob/master/src/cn/joim/jdk8/lambda/Base2.java](https://github.com/joimxjtuse/FuncTest/blob/master/src/cn/joim/jdk8/lambda/Base2.java)

代码详情：[https://github.com/joimxjtuse/FuncTest/tree/master/src/cn/joim/jdk8/lambda](https://github.com/joimxjtuse/FuncTest/tree/master/src/cn/joim/jdk8/lambda)

**三 FunctionalInterface注解与Lambda**

关于@FunctionalInterface注解：

1. FunctionalInterface告诉读者这个类接口支持Lambda；

2. 除非这个类有一个抽象方法，否则这个接口不会编译；

3. 阻止维护人员在该接口内增加抽象方法。

java.util.Function包中，提供了**43**个接口。这其中，包含6个基本的接口。

操作者（**Operator**）接口代表一系列结果和参数类型相同的函数；

谓词\(**Predicate**\)接口代表一个接受一个参数并返回布尔值的函数；

方法（**Function**）接口代表一个输入和输出类型不一致的函数；

供应商（**Supplier**）接口代表一个函数没有参数，返回一个值；同时，消费者（**Consumer**）接口代表一个函数有一个参数没有返回值。

| 接口 | 方法名 | 举例 |
| :---: | :---: | :---: |
| UnaryOperator &lt;T&gt; | T apply\(T t\) | String :: toLowerCase |
| BinaryOperator &lt;T&gt; | T apply\(T t1, T t2\) | BigInteger :: add |
| Predicate &lt;T&gt; | boolean test\(T t\) | Collection :: isEmpty |
| Function&lt;T, R&gt; | R apply\(T t\) | Arrays :: asList |
| Supplier&lt;T&gt; | T get\(\) | Instant :: now |
| Consumer&lt;T&gt; | void accept\(T t\) | System.out::println |

自定义函数式接口（关于函数式的概念后面会提到）：

```
@FunctionalInterface
public interface GreetingService {

    void greeting(String message);
}

public static void main(String[] args) {

        GreetingService serviceWithlambda = message -> {
            System.out.println(message);
        };

        GreetingService serviceWithMethodRefrence = System.out::println;
}
```

**四 Lambda表达式有关的高效编码建议**

1. 采用Lambda表达式来替换匿名内部类方式；
2. 尽量使用系统提供的标准函数式接口；
3. 方法引用通常会比lambda更简洁。当方法引用代码更加少并整洁时，使用方法引用，否则，请使用lambda。

通常情况下，方法引用会比Lambda更简洁，但是也有特殊。当调用者与调用的方法在同一个类内，使用Lambda会比方法引用更简洁。

```
service.execute(GreetingServiceImpl::action);

service.execute(() ->action());
```

总结一下，我们首先介绍了Lambda表达式的基本语法以及一些基本例子（迭代，匿名类）；随后结合Stream，介绍了带有Stream的Lambda，包括forEach, filter, limit, min, max, sort, map等；继而提到了java库中提供的支持Lambda表达式的一些基础类；最后，介绍了Lambda表达式在实践中的几条建议。第四节中提到了函数式，虽然我们通篇在讲述的都在函数式里，但是对这些概念还没有讲清楚，下一节会从函数式等概念来讨论Lambda表达式是怎么引入到jdk中的。

* Java 8 中的 lambda 为什么要设计成这样？
* Java 8 是如何对 lambda 进行类型推导的？它的类型推导做到了什么程度？
* Java 编译器如何处理 lambda？

参考文章：

[https://www.developer.com/java/start-using-java-lambda-expressions.html](https://www.developer.com/java/start-using-java-lambda-expressions.html)

代码地址：

[https://github.com/joimxjtuse/FuncTest.git](https://github.com/joimxjtuse/FuncTest.git)

