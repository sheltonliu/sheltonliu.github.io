---
title: kotlin-05-类和对象(泛型、嵌套类、枚举类)
date: 2017-08-26 14:47:01
categories: "kotlin语言文档"
tags:
	- "kotlin"
---

# 泛型
<font size=4>

与 Java 类似,Kotlin 中的类也可以有类型参数:

	class Box<T>(t: T) { 
		var value = t	}

一般来说,要创建这样类的实例,我们需要提供类型参数:
	
	val box: Box<Int> = Box<Int>(1)

但是如果类型参数可以推断出来,例如从构造函数的参数或者从其他途径,允许省略类型参数:

	val box = Box(1) // 1 具有类型 Int,所以编译器知道我们说的是 Box<Int>。
	

## 型变

	// Java	List<String> strs = new ArrayList<String>();	List<Object> objs = strs; // !即将来临的问题的原因就在这里。Java 禁止这样! 
	objs.add(1); // 这里我们把一个整数放入一个字符串列表	String s = strs.get(0); // !ClassCastException:无法将整数转换为字符串


因此,Java 禁止这样的事情以保证运行时的安全。但这样会有一些影响。例如,考虑 Collection 接口中的 addAll() 方法。该方法的签名应该是什 么?直觉上,我们会这样:

	// Java	interface Collection<E> ...... {		void addAll(Collection<E> items);	}
	
但随后,我们将无法做到以下简单的事情(这是完全安全):

	// Java	void copyAll(Collection<Object> to, Collection<String> from) { 
		to.addAll(from); // !对于这种简单声明的 addAll 将不能编译:
		//Collection<String> 不是 Collection<Object> 的子类型	}


这就是为什么 addAll() 的实际签名是以下这样:
	
	// Java	interface Collection<E> ...... {		void addAll(Collection<? extends E> items);	}


### 声明处型变

