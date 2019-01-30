* Lambda 表达式基础语法

```
(parameters) -> expression  or  (parameters) -> {statements;}
```

  以下，是Java中lambda表达式的简单例子。

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



