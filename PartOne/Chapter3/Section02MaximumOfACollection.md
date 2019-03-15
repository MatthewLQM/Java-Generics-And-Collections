# 3.2 集合中的最大值
在本节，我们会展示如何使用 `Comparable<T>` 接口来找到一个集合中的最大值。首先，让我们来看一个最简单的版本。在集合框架中的实际版本，会有一些复杂，他带了一个类型标记，之后我们会看一下为什么这样设计。
下面是类 `Collections` 中，在一个非空的集合中找到最大值的方法：
```Java
public static <T extends Comparable<T>> T max(Collection<T> coll) {
    T candidate = coll.iterator().next();
    for (T elt : coll) {
        if (candidate.compareTo(elt) < 0) candidate = elt;
    }
    return candidate;
}
```
我们首先看到了在1.4节中签名中声明新类型变量的泛型方法。比如，`asList` 方法将类型 `E[]` 作为输入，并返回一个 `List<E>` 的类型，并且适配于所有的类型 `E`。在这里有一个声明了类型边界的泛型方法。`max` 方法将类型 `Collection<T>` 作为输入，并输出一个 `T`，并且是适配于所有 `Comparable<T>` 的子类 `T`。
类型签名开头的尖括号中的短语声明了类型 `T`，而且 `T` 是被 `Comparable<T>` 所限制的。作为通配符，类型限制通常以 `extends` 来声明，即使范围类型是一个接口而不是类。但是不像通配符的是，类型参数必须使用 `extends` 作限制，但是决不能使用 `super`。
在这个案例中，范围是可以递归的，在这种情况下， `T` 本身的边界还可能依赖于 `T`。甚至还可能会有互相递归的边界，比如：
```Java
<T extends C<T, U>, U extends D<T, U>>
```
在 9.5 节，我们可以看到一个互相递归的示例。
`max` 这个方法将集合中的第一个值当做最大值的候选，然后将这个值与集合中的其他值进行比较，如果新的值更大，则进行替换。方法中使用 `iterator().netx()` 而不是 `get(0)` 去获得第一个元素，是因为 `get()` 不是定义在集合上的，而是列表上的。这个方法会在集合为空时抛出 `NoSuchElement` 的异常。
当调用此方法时， 类型 `T` 可以被指定为 `Integer` (`Integer` 实现了 `Comparable<Integer>`) 或 `String` (`String` 实现了 `Comparable<String>`)：
```Java
List<Integer> ints = Arrays.asList(0, 1, 2);
assert Collections.max(ints) == 2;
List<String> strs = Arrays.asList("zero", "one", "two");
assert Collections.max(strs).equals("zero");
```
但是我们并不会选择将 `Number` 作为 `T` (因为 `Number` 并没有实现 `Comparable`)：
```Java
List<Number> nums = Arrays.asList(0, 1, 2, 3.14);
assert Collections.max(nums) == 3.14; // compile-time error
```
在这里调用 `max` 是非法的。
这里有一个很有用的技巧。前面的实现，可以通过使用 foreach 循环来提高简洁性和清晰度。如果效率是大家的关注点，那么你可以使用明确的迭代器来覆写此方法，如下所示：
```Java
public static <T extends Comparable<T>> T max(Collection<T> coll) {
    Iterator<T> it = coll.iterator();
    T candidate = it.next();
    while (it.hasNext()) {
        T elt = it.next();
        if (candidate.compareTo(elt) < 0) {
            candidate = elt;
        }
    }
    return candidate;
}
```
这种方式只会使用一个迭代器，而不是两个，并且少了一次比较的操作。
方法的签名应该尽可能的通用，以最大化效能。如果你可以用一个通配符代替类型参数，那么就应该这么做。我们可以作如下的替换：
```Java
<T extends Comparable<T>> T max(Collection<T> coll)
```
替换为：
```Java
<T extends Comparable<? super T>> T max(Collection<? extends T> coll)
```
根据 `Get And Put` 法则，我们可以在 `Collection` 上使用 `extend`，因为我们是从集合中**获取**值，同样可以在 `Comparable` 上使用 `super` 是因为我们将值放到 `compareTo` 方法中。在下一节中，我们会看到一个如果省略了 `super` 以后丢失类型检查的示例。
如果你看到 Java 库中的此方法的签名，你会看到比之前的代码更糟糕的东西：
```Java
<T extends Object & Comparable<? super T>> T max(Collection<? extends T> coll) 
```
这是为了兼容之前的逻辑，我们将会在 3.6 节的最后解释他。
