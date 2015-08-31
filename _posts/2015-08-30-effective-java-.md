---
layout: post
title: "effective java 总结"
description: ""
category: 
tags: []
---
{% include JB/setup %}

#### 创建和销毁对象

###### 考虑用静态工厂方法代替构造函数

静态工厂方法的好处:
1. 静态工厂方法具有名字,能够更清晰表达方法的含义
 如BigInteger.probablePrime()比构造函数表达得更清晰
2. 不一定要求要创建一个对象,可以为重复的调用返回同一对象
 String.intern()返回常量池的对象,而Boolean.valueOf()返回单例对象
3. 可以返回一个原返回类型的子类的对象
 应用之一是,一个API可以返回一个对象,同时又不使该对象的类成为公有的.
 这样把具体实现隐藏起来,可以的到非常简介的API,特别适合于基于接口的框架结构.同时使得API的实现变得可维护性更强.
 
这种类可以产生对象,但是让对象的产生方式变得更加灵活.如工厂方法可以根据配置文件的配置来决定实例化何种对象,如下代码:
{% highlight java %}
public abstract class Foo {
	// Maps String key to corresponding Class object
	private static map implementations = null;
    private static synchronized void initMapIfNecessary() {
    	if(implementations == null) {
        	implementations = new HashMap();
            
            // load implementations class names and keys from
            // properties file, translate names into Class
            // objects using Class.forname and store mappings.
            ...
        }
    }
}

public static Foo getInstance(String key) {
	initMapIfnecessary();
    Class c = (Class)implementations.get(key);
    if(c == null) {
    	return new DefaultFoo();
    }
    try {
    	return (Foo)c.newInstance();
    } catch(Exception e) {
    	return new defaultFoo();
    }
}

{% endhighlight %}

###### 使用私有构造函数强化singleton属性

pass

###### 通过私有构造函数强化不可实例化的能力

为了避免默认的构造函数使得一些API无意中进行了实例化,将构造函数声明为私有的

###### 避免创建重复的对象

如果在一个类中创建的对象每次都相同,那么可以将其声明为静态的.
创建对象和不创建对象的权衡:
- 当所创建的对象开销大,应该避免冗余的对象创建
- 在要求保护性拷贝的情况下没有实施保护性拷贝,将会导致潜在的错误和漏洞(拷贝指创建新的对象)

###### 消除过期的对象引用

内存泄露值的是进程中不再用到的内存不能得到及时的释放,java提供的垃圾收集机制不能完全避免内存泄露.
典型的情况就是栈的使用,我们会用数组来保存元素的引用,当一个元素出栈后,这个数组仍然持有对象引用,那么GC就无法将这部分内存回收.

一旦一个***类自己管理内存***,就应该警惕内存泄露问题.另一个常见来源是***缓存***.
策略就是使用WeakHashMap来管理缓存引用,或者及时地将引用置为null,或者开启后台线程及时地清理.
及时的将引用置为null,能够及时抛出nullpointerexception找到程序的错误.

###### 避免使用终结函数(finalizer)

终结函数的调用时机不确定,不同jvm实现有不同的调用方式
关键的系统资源不应该由终结函数来释放,正确的方法是在try-finally块中释放
除非是作为安全网,或者是为了终止非关键的本地资源(本地资源不会由jvm回收),否则不要使用终结函数.

#### 对所有对象都通用的方法

尽管Object类是一个具体类,但是设计它主要是为了扩展.它的所有非final方法(equals,hashCode,toString,clone和finalize)都有明确的通用约定,因为它主要是为了要被重写而设计的.
不遵守约定,其他一些依赖于这些约定的类就无法与这些类结合在一起正常工作.

###### 改写equals的约定
改写equals方法的处方:
1. 使用==操作符检查"实参是否是指向同一个对象的引用"
 对耗时的比较操作的一个优化
2. 使用isntanceof操作符检查"实参是否为正确的类型"
 因为后面的比较可能要进行类型转换.一般"正确的类型"指的是equals方法所在的类.
3. 将实参转换的正确的类型
4. 检查每个需要比较的"关键域"是否相等
5. 检查equals方法是否满足*对称性*,*传递性*和*一致性*
6. 改写equals方法总是要改写hashCode方法
7. 不要将equals声明中的Object对象替换为其他的类型
 这样,父类方法并没有被改写,而约定使用的是参数为Object的方法
