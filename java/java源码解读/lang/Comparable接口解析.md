## Comparable接口源码分析<java.lang.Comparable>

这是个用于实现类对象排序的接口，通过该接口来自定义对象的自然排序。

### 1. 我们还是先来看看源码定义是怎么样的

```java
public interface Comparable<T>{
	public int compareTo(T o);
}
```

这就是comparable接口的所有的源码了，就是这么简单的代码。我们可以看出来，接口接收一个泛型参数<T>来进行类型排序。
该接口也只有一个compareTo方法，实现该接口可以自定义排序方式。

### 2、接口comparable的特性

对于实现该接口的类的对象会进行强制整体排序，这个排序简称为类的自然排序，这个类的compareTo方法简称为自然比较方法。
- 实现了该接口的list或array对象可以通过Collections.sort/Arrays.sort进行自动排序；实现了该接口的对象，如果存储在SortedMap中的key中，
	或者作为元素存放进SortedSet中，则不需要去指定comparator
- 类的自然排序：对于类中的元素e1.compareTo(e2) == 0的返回值是和e1.equals(e2)等同的，null不是任何类的实例，
	尽管e.equals(null)返回false，但是e.compareTo(null)应该抛出NullPointerException
- 虽然不是必须的，但是强烈建议：自然排序应该和equals保持一致，因为有序sets和有序maps没有明确的比较器就会表现的很奇怪，就是使用它们的元素
	或者key值时这自然排序是和equals不一致的。特别是这样的set/map违反了set/map定义在equals方法里的通用协议。
	
	打个比方：在没有使用明确的comparato情况下，一个集合添加了两个元素a和b，使!a.equals(b) && a.compareTo(b) == 0成立。这两个添加操作是
	返回false的（有序set的大小没有变化），因为在有序set的角度来看这a和b是相等的。
- 所有实现了Comparable接口的java核心类的自然排序是和equals一致的。一个特例是java.math.BigDecimal，对于BigDecimal对象而言，
	相同的值集备不同的精度，其自然排序是相等的，比如4.0和4.00
- 对于数学来说，一个给定的类C，这个自然排序的定义关系是：(x, y) 要满足于 x.compareTo(y) <= 0;也就是说x是小于等于y的，
	排序相等的情况是：(x, y) 满足于 x.compareTo(y) == 0;
- 关于compareTo的协议是：对于C类而言，这个商是个相等关系，这自然排序是一个整体排序，我们说的自然排序是和equals保持一致的，我们所说
	的自然排序的相等关系是通过Object#equals(Object)方法来定义的。
- 这个接口是Java集合框架的一个成员

通过上面的接口特性可以看出：实现该接口的类会进行自然排序，而且该排序是跟equals()方法等价的.

### 3、探究下compareTo(T o)这个唯一方法
```java
	/**
	 * @param   o 要被比较的对象.
	 * @return  返回负值, 0, 或者正数；当对象<,=,或者>指定对象的时候（即参数o对象）
	 * @throws NullPointerException 如果参数o为null
     * @throws ClassCastException 参数o的类型和比较的对象不一致.
	 */
	public int compareTo(T o);

```
从定义可以了解以下信息：

	* 参数：1个，
		- T o，T表示泛型，o是需要比较的对象
	* 返回值：int类型
	* 可以发生的异常：
		- NullPointerException：如果传入的参数是null，则抛出NullPointerException异常
		- ClassCastException：如果传入的参数类型不一致，则抛出ClassCastException异常

我们是通过实现compareTo这个接口方法，与指定对象进行比较来达到排序的目的，就 x.compareTo(y)来说：如果
	返回值 小于0 的话则表示 x < y;
	返回值 等于0 的话则表示 x = y;
	返回值 大于0 的话则表示 x > y;
对于所有的x和y，方法实现必须确保： 当 sgn(x.compareTo(y)) = -sgn(y.compareTo(y)) 时,则意味着如果y.compareTo(x)
抛出异常的话则x.compareTo(y)也要抛出一个异常

而且这个方法的实现也要确保传递性：当 (x.compareTo(y) > 0 和 y.compareTo(z) > 0 则 x.compareTo(z) > 0 也要成立。
最后还要确保：对于所有z而言，当 x.compareTo(y) = 0 时则 sgn(x.compareTo(z)) = sgn(y.compareTo(z))。

强烈建议但不严格要求：(x.compareTo(y)==0) 等价于 (x.equals(y))，通常来说，类实现了Comparable接口并且违反该条件的
都要清楚一点：类的自然排序与equals()方法不一致了。

ps：sgn是个符号函数，有如下性质：sgn(>0) = 1, sgn(==0) = 0, sgn(<0) = -1;


