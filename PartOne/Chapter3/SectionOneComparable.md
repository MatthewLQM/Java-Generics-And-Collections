# 3.1 Comparable 接口
接口 `Comparable<T>` 包含了一个很简单的方法，可以用来和另外一个对象进行比较。
```Java
interface Comparable<T> {
    public int compareTo(T o);
}
```
这个接口根据此对象比入参小、相同还是大，分别返回负数、0和正数。当一个类实现了 `Comparable` 接口时，根据此接口进行的排序叫做这个类的自然排序。
通常来说，一个类的对象，只能和相同类的对象进行比较。例如，`Integer` 实现了 `Comparable<Integer>`：
```Java
Integer int0 = 0;
Integer int1 = 1;
assert int0.compareTo(int1) < 0
```
因为在数字排序下，0 小于 1，所以这个比较返回了一个负数。相似的情况还有：`String` 实现了 `Comparable<String>`：
```Java
String str0 = "zero";
String str1 = "one";
assert str0.compareTo(str1) > 0;
```
因为在字母排序下，zero 在 one 后面，所以这个比较返回了一个正数。
这个接口的类型参数可以保证在编译阶段发现不同类型之间的比较。
```Java
Integer i = 0;
String s = "one";
assert i.compareTo(s) < 0; // compile-time error
```
你可以比较两个整数或者两个字符串，但是比较一个整数和一个字符串会在编译时发生错误。
同时，比较也不允许在不同的两种数字类型中发生：
```Java
Number m = new Integer(2);
Number n = new Double(3.14);
assert m.compareTo(n) < 0; // compile-time error
```
因为 `Number` 并不会实现 `Comparable` 接口，所以在这里，比较是不允许的。
**和 Equals 保持一致** 通常来说，我们要求两个对象是相等的，当且仅当他们的比较是相等的：
`x.equals(y) if and only if x.compareTo(y) == 0`

在这种情况下，我们会说自然排序和相等是一致的。

在你设计一个类时，建议选择自然排序和相等保持一致。这在你使用集合框架中的 `SortedSet` 和 `SortedMap` 时是尤为重要的，因为这两者都会使用自然排序。如果两个比较结果相同的对象同时加入到排序集合中，那么只有一个会被存储，即使两个对象不相等。排序映射表也是这样。（您也可以使用第 3.4 节中描述的比较器指定与排序集或有序映射一起使用的不同排序，但在这种情况下，指定的排序应该再次与equals一致。）
在 Java 核心的库中，几乎所有的类都保证了自然排序和相等是保持一致的。`java.math.BigDecimal` 是一个例外，这个类会比较两个同样的值的精度是否相同，比如 `4.0` 和 `4.00` 是不同的。第 3.3 节给出了另外一个反例。
比较函数和相等函数不同的一点是他不接受空的参数，如果 `x` 非空，则 `x.equals(null)` 必须返回 `false`，而 `x.compareTo(null)` 必须抛出一个 `NullPointerException`。
我们使用标准的语法进行比较，使用 `x.compareTo(y) < 0` 而不是 `x < y`，使用 `x.compareTo(y) <= 0` 而不是 `x <= y`。即使在一些自然排序和相等不一致的类上使用上述法则也是非常明智的。
**可比规范** `Comparable<T>` 接口规范明确了三个属性。这些属性使用 sign 函数定义的，比如 `sgn(x)` 会在 `x` 为负数、0 和正数的情况下，分别返回 `-1`, `0` 和 `1`。
首先，比较函数应该是反对称的。交换参数的顺序，会得到相关的结果：
```Java
sgn(x.compareTo(y)) == -sgn(y.compareTo(x))
```
这一规则概括了数字的属性：`x < y` 当且仅当 `y > x`。这条规则也要求了 `x.compareTo(y)` 抛出异常当且仅当 `y.compareTo(x)` 抛出异常。
第二，比较函数应该是具有传递性的，如果一个值比第二个值小，第二个值比第三个值小，那么第一个值也应该比第三个值小：
```Java
if x.compareTo(y) < 0 and y.compareTo(z) < 0 then x.compareTo(z) < 0
```
这一规则概括了数字的属性：如果 `x < y` 且 `y < z` 那么 `x < z`。
第三，比较函数应该是具有一致性的。如果两个值比较结果相同，那么他们与第三个数的比较结果也应该相同：
```Java
if x.compareTo(y) == 0 then sgn(x.compareTo(z)) == sgn(y.compareTo(z))
```
这一规则概述了数字的属性：如果 `x == y` 那么 `x < z` 当且仅当 `y < z`。虽然没有明确说明，但是这也要求当 `x.compareTo(y) == 0` 时，`x.compareTo(z)` 抛出异常当且仅当 `y.compareTo(z)` 抛出异常。
强烈建议比较和平等是兼容的：
```Java
x.equals(y) 当且仅当 x.compareTo(y) == 0
```
正如我们刚刚看到的，只有一小部分类，比如 `java.math.BigDecimal` 等违反了这条规则。
然而，比较函数一定是自反的，每个值必须和自己是相等的：
```Java
x.compareTo(x) == 0
```
这是根据第一条规则推导出来的：`sgn(x.compareTo(x)) == -sgn(x.compareTo(x))`。

**注意** 值得指出比较定义的微妙之处。 这是比较两个整数的正确方法：
```Java
class Integer implements Comparable<Integer> {
    ...
    public int compareTo(Integer that) {
        return this.value < that.value ? -1 :
                this.value == that.value ? 0 : 1;
    }
    ...
}
```
这个条件表达式只会返回 -1，0 或 1。你或许认为下面的代码也可以，因为比较方法值要求返回负数：
```Java
class Integer implements Comparable<Integer> {
    ...
    public int compareTo(Integer that) {
        // bad implementation --- don't do it this way!
        return this.value - that.value;
    }
    ...
}
```
但是这段代码会在溢出时给出错误的答案。例如，当比较一个很大的负数和一个很大的正数时，结果可能会比 `Integer.MAX_VALUE` 还要大。