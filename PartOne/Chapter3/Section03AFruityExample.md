# 3.3 一个水果的例子
`Comparable<T>` 接口让什么可以被比较，什么不能被比较有了一个很好的控制。假设我们有一个类 `Fruit` 和他的两个子类 `Apple` 和 `Orange`。根据我们如何设置他们，我们可能可以，或者不可以比较橘子和苹果。
示例 3.2 不允许苹果和橘子之间的比较，他是这样进行声明的：
```Java
class Fruit {...}
class Apple extends Fruit implements Comparable<Apple> {...}
class Orange extends Fruit implements Comparable<Orange> {...}
```
每一个水果都有名字和大小，如果两个水果有相同的名字和大小，那么他们就是相等的。依据好的实践，我们也要定义一个哈希函数，来保证两个相同的对象具有相同的哈希值。苹果通过比较他们的大小来进行比较，橘子也是。既然 `Apple` 实现了 `Comparable<Apple>`，那么很明显，你只能用 `Apple` 比较 `Apple`，而不能和 `Oranges` 比较。测试方法建立了三个列表：一个只有苹果，一个只有橘子，一个是两种水果的混合。我们可以找到前两个列表的最大值，但是当尝试找到混合列表的最大值时，会抛出一个编译异常。
示例 3.1 允许了苹果和橘子相比，两者的差别如下：
## 示例 3-1. 允许苹果和橘子的对比
```Java
abstract class Fruit implements Comparable<Fruit> {
    protected String name;
    protected int size;
    protected Fruit(String name, int size) {
        this.name = name;
        this.size = size;
    }
    public boolean equals(Object o) {
        if (o instanceof Fruit) {
            Fruit that = (Fruit) o;
            return this.name.equals(this.name) && this.size == that.size;
        } else {
            return false;
        }
    }
    public int hash() {
        return name.hash() * 29 + size;
    }
    public int compareTo(Fruit that) {
        return this.size < that.size ? -1 :
        this.size == that.size ? 0 : 1;
    }
}
class Apple extends Fruit {
    public Apple(int size) {
        super("Apple", size);
    }
}
class Orange extends Fruit {
    public Orange(int size) {
        super("Orange", size);
    }
}
class Test {
    public static void main(String[] args) {
        Apple a1 = new Apple(1);
        Apple a2 = new Apple(2);
        Orange o3 = new Orange(3);
        Orange o4 = new Orange(4);

        List<Apple> apples = Arrays.asList(a1, a2);
        assert Collections.max(apples).equals(a2);

        List<Orange> oranges = Arrays.asList(o3, o4);
        assert Collections.max(oranges).equals(o4);

        List<Fruit> mixed = Arrays.<Fruit>asList(a1, o3);
        assert Collections.max(mixed).equals(o3); // ok
    } 
}
```
## 示例 3-2. 不允许苹果和橘子的对比
```Java
abstract class Fruit {
    protected String name;
    protected int size;
    protected Fruit(String name, int size) {
        this.name = name;
        this.size = size;
    }
    public boolean equals(Object o) {
        if (o instanceof Fruit) {
            Fruit that = (Fruit) o;
            return this.name.equals(this.name) && this.size == that.size;
        } else {
            return false;
        }
    }
    public int hash() {
        return name.hash() * 29 + size;
    }
    protected int compareTo(Fruit that) {
        return this.size < that.size ? -1 :
        this.size == that.size ? 0 : 1;
    }
}
class Apple extends Fruit implements Comparable<Apple> {
    public Apple(int size) {
        super("Apple", size);
    }
    public int compareTo(Apple a) {
        return super.compareTo(a);
    }
}
class Orange extends Fruit implements Comparable<Orange> {
    public Orange(int size) {
        super("Orange", size);
    }
    public int compareTo(Orange a) {
        return super.compareTo(a);
    }
}
class Test {
    public static void main(String[] args) {
        Apple a1 = new Apple(1);
        Apple a2 = new Apple(2);
        Orange o3 = new Orange(3);
        Orange o4 = new Orange(4);

        List<Apple> apples = Arrays.asList(a1, a2);
        assert Collections.max(apples).equals(a2);

        List<Orange> oranges = Arrays.asList(o3, o4);
        assert Collections.max(oranges).equals(o4);

        List<Fruit> mixed = Arrays.<Fruit>asList(a1, o3);
        assert Collections.max(mixed).equals(o3); // compile-time error
    } 
}
```