8. *关键域*中不要包含不可靠的资源
  如java.net.URL的equals方法依赖于被比较URL中主机的IP地址,而这项工作需要访问网络,不能保证每次得到的IP地址都相同
  
###### 改写equals时总要改写hashCode

许多散列容器类依赖与hashCode来找到正确的"桶",当我们在散列容器里面查找元素时候,先使用hashCode找到正确的桶,这个桶上维护一个链表,寻找相应的元素,使用equals比较两个元素是否相等.

把冗余域排除在hashCode计算之外是可以接受的,但是关键域最好都考虑进来,以获取一个好的散列分布.

###### 总是要改写toString方法

toString方法可以给编程带来更多的方便,其中一定要返回感兴趣的信息.

###### 谨慎地改写clone

实现clonable接口的类,在类的内部调用super.clone()递归调用父类的clone,整个调用链一直到Object类.如果实现了它,那么将返回实现cloneable接口的类的一个拷贝,这个拷贝默认情况下是浅拷贝.
这个接口改变了父类Object的行为,如果没有实现这个接口,将抛出异常.

一般使用拷贝构造函数,或者等价的静态工厂方法来代替clone.

###### 考虑实现Comparable接口

comparable接口中的compareTo方法有如下的规范:
> 如果一个对象小于,等于,大于另一个对象,那么当前对象的compareTo方法返回负整数,零或者正整数.如果因为制定的类型而导致无法比较,则抛出ClassCastException

所有依靠Comparable接口的实现包括:排序集合类(TreeSet, TreeMap, sort),在遵循comparable规范同时,最好遵循equals规范(当equals返回true, compareTo返回0).

compareTo的实现中,从最关键的域开始比较,当比较结果为0时继续进行其他域的比较,否则返回.

#### 类和接口

###### 使类和成员的可访问能力最小化

public的类是公有类,作为包API的一部分,没有public修饰的类是包级访问权限的,由包内部使用
类成员包含private, protected, public, package-private访问权限. public和protected修饰的成员是API的一部分
尽量减少公有域(相对于公有方法)的声明,非可变的公有域(final修饰的域且指向的对象不可变)被接受

###### 支持非可变性

非可变对象是线程安全的,它们不要求同步.因此,它们是很好的组件,使得新类的状态的不变性更容易维护.
共享非可变对象特别简单,因为不需要同步.同样,共享它们的内部信息也是可能的.例如,两个符号相反的BigInteger可以共享一个数组表示数字.
非可变类的唯一缺点是:对于每个不同的值,都要构造一个单独的对象.
因此,在非可变对象的实现内部,都会提供一个可变的"配套类",它们一般是包级私有的.
相对于String类,StringBuffer是可变配套类;相对于BigInteger,BitSet是对应的可变配套类.

如果一个类不能被做成非可变类,仍然应该尽可能限制它的可变性.
构造函数应该创建完全初始化的对象,所有的约数关系应该在这时候建立起来.

###### 复合优先于继承

在考虑应该使用*继承*还是使用*复合*的时候,考虑下面方面
同一个包里面的类可以使用继承,因为一个包的实现者一般是同一个人,他清楚实现中的每个细节.
如果是跨包之间的继承,且超类不是为了继承而设计的,那么最好使用复合.
子类的作者和超类的作者很可能不是一个人,他不清楚超类中的每个方法是如何实现的,以及将来的版本将会如何改变,因为超类的作者并不保证每个方法的实现在将来不会改变.
典型的例子如下,一个集合要扩展成能够记录总共有多少次插入的次数.因此,它继承HashSet,并且改写了add和addAll方法,如下 :
{% highlight java %}
public class IncrementHashSet extends HashSet {
	private int addCount = 0;
    public IncrementHashSet(Collection c) {
    	super(c);
    }
    public boolean add(Object 0) {
    	addCount++;
        return super.add(o);
    }
    public boolean addAll(Collection c) {
    	addCount += c.size();
        return super.addAll(C);
    }
    public int getAddCount() {
    	return addCount;
    }
    // 其他部分
    .....
}
{% endhighlight %}
问题在于,在HashSet内部,addAll是通过调用add方法实现的.在调用子类的addAll方法时,count会加,然后调用父类的addAll,父类的addAll调用add,因为多态性,子类的add方法被调用,此时的count再次被计数.
如果重新实现超类方法,很困难或者很耗时,并且某些域对于子类来说不可访问,因此这些方法在子类中无法重新实现.
更糟糕的是,如果超类在后续版本中增加了新的方法,并且子类的某些方法声明更超类新方法相同,那么可能造成不希望发生的*重写*.

