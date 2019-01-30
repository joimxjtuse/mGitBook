**Lambda 表达式基础语法**

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

* **基本Lambda表达式举例**

让我们通过一些基本的例子来了解下Lambda表达式是怎么影响我们的日常编码的。

1. 遍历操作

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

// 使用函数式来来表达
players.forEach(System.out:: println );
```

从上面的代码可以看到，Lambda表达式可以将我们的遍历代码缩减为一行。

1. 匿名类

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

* 使用Lambda对集合进行排序

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

* 将Lambda与Stream结合使用

Stream 是对集合的封装，通常与Lambda一起使用。它们支持许多种操作，例如map, filter, limit, sorted, count, min, max, sum, collect等等。同时，Stream使用了懒计算，它并不会访问集合中的所有数据，当遇到getFirst\(\)这样的方法后会结束链式语法。

