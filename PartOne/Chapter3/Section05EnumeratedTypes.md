# 3.5 枚举类型
Java 5 支持了枚举类型，下面是一个简单的例子：
```Java
enum Season { WINTER, SPRING, SUMMER, FALL }
```
每一个枚举类的声明可以被扩展成一个程式化的对应的类。这种对应的类被设计成为对于每一个枚举常量，只会有唯一的一个实例，并且和一个对应的 final 变量绑定。在上面的例子当中，`enum` 被声明成 `Season`。有且只有四个这个类的实例，分别对应四个 final 变量：`WINTER, SPRING, SUMMER, FALL`。
每一个枚举类都是 'java.lang.Enum' 的子类。这个类在 Java 文档中的定义如下所示：
```Java
class Enum<E extends Enum<E>>
```
在第一次看到这种代码时，可能会感觉到害怕，但是不要慌。事实上，我们已经看到过一些相似的东西。`E extends Enum<E>` 很像我们在定义 `max` 时使用的短语 `T extends Comparable<T>`，之后，我们会看到，他们的出现是因为相同的原因。
如果想理解发生了什么，我们需要去看一下代码。下面的例 3.4 展示了 `Enum` 这个基础类，例 3.5 展示了 `Season` 这个类对应的枚举类型声明（下面的 `Enum` 类代码是在 Java 库中的，但是我们精简了一下）。
## 例 3.4 枚举类的基类
```Java
public abstract class Enum<E extends Enum<E>> implements Comparable<E> {
    private final String name;
    private final int ordinal;
    protected Enum(String name, int ordinal) {
        this.name = name;
        this.ordinal = ordinal;
    }
    public final String name() {
        return name;
    }
    public final int ordinal() {
        return ordinal;
    }
    public String toString() {
        return name;
    }
    public final int compareTo(E o) {
        return ordinal - o.ordinal;
    }
}
```
## 例 3.5 一个枚举类型的实现
```Java
// corresponds to
// enum Season { WINTER, SPRING, SUMMER, FALL }
final class Season extends Enum<Season> {
    private Season(String name, int ordinal) {
        super(name, ordinal);
    }
    public static final Season WINTER = new Season("WINTER", 0);
    public static final Season SPRING = new Season("SPRING", 1);
    public static final Season SUMMER = new Season("SUMMER", 2);
    public static final Season FALL = new Season("FALL", 3);
    private static final Season[] VALUES = { WINTER, SPRING, SUMMER, FALL };
    public static Season[] values() {
        return VALUES.clone();
    }
    public static Season valueOf(String name) {
        for (Season e : VALUES) {
            if (e.name().equals(name)) {
                return e;
            }
        }
        throw new IllegalArgumentException();
    }
}

下面是枚举类的声明：
```Java
public abstract class Enum<E extends Enums<E>> implements Comparable<E>
```
这个是 `Season` 类的声明：
```Java
class Season extends Enum<Season>
```
在匹配之后，我们就可以看到他是如何工作的了。`E` 这个类型变量表示了实现了枚举类的 `Enum` 的子类，比如 `Season`。每一个 E 必须要满足：
```Java
E extends Enum<E>
```
所以我们可以用 `Season` 代替 `E`，因为：
```Java
Season extends Enum<Season>
```
另外，`Enum` 的定义显示了如下的信息：
```Java
Enum<E> implements Comparable<E>
```
所以：
```Java
Enum<Season> implements Comparable<Season>
```