避免这些问题的办法是使用复合,在新的类中增加一个私有域,它引用了超类的一个实例.声明新类的转发方法(直接调用超类方法的方法),然后在这个类中进行扩展.

###### 要么专门为继承而设计,并给出文档说明,要么禁止继承

该类的文档必须精确地描述了改写每一个方法所带来的影响
文档中应该明确方法的自用模式(self-use pattern),以及对于受保护方法和域所隐含的实现细节.实际上已经作出了永久的承诺,这将使得将来的版本的修改受到的限制会更多.

如java.util.AbstractCollection的规范:

> This implementation iterates over the collection looking for the specified element. If it finds the element, it removes the element from the collection using the iterator's remove method. Note that this implementation throws an UnsupportedOperationException if the iterator returned by this collection's iterator method does not implement the remove method.

该文档就很清晰的说明了,改写了iterator方法将会影响remove方法的行为,而且确切描述了iterator方法返回的Iterator的行为将会怎样影响remove方法的行为.

为了允许继承,构造函数一定不能调用可被改写的方法.当超类的构造函数调用了可改写方法时,当子类调用超类构造方法时候,调用的可改写方法将会是重写后的子类方法,子类方法可能试图访问子类的域,构造函数在这个阶段还没对这个域进行初始化.

###### 接口优于抽象类

接口是mixin(混合类型)的理想选择.
在继承层次的类型框架中,一种类型非此即彼.如何表示诸如又是作家又是音乐家这种类型,那么可能就要有4中组合方式.
使用接口可以避免组合爆炸,一般公布一个接口之后,还会伴随一个接口的***抽象骨架实现***,命名为Abstract*Interface*,其中Interface是接口的名字.这样,能够编写公有的一些接口实现.
任何想要组合接口的子类,可以使用组合的方式来使用抽象骨架实现
抽象骨架实现包括AbstractCollection, AbstractSet, AbstractList和AbstractMap
接口的演化比抽象类的演化要难得多,当给接口增加一个新的方法,现有的任何实现该接口的代码都会编译报错.当增加方法时,给相应的抽象骨架实现也添加这个新方法,可以部分缓解这种情况.

###### 接口只是被用于定义类型

pass

###### 优先考虑静态成员类

嵌套类是指被定义在另一个类的内部的类.

嵌套类如果是非静态的,那么它的每个实例会包含一个外围类的引用.

Map的集合视图使用非静态内部类实现,这些视图是keySet,entrySet和valueSet.诸如Set和List这样的集合接口的实现往往也使用非静态成员类来实现它们的迭代器:
{% highlight java %}
public class MySet extends AbstractSet {
	...// bulk of the class ommited
    
    public Iterator iterator() {
    	return new MyIterator();
    }
    
    private class MyIterator implements Iterator {
    	...
    }
}
{% endhighlight %}

私有静态成员通常与偶那个发是用来代表外围类的对象组件.例如,Map实例需要把key和value关联起来,Map中使用Entry对象将key-value对象关联起来.因为这个对象不需要访问外部类方法和域,因此将其声明为静态内部类

对于匿名类,它是在被使用的点上同时被声明和实例化的,取决于实例化时所处的环境:如果出现在静态环境中,它是静态的.如果出现在非静态环境中,则它有一个外围类实例.

###### 用类代替结构

pass

###### 用类层次来代替联合

union的使用可以使用类层次结构来解决(pass)

###### 用类来代替enum结构

{% highlight java %}
public class Suit implements Comparable {
	private static int nextOrdinal = 0;
    
    private final int ordinal = nextOrdinal++;
    
    private Suit(String name) { this.name = name; }
    
    public String toString() { return name; }
    
