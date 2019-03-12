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


2019.03.12 分割线--------------