## Serializable接口源码分析<java.io.Serializable>

该接口是个序列化的标识接口，只有实现了该接口的类才能进行状态序列化和反序列化

### 1. 我们先来看看接口的定义

```java
public interface Serializable{}
```
这是源码上的全部代码，没有定义任何的方法和变量，也就是说明了该接口仅仅是用来做标识的。
### 2. Serializable接口具备的特性

- 一个类实现了序列化，则它的子类即便没有实现Serializable接口也是可以序列化的。
- 该子类可能需要负责父类（public,proteced修饰的）字段的存储和恢复，但是该父类必须有一个无参构造函数，
	如果没有该构造函数的话就会运行时报错。
- 反序列化时，对于子类的序列化，一个无参构造是必须的，可序列化子类的字段将会从流中恢复。
- 当遍历图形时，可能会遇到不支持Serializable接口的对象，在这种情况下回抛出异常NotSerializableException,
	并且标识该类是不可序列化对象
- 有些类在序列化和反序列化操作中如果需要特殊的处理，则必须实现特定方法的签名：
	* private void writeObject(java.io.ObjectOutputStream out) throws IOException
	* private void readObject(java.io.ObjectInputStream in) throws IOException, ClassNotFoundException
	* private void readObjectNoData() throws ObjectStreamException
- writeObject负责写对象的状态，以便相应的readObject方法能够恢复它。可以通过out.defaultWriteObject调用默认的机制来
	保存对象的字段，该方法不需要关注它的状态是属于父类或者子类的，它的状态是由ObjectOutputStream的writeObject方法或者
	DataOutput的原始类型来写的
- readObject是负责将类的字段从流中恢复过来，也可以调用in.defaultReadObject方法使用默认的机制来恢复对象的非静态和非瞬态字段。
	defaultReadObject方法会根据流中的信息将存储在流中的字段分配给相应对象的字段。
- 如果序列化流没有指定这反序列化对象作为父类，则readObjectNoData方法将负责对象的状态初始化，这可能会发生一些情况：
	* 接收方使用和发送方不一样的反序列化对象实例的版本；
	* 接收方的版本扩展了类，而该类不被发送方扩展；
	* 序列化流有可能被篡改；
	
	readObjectNoData就是被用来正确的初始化反序列化对象，即便对象是敌对或不完整的
- 在将对象写入到流中时，需要指定可替代对象的可序列化类应实现具有确切签名的特殊方法
	* ANY-ACCESS-MODIFIER Object writeReplace() throws ObjectStreamException
- writeReplace方法可被序列化调用,如果该方法存在并且可被序列化对象的类中的方法访问,该方法可以具有私有，受保护和包私有访问。 
	子类访问此方法遵循java可访问性规则。
- 当从流中读取实例时需要指定替换的类，应实现具有确切签名的特殊方法。
	* ANY-ACCESS-MODIFIER Object readResolve() throws ObjectStreamException
- readResolve方法的规则跟writeReplace一样。
- 可序列化运行时会给每个可序列化类分配一个叫serialVersionUID的序列号，反序列化期间该序列号可用于验证发送者和接收者的序列化对象
	已加载对象的类是可兼容序列化的，如果接收者加载了一个对象类和发送者的具备不同的serialVersionUID，则发序列化会造成
	InvalidClassException。
- 一个可序列化类可以声明自己的serialVersionUID字段，该字段必须是static、final 和 long.
	* ANY-ACCESS-MODIFIER static final long serialVersionUID = 72L;
- 如果一个可序列化类不明确声明一个serialVersionUID，则可序列运行时将会基于类的可序列化规范的各个层面计算一个默认的serialVersionUID。
- 但是，我们强烈建议所有的可序列化类都要声明一个serialVersionUID字段，因为默认的serialVersionUID计算是高度敏感的对于类的详情，
	这就可能会因为编译的实现而不一样，这就会造成在反序列化期间的InvalidClassException。因此，为了保证一致性serialVersionUID，
	必须得明确声明serialVersionUID字段，而且字段也是强烈建议使用private修饰，这样对于继承成员是无效的。
- 数组类不能声明一个serialVersionUID值，所以它们有默认的计算值，但是这个必要性对于数组来说已经放弃了。


ps：这些特性都是源码上翻译过来的，所以看过这篇内容的不要说自己没看过源码，这里和源码一模一样，只是这里内容是中文而已。

