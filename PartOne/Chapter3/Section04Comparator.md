# 3.4 Comparator
有时候，我们想比较一些没有实现 `Comparable` 接口的对象，或者是使用另外一种顺序进行比较。通过 `Comparable` 接口进行比较得到的顺序叫做自然排序，所以说 `Comparator` 接口提供了一种非自然排序。
我们通过 `Comparator` 接口实现了更多的排序，这个接口只有一个简单的方法：
```Java
interface Comparator<T> {
    public int compare(T o1, T o2);
}
```
`compare` 方法在 o1 比 o2 小、相等或大时，分别返回负数、0 和整数，和 `compareTo` 方法相同。
这里有一个比较器，认为两个字符串中更短的那个更小。只有两个字符串长度相同时，他们才会通过自然排序进行比较：
```Java
Comparator<String> sizeOrder = new Comparator<String>() {
    public int compare(String s1, String s2) {
        return s1.length() < s2.length() ? -1 :
        s1.length() > s2.length() ? 1 : s1.compareTo(s2);
    }
}
```
下面是一个示例：
```Java
assert "two".compareTo("three") > 0;
assert sizeOrder.compare("two", "three") < 0;
```
在字母序中，"two" 是大于 "three"，但是在大小序中，它更小。
Java 库经常提供 `Comparable` 和 `Comparator` 的选择。对于每一个使用 `Comparable` 来进行类型限制的泛型方法，总会有一个带有额外参数的 `Comparator` 泛型方法。比如来说：
```Java
public static <T extends Comparable<? super T>> T max(Collections<? extends T> coll)
```
我们也会有”
```Java
public static <T> T max(Collection<? extends T> coll, Comparator<? super T> cmp)
```
同样也有相似的方法去寻找最小值。举个例子，下面标识了如何使用自然排序或大小排序来找到一个列表的最大值和最小值：
```Java
Collection<String> strings = Arrays.asList("from", "aaa", "to", "zzz");
assert max(strings).equals("zzz");
assert min(strings).equals("aaa");
assert min(strings, sizeOrder).equals("from");
assert max(strings, sizeOrder).equals("to");
```
在大小序的规则下，`from` 是最大的，`to` 是最小的。
下面是使用比较器的 `max` 函数版本：
```Java
public static <T> T max(Collection<? extends T> coll, Comparator<? super T> cmp) {
    T candidate = coll.iterator().next();
    for (T elt : coll) {
        if (cmp.compare(candidate, elt) < 0) {
            candidate = elt;
        }
    }
    return candidate;
}
``` 
和之前的版本相比，唯一的变化是之前我们的写法为 `candidate.compareTo(elt)`，现在是 `cmp.compare(candidate, elt)`。
定义一个提供自然排序的比较器是很容易的：
```Java
public static <T extends Comparable<? super T>> Comparator<T> naturalOrder() {
    return new Comparator<T> {
        public int compare(T o1, T o2) {
            return o1.compareTo(o2);
        }
    }
}
```
使用这种方式，可以很简单的使用一个给定的比较器来完成 `max` 函数：
```Java
public static <T extends Comparable<? super T>> T max(Collection<? extends T> coll) {
    return max(coll, Comparators.<T>naturalOrder());
}
```
必须显式地为调用泛型方法naturalOrder提供类型参数，因为推断类型的算法将无法计算出正确的类型。
定义一个采用比较器并以与给定顺序相反的方式返回新比较器的方法也很容易：
```Java
public static <T> Comparator<T> reverseOrder(final Comparator<T> cmp) {
    return new Comparator<T>() {
        public int compare(T o1, T o2) {
            return cmp.compare(o2, o1);
        }
    }
}
```
这只是简单的交换了两个入参的顺序。（根据比较器的定义，这和保证参数的顺序而对结果取反是相同的）下面是一个返回自然排序相反结果的方法：
```Java
public static <T extends Comparable<? super T>> Comparator<T> reverseOrder() {
    return new Comparator<T>() {
        public int compare(T o1, T o2) {
            return o2.compareTo(o1);
        }
    }
}
```
可以在 17.4 节看到，`java.util.Collections` 中有相似的方法。
最后，我们可以使用 `reverseOrder` 的两个版本根据 `max` 的两个版本定义 `min` 的两个版本：
```Java
public static <T> T min(Collection<? extends T> coll, Comparator<? super T> cmp) {
    return max(coll, reverseOrder(cmp));
}
public static <T extends Comparable<? super T>> T min(Collection<? extends T> coll) {
    return max(coll, Comparators.<T>reverseOrder());
}
```
## 示例 3-3. 比较器
```Java
class Comparators {
    public static <T> T max(Collection<? extends T> coll, Comparator<? super T> cmp) {
        T candidate = coll.iterator().next();
        for(T elt : coll) {
            if (cmp.compare(candidate, elt) < 0) {
                candidate = elt;
            }
        }
        return candidate;
    }
    public static <T extends Comparable<? super T>> T max(Collection<? extends T> coll) {
        return max(coll, Comparators.<T>naturalOrder());
    }
    public static <T> T min(Collection<? extends T> coll, Comparator<? super T> cmp) {
        return max(coll, reverseOrder(cmp));
    }
    public static <T extends Comparable<? super T>>
    T min(Collection<? extends T> coll)
  {
    return max(coll, Comparators.<T>reverseOrder());
  }
  public static <T extends Comparable<? super T>>
    Comparator<T> naturalOrder()
  {
    return new Comparator<T>() {
      public int compare(T o1, T o2) { return o1.compareTo(o2); }
    };
  }
  public static <T> Comparator<T>
    reverseOrder(final Comparator<T> cmp)
  {
    return new Comparator<T>() {
      public int compare(T o1, T o2) { return cmp.compare(o2,o1); }
    };
  }
  public static <T extends Comparable<? super T>>
    Comparator<T> reverseOrder()
  {
    return new Comparator<T>() {
      public int compare(T o1, T o2) { return o2.compareTo(o1); }
    };
  }

}
```
集合框架确实提供了min和max的两个版本，其中包含此处给出的签名，请参见第17.1节。但是，如果您检查库的源代码，您将看到四个中没有一个是根据其他任何一个定义的;相反，每个都是直接定义的。更直接的版本更长，更难维护，但速度更快。使用Sun目前的JVM，测量结果显示速度提升约30％。这种加速是否值得代码重复取决于使用代码的情况。由于Java实用程序可能很好地用于关键的内部循环中，因此库的设计者更喜欢执行速度而不是表达式经济。但情况并非总是如此。 30％的改进可能听起来令人印象深刻，但除非程序的总时间很长并且例程出现在频繁使用的内循环中，否则它是微不足道的。不要让你自己的代码不必要地延长，只是为了勉强做出一点改进。

作为比较器的最后一个例子，这里有一个方法，它在元素上采用比较器并在元素列表上返回一个比较器：
```Java
public static <E>
  Comparator<List<E>> listComparator(final Comparator<E> comp) {
  return new Comparator<List<E>>() {
    public int compare(List<E> list1, List<E> list2) {
      int n1 = list1.size();
      int n2 = list2.size();
      for (int i = 0; i < Math.min(n1,n2); i++) {
        int k = comp.compare(list1.get(i), list2.get(i));
        if (k != 0) return k;
      }
      return (n1 < n2) ? -1 : (n1 == n2) ? 0 : 1;
    }
  };
}
```
循环比较两个列表的相应元素，并在找到不相等的相应元素时终止（在这种情况下，具有较小元素的列表被认为较小）或当达到任一列表的末尾时（在这种情况下， 较短的列表被认为较小）。 这是列表的通常排序; 如果我们将一个字符串转换为一个字符列表，它会给出字符串的通常顺序。