    public int compareTo(Object o) {
    	return ordinal - ((Suit)o).ordinal;
	}
    
    
    public static final Suit CLUBS = new suit("clubs");
    public static final Suit DIAMONDS = new suit("diamonds");
    public static final Suit HEARTS = new suit("hearts");
    public static final Suit SPADES = new suit("spades");
    
}
{% endhighlight %}

在要求使用一个枚举类型的环境下,首先应该考虑类型安全枚举模式.

###### 用类和接口来代替函数指针

c语言中的函数指针主要用来实现"策略模式".java中使用策略类实现.
比如,要实现一个比较器策略(比较函数的指针),声明一个表示比较器的接口:
{% highlight java %}
public interface Comparator {
	public int compare(Object o1, Object o2);
}
{% endhighlight %}

在使用比较器(传入函数指针时候)的时候,定义一个具体类,传入这个具体类的实例,其中具体类使用了单例模式.如下:
{% highlight java %}
class StringLengthComparator {
	private StringLengthComparator() {}
    public static final StringLengthComparator
    	INSTANCE = new StringLengthCOmparator();
        
    public int compare(Object o1, Object o2) {
    	String s1 = (String)o1, s2 = (String)o2;
        return s1.length()-s2.length();
    }
}
{% endhighlight %}

如果一个具体策略类需要被到出去重复使用,那么它的类通常是一个私有的静态成员类,并且通过一个公有的静态final域被导出去,其类型为该策略的接口.上面的代码中,就是将INSTANCE声明为Comparator接口.

#### 方法

###### 检查参数的有效性

编写一个方法或者构造函数的时候,应该考虑对于它的参数有哪些限制.这样可以把可能错误尽早的发现.
但是对于包访问范围的方法,你可以控制这个方法传入的参数,因此这时候传入非法的操作是不应该被容许的,因此更适合使用assertions来确保参数的正确性.

###### 需要时使用保护性拷贝

对象传入时候,以及对象返回时候.

###### 谨慎设计方法的原型

- 谨慎选择方法的名字
- 不要过于追求提供便利的方法
- 避免长长的参数列表,类型相同的长参数序列尤其有害
 解决方案之一是将一个方法分解为多个方法,之二是使用一个辅助类,一般是内部静态类来封装参数.
- 对于参数类型,优先使用接口
- 谨慎使用函数对象

###### 谨慎的使用重载

声明一个方法时,尽量避免导出具有相同参数数目的重载方法.
如果这个无法避免,那么要保证两个方法的参数列表中至少有一对参数是互不相关的.
"互不相关"指的是在继承层次上其中一个不是另一个的子类.

###### 返回零长度的数组而不是null

返回null会让客户端代码使用多余的代码来处理特殊情况.返回的零长度数组,本质上是非可变对象,因此可以使用单例来保证返回相同的数组.

###### 为所有导出的API元素编写文档注释

pass

#### 通用程序设计

###### 将局部变量的作用域最小化

- 在第一次使用时声明局部变量
- 一般情况下包含一个初始化表达式,try语句块除外
- 尽量使用for类型循环语句
- 将方法块的大小缩小

###### 了解和使用库

pass

###### 如果要求精确的答案,请避免使用float和double
这个程序试图买10,20,30,40美分的产品,结果是无法买第四个.
使用double的问题,如下:
{% highlight java %}
public static void main(String args[]) {
	double funds = 1.00;
    int itemsBought = 0;
    for(double price = 0.10; funds >= price; price += .10) {
    	funds -= price;
        itemsBought ++;
    }
}
{% endhighlight %}
在支付了3个产品之后,发现余额为0.3999999....

在有精度要求的场合,使用BigDecimal或者int或者long更为合理.
BigDecimal提供了更高精度的计算方法.
int和long的用法是将小数点之后的数字提到之前,最后再除以相应的数.

###### 如果其他类型更适合,则尽量避免使用字符串

pass

###### 了解字符串连接的性能

pass

###### 通过接口引用对象

pass

###### 接口优于反射机制

pass

###### 谨慎的使用本地代码

pass

###### 遵守普遍接受的命名惯例

pass

#### 异常

###### 只针对不正常的条件才使用异常

如果一个类具有一个"状态相关"的方法,例如Iterator.next方法.那么应该提供对应的"状态测试方法,如Iterator.hasNext方法,或者返回一个可识别的返回值,如null.这里不适用返回值,因为null是一个合法的值.
如果一个对象可以被并发访问,那么最好使用"可识别的返回值",因为在测试状态和调用状态相关方法之间状态可能被并发的修改.其他情况下,状态测试方法提供了更好的可读性.

