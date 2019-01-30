* **Lambda 表达式基础语法**

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

// 5. 接收一个String对象，并在控制台打印，无返回值.
(String s) -> System.out.print(s)
```

* **基本Lambda表达式举例**

让我们通过一些基本的例子来了解下Lambda表达式是怎么影响我们的日常编码的。假设有一个玩家列表，为了遍历这个列表，在JDK8之前，我们可能使用for-each循环来实现.

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

//使用Lambda表达式及函数式来表示.

```



