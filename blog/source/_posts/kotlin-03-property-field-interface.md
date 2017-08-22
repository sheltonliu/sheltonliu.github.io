---
title: kotlin-03-类和对象(属性和字段、接口、可见性修饰符)
date: 2017-08-21 16:39:43
categories: "kotlin语言文档"
tags:
	- "kotlin"
---

# 1.属性和字段
<font size=4>

## 1.1声明属性

Kotlin的类可以有属性。属性可以用关键字var 声明为可变的,否则使用只读关键字val。

	class Address {		var name: String = ...... 
		var street: String = ...... 
		var city: String = ...... 
		var state: String? = ...... 
		var zip: String = ......	}
	
要使用一个属性,只要用名称引用它即可,就像 Java 中的字段:

	fun copyAddress(address: Address): Address {		val result = Address() // Kotlin 中没有“new”关键字 		
		result.name = address.name // 将调用访问器 		
		result.street = address.street		// ......		return result	}	

## 1.2Getters和Setters

声明一个属性的完整语法是

	var <propertyName>[: <PropertyType>] [= <property_initializer>] 
		[<getter>]			[<setter>]

其初始器(initializer)、getter 和 setter 都是可选的。属性类型如果可以从初始器(或者从其 getter 返回值,如下文所示)中推断出来,也可以省略。 

例如:

	￼var allByDefault: Int? // 错误:需要显式初始化器,隐含默认 getter 和 setter 
	var initialized = 1 // 类型 Int、默认 getter 和 setter	

一个只读属性的语法和一个可变的属性的语法有两方面的不同:1、只读属性的用 val 开始代替 var 2、只读属性不允许 setter	
	￼val simple: Int? // 类型 Int、默认 getter、必须在构造函数中初始化 
	val inferredType = 1 // 类型 Int 、默认 getter	
自定义getter

	￼val isEmpty: Boolean		get() = this.size == 0	

自定义的 setter 

	var stringRepresentation: String 
		get() = this.toString() 
		set(value) {			setDataFromString(value) // 解析字符串并赋值给其他属性 
		}	

按照惯例,setter 参数的名称是 value ,但是如果你喜欢你可以选择一个不同的名称。

自 Kotlin 1.1 起,如果可以从 getter 推断出属性类型,则可以省略它:

	val isEmpty get() = this.size == 0 // 具有类型 Boolean


如果你需要改变一个访问器的可⻅性或者对其注解,但是不需要改变默认的实现,你可以定义访问器而不定义其实现:

	var setterVisibility: String = "abc"		private set // 此 setter 是私有的并且有默认实现	var setterWithAnnotation: Any? = null 
		@Inject set // 用 Inject 注解此 setter
	
### 幕后字段
Kotlin 中类不能有字段。然而,当使用自定义访问器时,有时有一个幕后字段(backing field)有时是必要的。为此 Kotlin 提供 一个自动幕后字段,它可通 过使用 field 标识符访问。

	var counter = 0 // 此初始器值直接写入到幕后字段 		
		set(value) {			if (value >= 0) field = value		}

field 标识符只能用在属性的访问器内。

如果属性至少一个访问器使用默认实现,或者自定义访问器通过 field 引用幕后字段,将会为该属性生成一个幕后字段。

例如,下面的情况下,就没有幕后字段:

	val isEmpty: Boolean		get() = this.size == 0



### 幕后属性	
如果你的需求不符合这套“隐式的幕后字段”方案,那么总可以使用 幕后属性(backing property):	private var _table: Map<String, Int>? = null 
	public val table: Map<String, Int>		get() {			if (_table == null) {				_table = HashMap() // 类型参数已推断出 
			}			return _table ?: throw AssertionError("Set to null by another thread") 
		}


## 1.3编译期常量
已知值的属性可以使用 const 修饰符标记为编译期常量。这些属性需要满足以下要求:* — 位于顶层或者是 object 的一个成员 
* — 用 String 或原生类型 值初始化* — 没有自定义 getter这些属性可以用在注解中:
	const val SUBSYSTEM_DEPRECATED: String = "This subsystem is deprecated" 
	@Deprecated(SUBSYSTEM_DEPRECATED) fun foo() { ...... }


## 1.4惰性初始化属性

一般地,属性声明为非空类型必须在构造函数中初始化。然而,这经常不方便。例如:属性可以通过依赖注入来初始化,或者在单元测试的 setup 方法中初 始化。这种情况下,你不能在构造函数内提供一个非空初始器。但你仍然想在类体中引用该属性时避免空检查。为处理这种情况,你可以用 lateinit 修饰符标记该属性:

	public class MyTest {		lateinit var subject: TestSubject		@SetUp fun setup() { 
			subject = TestSubject()		}		@Test fun test() { 
			subject.method() // 直接解引用		} 
	}
该修饰符只能用于在类体中(不是在主构造函数中)声明的 var 属性,并且仅 当该属性没有自定义 getter 或 setter 时。该属性必须是非空类型,并且不 能是 原生类型。

在初始化前访问一个 lateinit 属性会抛出一个特定异常,该异常明确标识该属性 被访问及它没有初始化的事实。



# 2.接口
Kotlin 的接口与 Java 8 类似,既包含抽象方法的声明,也包含 实现。与抽象类不同的是,接口无法保存状态。它可以有 属性但必须声明为抽象或提供访问器实现。

使用关键字 interface 来定义接口

	interface MyInterface { 
		fun bar() //没有方法体时默认为抽象方法。		fun foo() {			// 可选的方法体		} 
	}


## 2.1实现接口

一个类或者对象可以实现一个或多个接口。

	class Child : MyInterface { 
		override fun bar() { //实现抽象方法			// 方法体 
		}	}



## 2.2接口中的属性

你可以在接口中定义属性。在接口中声明的属性要么是抽象的,要么提供 访问器的实现。在接口中声明的属性不能有幕后字段(backing field),因此接口 中声明的访问器 不能引用它们。

	interface MyInterface { 
		val prop: Int // 抽象的		val propertyWithImplementation: String 
			get() = "foo"		fun foo() { 
			print(prop)		} 
	}
		class Child : MyInterface { 
		override val prop: Int = 29	}	

## 2.3解决覆盖冲突
实现多个接口时,可能会遇到同一方法继承多个实现的问题。例如

	interface A {		fun foo() { print("A") } 
		fun bar() //没有方法体，抽象方法.	}	interface B {		fun foo() { print("B") } 
		fun bar() { print("bar") }	}
		class C : A {		override fun bar() { print("bar") }	}
		class D : A, B { 
		override fun foo() {			super<A>.foo()			super<B>.foo() 
		}		override fun bar() { 
			super<B>.bar()		} 
	}

上例中,接口 A 和 B 都定义了方法 foo() 和 bar()。两者都实现了 foo(), 但是只有 B 实现了 bar() (bar() 在 A 中没有标记为抽象,因为没有方法体时默认 为抽象)。因为 C 是一个实现了 A 的具体类,所以必须要重写 bar() 并 实现这个抽象方法。
然而,如果我们从 A 和 B 派生 D,我们需要实现 我们从多个接口继承的所有方法,并指明 D 应该如何实现它们。这一规则 既适用于继承单个实现(bar())的 方法也适用于继承多个实现(foo())的方法。



# 3.可见性修饰符
kotlin有四个可见性饰符:private 、protected 、internal 和 public 。如果没有显式指定修饰符的话,默认可⻅性是 public 。 下面将根据声明作用域的不同来解释。

## 3.1包名
函数、属性和类、对象和接口可以在顶层声明,即直接在包内:

	//文件名:example.kt 
	package foo	
	fun baz() {} 
	class Bar {}

* — 如果你不指定任何可⻅性修饰符,默认为 public ,这意味着你的声明 将随处可⻅; 
* — 如果你声明为 private ,它只会在声明它的文件内可⻅;* — 如果你声明为 internal ,它会在相同模块内随处可⻅;* — protected 不适用于顶层声明。

例如：

	// 文件名:example.kt 
	package foo	private fun foo() {} // 在 example.kt 内可⻅ 
	public var bar: Int = 5 // 该属性随处可⻅	private set // setter 只在 example.kt 内可⻅ 
	internal val baz = 6 // 相同模块内可⻅

## 3.2类和接口

对于类内部声明的成员:

* — private 意味着只在这个类内部(包含其所有成员)可⻅;* — protected ⸺ 和 private 一样 + 在子类中可⻅。* — internal ⸺ 能⻅到类声明的 本模块内 的任何客戶端都可⻅其 internal 成员; 
* — public ⸺ 能⻅到类声明的任何客戶端都可⻅其 public 成员。

注意 对于Java用戶:Kotlin 中外部类不能访问内部类的 private 成员。

如果你覆盖一个 protected 成员并且没有显式指定其可⻅性,该成员还会是 protected 可⻅性。

例子：

	open class Outer {		private val a = 1 
		protected open val b = 2 
		internal val c = 3 
		val d=4 //默认public		
		protected class Nested { 
			public val e: Int = 5		} 
	}
		class Subclass : Outer() { 
		// a 不可⻅		// b、c、d 可⻅		// Nested 和 e 可⻅		
		override val b = 5 // “b”为 protected 
	}
		class Unrelated(o: Outer) { 
		// o.a、o.b 不可⻅		// o.c 和 o.d 可⻅(相同模块)		// Outer.Nested 不可⻅,Nested::e 也不可⻅ 
	}
	
构造函数

要指定一个类的的主构造函数的可⻅性,使用以下语法(注意你需要添加一个 显式 constructor 关键字):

	class C private constructor(a: Int) { ...... }	

这里的构造函数是私有的。默认情况下,所有构造函数都是 public,这实际上等于类可⻅的地方它就可⻅(即一个 internal 类的构造函数只能在相 同模块内可⻅).

## 3.3模块
可⻅性修饰符 internal 意味着该成员只在相同模块内可⻅。更具体地说,一个模块是编译在一起的一套 Kotlin 文件:

* — 一个 IntelliJ IDEA 模块;* — 一个 Maven 或者 Gradle 项目;* — 一次 <kotlinc> Ant 任务执行所编译的一套文件。