###### 对于可恢复的条件使用被检查的异常,对于程序错误使用运行时异常

java的三种可抛出结构:被检查的异常(checked exception), 运行时异常(runtime exception)和错误(error).

被检查异常一般期望调用者能够解决,而运行时异常则不然.
不管怎样,如果异常能够在被调用的地方得到解决,那么异常本身应该提供足够的信息给调用者.
如一个余额不足的异常,应该提供一个方法查询所需填补的空缺.

###### 避免不必要使用受检查的异常

java中RuntimeException以及它的子类都属于**未受检查的异常**,除了这些异常,其他抛出的异常都必须做到:
- 要么继续抛出
- 要么在代码中catch

因此,除非我们期望在调用方能够解决异常,否则我们应该抛出未受检查的异常,并且提供相应的方法来测试可行性.如果**状态测试方法**或者**可区分的返回值**

###### 尽量使用标准的异常

基本上,所有的错误的方法调用都可以被归结为**非法参数**或**非法状态**.
java中提供了可重用的一些异常,都是被检查的异常,如下:
- IllegalArgumentException
- IllegalStateException
- NullPointerException
- IndexOutOfBoundsException
- ConcurrentModificationException
- UnsupportedOperationException

###### 抛出的异常要适合于相应的抽象

当一个方法传递一个由底层抽象抛出的异常时,抛出的异常可能与它所执行的任务没有明显的关联关系.
如果高层的实现在后续的发行版本中发生了变化,那么它所抛出的异常也可能会跟着发生变化.
因此,编写高层代码时候,要试图让高层的异常也抽象出来.

高层的实现应该捕获底层的异常,同时抛出一个可以按照高层抽象进行解释的异常,这种做法称为***异常转译***, 如下例子取自AbstractSequentialList类,由于List接口中get方法的规范要求,转译是必须的:
{% highlight java %}
public Object get(int index) {
	ListIterator i = listIterator(index);
    try {
    	return i.next();
    } catch(NoSuchElementException e) {
    	throw new IndexOutOfBoundException("Index: " + index);
    }
}
{% endhighlight %}

另一种特殊形式的转译称为***异常链接***:
{% highlight java %}
try {
	// use low-level abstraction to do our bidding
} catch(LowerLevelException e) {
	throw new HigherLevelException(e);
}
{% endhighlight %}

这样会记录下每一层被捕获的异常.


###### 每个方法抛出的文档都要有文档

pass

###### 在细节消息中包含失败-捕获信息

如IndexOutOfBoundsException并不是有一个String构造函数,而是有一个这样的构造函数:
{% highlight java %}
public IndexOutOfBoundsException(int lowerBound, int upperBound, int index);
{% endhighlight %}

###### 努力使失败保持原子性

一个失败的方法调用应该使对象保持"它在被调用之前的状态",这种属性的方法称作具有"失败原子性".
四中方法可以保持失败原子性:
1. 保持对象非可变性
2. 在调用方法之前抛出异常
3. 编写一段恢复代码
4. 在对象的临时拷贝上执行操作.(如Collections.sort)

#### 线程

###### 对共享可变数据的同步访问
对于共享的原子数据,还是应该实施同步,因为一个线程的写入值不能保证被另一个线程是可见的.

volatile关键词修饰的原子变量可以保证任何最近的写操作可以被其他线程"看到".

###### 避免过多的同步

为避免死锁,不要在同步方法或者代码块中调用可被外部代码改变的方法(如public, protected).

###### 永远不要在循环的外面调用wait

Object.wait方法的作用是使一个线程等待某个条件
使用wait的标准模式如下:
{% highlight java %}
synchronized (obj) {
	while(<condition does not hold>)
    	obj.wait();
    .... // perform action appropriate to condition
}
{% endhighlight %}

notify方法会试图唤醒一个线程,而notifyAll会唤醒所有线程,因此在被唤醒后继续检查条件很有必要.同时,检查条件可以保证在睡眠之前条件得到满足,则跳过睡眠阶段.

###### 线程安全性的文档化

pass

###### 避免使用线程组

pass

#### 序列化

  `