在 Kotlin 中,有一种方法向编译器解释这种情况。这称为声明处型变:我们可以标注 Source 的类型参数 T 来确保它仅从 Source<T> 成员中返回(生 产),并从不被消费。为此,我们提供 out 修饰符:	abstract class Source<out T> { 
		abstract fun nextT(): T	}
		fun demo(strs: Source<String>) {		val objects: Source<Any> = strs // 这个没问题,因为 T 是一个 out-参数 // ......	}

关于Out操作符，我的理解就是：Source<? extends T> 可以向上转型为T。

in 操作符	

	abstract class Comparable<in T> { 
		abstract fun compareTo(other: T): Int	}
		fun demo(x: Comparable<Number>) {		x.compareTo(1.0) // 1.0 拥有类型 Double,它是 Number 的子类型 
		// 因此,我们可以将 x 赋给类型为 Comparable <Double> 的变量		val y: Comparable<Double> = x // OK!	}

关于in操作符，可以向下转型

## 类型投影

将类型参数 T 声明为 out 非常方便,并且能避免使用处子类型化的麻烦,但是有些类实际上不能限制为只返回 T !一个很好的例子是 Array:	class Array<T>(val size: Int) {		fun get(index: Int): T { ///* ...... */ }		fun set(index: Int, value: T) { ///* ...... */ 
		
		}	}该类在 T 上既不能是协变的也不能是逆变的。这造成了一些不灵活性。考虑下述函数:	fun copy(from: Array<Any>, to: Array<Any>) { 
		assert(from.size == to.size)		for (i in from.indices)
			to[i] = from[i]	}	
这个函数应该将项目从一个数组复制到另一个数组。让我们尝试在实践中应用它:

	val ints: Array<Int> = arrayOf(1, 2, 3)	val any = Array<Any>(3)	copy(ints, any) // 错误:期望 (Array<Any>, Array<Any>)

	
那么,我们唯一要确保的是 copy() 不会做任何坏事。我们想阻止它写到 from ,我们可以:	fun copy(from: Array<out Any>, to: Array<Any>) { 
		// ......	}	
	结合上面的代码，out关键字：对应于 Java 的 Array<? extends Object>		
	
你也可以使用 in 投影一个类型:

	fun fill(dest: Array<in String>, value: String) { 
		// ......	}	

Array<in String> 对应于 Java 的 Array<? super String> ,也就是说,你可以传递一个 CharSequence 数组或一个 Object 数组给 fill() 函数。	
	
## 星投影

Kotlin 为此提供了所谓的星投影语法:	— 对于 Foo <out T> ,其中 T 是一个具有上界 TUpper 的协变类型参数,
	Foo <*> 等价于 Foo <out TUpper> 。
	这意味着当 T 未知时,你可以安全地从 Foo <*> 读取 TUpper 的值。
	— 对于 Foo <in T>,其中 T 是一个逆变类型参数,
	Foo <*> 等价于 Foo <in Nothing>。
	这意味着当 T 未知时,没有什么可以以安全的方式写入 Foo <*> 。	— 对于 Foo <T>,其中 T 是一个具有上界 TUpper 的不型变类型参数,
	Foo<*> 对于读取值时等价于 Foo<out TUpper> 而对于写值时等价于Foo<in Nothing> 。
	
	
	
如果泛型类型具有多个类型参数,则每个类型参数都可以单独投影。例如,如果类型被声明为 interface Function <in T, out U>,我们可以想象以下星投影:

	— Function<*, String> 表示 Function<in Nothing, String> ; 
	
	— Function<Int, *> 表示 Function<Int, out Any?> ;	— Function<*, *> 表示 Function<in Nothing, out Any?>。## 泛型函数

不仅类可以有类型参数。函数也可以有。类型参数要放在函数名称之前:

	fun <T> singletonList(item: T): List<T> { 
		// ......	}	fun <T> T.basicToString() : String { // 扩展函数 
		// ......	}		
	
	
要调用泛型函数,在调用处函数名之后指定类型参数即可:

	val l = singletonList<Int>(1)


## 泛型约束
能够替换给定类型参数的所有可能类型的集合可以由泛型约束限制。

### 上界

最常⻅的约束类型是与 Java 的 extends 关键字对应的 上界:

	fun <T : Comparable<T>> sort(list: List<T>) { 
		// ......	}

冒号之后指定的类型是上界:只有 Comparable<T> 的子类型可以替代 T 。例如

	sort(listOf(1, 2, 3)) // OK。Int 是 Comparable<Int> 的子类型
		sort(listOf(HashMap<Int, String>())) // 错误:HashMap<Int, String> 不是 Comparable<HashMap<Int, String>> 的子 类型


默认的上界(如果没有声明)是 Any? 。在尖括号中只能指定一个上界。如果同一类型参数需要多个上界,我们需要一个单独的 where-子句:

	fun <T> cloneWhenGreater(list: List<T>, threshold: T): List<T> 
		where T : Comparable,			   T : Cloneable {		return list.filter { it > threshold }.map { it.clone() }	}
	


# 嵌套类

类可以嵌套在其他类中

	class Outer {		private val bar: Int = 1 
		class Nested {			fun foo() = 2 
		}	}	
	
	val demo = Outer.Nested().foo() // == 2

## 内部类

类可以标记为 inner 以便能够访问外部类的成员。内部类会带有一个对外部类的对象的引用:

	class Outer {		private val bar: Int = 1 
		inner class Inner {			fun foo() = bar 
		}
	}	val demo = Outer().Inner().foo() // == 1

参⻅限定的 this 表达式以了解内部类中的 this 的消歧义用法。

## 匿名内部类

使用对象表达式创建匿名内部类实例:

	window.addMouseListener(object: MouseAdapter() { 
		override fun mouseClicked(e: MouseEvent) {			// ......		}
			override fun mouseEntered(e: MouseEvent) { 
			// ......		} 
	})

如果对象是函数式 Java 接口(即具有单个抽象方法的 Java 接口)的实例,你可以使用带接口类型前缀的lambda表达式创建它：

	val listener = ActionListener { println("clicked") }
	


# 枚举类

枚举类的最基本的用法是实现类型安全的枚举

	enum class Direction { 
		NORTH, SOUTH, WEST, EAST	}	

每个枚举常量都是一个对象。枚举常量用逗号分隔。

## 初始化
	
因为每一个枚举都是枚举类的实例,所以他们可以是初始化过的。

	enum class Color(val rgb: Int) { 
		RED(0xFF0000),
		GREEN(0x00FF00),
		BLUE(0x0000FF)	}

## 匿名类
枚举常量也可以声明自己的匿名类

	enum class ProtocolState { 
		WAITING {			override fun signal() = TALKING 
		},
				TALKING {			override fun signal() = WAITING		};
				abstract fun signal(): ProtocolState 
	}

及相应的方法、以及覆盖基类的方法。注意,如果枚举类定义任何 成员,要使用分号将成员定义中的枚举常量定义分隔开,就像 在 Java 中一样。## 枚举常量

就像在 Java 中一样,Kotlin 中的枚举类也有合成方法允许列出 定义的枚举常量以及通过名称获取枚举常量。这些方法的 签名如下(假设枚举类的名称是EnumClass ):

	￼EnumClass.valueOf(value: String): EnumClass 
	EnumClass.values(): Array<EnumClass>


如果指定的名称与类中定义的任何枚举常量均不匹配,valueOf() 方法将抛出 IllegalArgumentException 异常。 

自 Kotlin 1.1 起,可以使用 enumValues<T>() 和 enumValueOf<T>() 函数以泛型的方式访问枚举类中的常量 :

	enum class RGB { RED, GREEN, BLUE }	
	inline fun <reified T : Enum<T>> printAllValues() { 
		print(enumValues<T>().joinToString { it.name })	}	printAllValues<RGB>() // 输出 RED, GREEN, BLUE


每个枚举常量都具有在枚举类声明中获取其名称和位置的属性:

	val name: String 
	val ordinal: Int

枚举常量还实现了 Comparable 接口,其中自然顺序是它们在枚举类中定义的顺序。









			