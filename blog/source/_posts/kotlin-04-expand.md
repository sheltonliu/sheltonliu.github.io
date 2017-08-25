---
title: kotlin-04-类和对象(扩展、数据类、密封类)
date: 2017-08-22 19:09:30
categories: "kotlin语言文档"
tags:
	- "kotlin"
---

# 1.扩展

<font size=4>

Kotlin 同 C# 和 Gosu 类似,能够扩展一个类的新功能而无需继承该类或使用像装饰者这样的任何类型的设计模式。这通过叫做_扩展_的特殊声明完成。 Kotlin 支持扩展函数 和 扩展属性。


## 1.1扩展函数
声明一个扩展函数,我们需要用一个 接收者类型 也就是被扩展的类型来作为他的前缀。下面代码为 MutableList<Int> 添加一个 swap 函数:	fun MutableList<Int>.swap(index1: Int, index2: Int) { 
		val tmp = this[index1] // “this”对应该列表 		
		this[index1] = this[index2]		this[index2] = tmp	}

这个this关键字在扩展函数内部对应到接收者对象(传过来的在点符号前的对象)现在,我们对任意 MutableList<Int> 调用该函数了:	￼val l = mutableListOf(1, 2, 3)	l.swap(0, 2) // “swap()”内部的“this”得到“l”的值当然,这个函数对任何 MutableList<T> 起作用,我们可以泛化它:

	fun <T> MutableList<T>.swap(index1: Int, index2: Int) { 
		val tmp = this[index1] // “this”对应该列表 
		this[index1] = this[index2]			this[index2] = tmp	}	





## 1.2扩展是静态解析的
扩展不能真正的修改他们所扩展的类。通过定义一个扩展,你并没有在一个类中插入新成员,仅仅是可以通过该类型的变量用点表达式去调用这个新函 数。

我们想强调的是扩展函数是静态分发的,即他们不是根据接收者类型的虚方法。这意味着调用的扩展函数是由函数调用所在的表达式的类型来决定的,而 不是由表达式运行时求值结果决定的。例如:

	open class C 
	
	class D: C()	
	fun C.foo() = "c" 
	
	fun D.foo() = "d"	
	fun printFoo(c: C) { 
		println(c.foo())	} 
	
	printFoo(D()) //结果是"c".  非运行时的结果

这个例子会输出 "c",因为调用的扩展函数只取决于 参数 c 的声明类型,该类型是 C 类。

如果一个类定义有一个成员函数和一个扩展函数,而这两个函数又有相同的接收者类型、相同的名字 并且都适用给定的参数,这种情况总是取成员函数。例如:

	class C {		fun foo() { println("member") }	}	fun C.foo() { println("extension") }

如果我们调用 C 类型 c 的 c.foo() ,它将输出“member”,而不是“extension”。

当然,扩展函数重载同样名字但不同签名成员函数也完全可以:

	class C {		fun foo() { println("member") }	}	
	fun C.foo(i: Int) { println("extension") }

调用 C().foo(1) 将输出"extension"。


## 1.3可空接收者
注意可以为可空的接收者类型定义扩展。这样的扩展可以在对象变量上调用,即使其值为null,并且可以在函数体内检测 this == null,这能让你在 没有检测 null 的时候调用 Kotlin 中的toString():检测发生在扩展函数的内部。

	fun Any?.toString(): String {		if (this == null) return "null"		// 空检测之后,“this”会自动转换为非空类型,所以下面的 toString() 
		// 解析为 Any 类的成员函数		return toString()	}



## 1.4扩展属性
和函数类似,Kotlin 支持扩展属性:

	val <T> List<T>.lastIndex: Int 
		get() = size - 1

注意:由于扩展没有实际的将成员插入类中,因此对扩展属性来说 幕后字段是无效的。这就是为什么扩展属性不能有 初始化器。他们的行为只能由显式提 供的 getters/setters 定义。

例如：

	val Foo.bar = 1 // 错误:扩展属性不能有初始化器
	
	

## 1.5伴生对象的扩展
如果一个类定义有一个伴生对象 ,你也可以为伴生对象定义 扩展函数和属性:

	class MyClass {		companion object { } // 将被称为 "Companion"	}	fun MyClass.Companion.foo() { // ......	}

就像伴生对象的其他普通成员,只需用类名作为限定符去调用他们

	MyClass.foo()



## 1.6扩展的作用域

大多数时候我们在顶层定义扩展,即直接在包里:

	package foo.bar	
	fun Baz.goo() { ...... }


要使用所定义包之外的一个扩展,我们需要在调用方导入它:	
	
	package com.example.usage	
	import foo.bar.goo // 以名字 "goo" 导入所有扩展 // 或者	import foo.bar.* // 从 "foo.bar" 导入一切	fun usage(baz: Baz) { 
		baz.goo()	)

## 1.7扩展声明为成员
在一个类内部你可以为另一个类声明扩展。在这样的扩展内部,有多个 隐式接收者 ⸺ 其中的对象成员可以无需通过限定符访问。扩展声明所在的类的实 例称为 分发接收者,扩展方法调用所在的接收者类型的实例称为 扩展接收者 。￼
	
	class D {		fun bar() { ...... }	}	
	class C {		fun baz() { ...... }		fun D.foo() {			bar() // 调用 D.bar 
			baz() // 调用 C.baz		}
			fun caller(d: D) {			d.foo() // 调用扩展函数		} 
	}
	
对于分发接收者和扩展接收者的成员名字冲突的情况,扩展接收者 优先。要引用分发接收者的成员你可以使用 限定的 this 语法。	
	class C {		fun D.foo() {			toString() // 调用 D.toString()			this@C.toString() // 调用 C.toString() 
		}
	}

声明为成员的扩展可以声明为 open 并在子类中覆盖。这意味着这些函数的分发对于分发接收者类型是虚拟的,但对于扩展接收者类型是静态的。

	open class D { }	
	class D1 : D() { }	
	open class C {		open fun D.foo() {			println("D.foo in C") 
		}		open fun D1.foo() { 
			println("D1.foo in C")		}		fun caller(d: D) {			d.foo() // 调用扩展函数		} 
	}
		class C1 : C() {		override fun D.foo() {        	println("D.foo in C1")    	}	
		override fun D1.foo() { 
			println("D1.foo in C1")		} 
	}
		C().caller(D()) // 输出 "D.foo in C" 
	C1().caller(D()) // 输出 "D.foo in C1 分发接收者虚拟解析
	C().caller(D1()) // 输出 "D.foo in C" 扩展接收者静态解析
	

# 2.数据类
我们经常创建一些只保存数据的类。在这些类中,一些标准函数往往是从 数据机械推导而来的。在 Kotlin 中,这叫做 数据类 并标记为 data :	data class User(val name: String, val age: Int)编译器自动从主构造函数中声明的所有属性导出以下成员:

* — equals() / hashCode() 对,* — toString() 格式是 "User(name=John, age=42)", 
* — componentN() 函数 按声明顺序对应于所有属性,* — copy() 函数(⻅下文)。如果这些函数中的任何一个在类体中显式定义或继承自其基类型,则不会生成该函数。

为了确保生成的代码的一致性和有意义的行为,数据类必须满足以下要求:* — 主构造函数需要至少有一个参数;* — 主构造函数的所有参数需要标记为 val 或 var ; 
* — 数据类不能是抽象、开放、密封或者内部的; 
* —(在1.1之前)数据类只能实现接口。

在 JVM 中,如果生成的类需要含有一个无参的构造函数,则所有的属性必须指定默认值。(参⻅构造函数)。	data class User(val name: String = "", val age: Int = 0)



## 2.1复制

在很多情况下,我们需要复制一个对象改变它的一些属性,但其余部分保持不变。copy() 函数就是为此而生成。对于上文的 User 类,其实现会类似下 面这样:

	fun copy(name: String = this.name, age: Int = this.age) = User(name, age)

这让我们可以写

	val jack = User(name = "Jack", age = 1) 
	val olderJack = jack.copy(age = 2)	
	
	

## 2.2数据类和解构声明

为数据类生成的 Component 函数 使它们可在解构声明中使用:

	val jane = User("Jane", 35)	val (name, age) = jane	println("$name, $age years of age") // 输出 "Jane, 35 years of age"

	
## 2.3标准数据类
标准库提供了 Pair 和 Triple 。尽管在很多情况下命名数据类是更好的设计选择,因为它们通过为属性提供有意义的名称使代码更具可读性。


# 3.密封类

密封类用来表示受限的类继承结构:当一个值为有限集中的 类型、而不能有任何其他类型时。在某种意义上,他们是枚举类的扩展:枚举类型的值集合 也是 受限的,但每个枚举常量只存在一个实例,而密封类 的一个子类可以有可包含状态的多个实例。

要声明一个密封类,需要在类名前面添加 sealed 修饰符。虽然密封类也可以 有子类,但是所有子类都必须在与密封类自身相同的文件中声明。(在 Kotlin 1.1 之前,该规则更加严格:子类必须嵌套在密封类声明的内部)。

	sealed class Expr	data class Const(val number: Double) : Expr()	data class Sum(val e1: Expr, val e2: Expr) : Expr() 	object NotANumber : Expr()	fun eval(expr: Expr): Double = when (expr) { 
		is Const -> expr.number		is Sum -> eval(expr.e1) + eval(expr.e2) 
		NotANumber -> Double.NaN	}

请注意,扩展密封类子类的类(间接继承者)可以放在任何位置,而无需在 同一个文件中。

使用密封类的关键好处在于使用 when 表达式 的时候,如果能够 验证语句覆盖了所有情况,就不需要为该语句再添加一个 else 子句了。

	fun eval(expr: Expr): Double = when(expr) {		is Expr.Const -> expr.number		is Expr.Sum -> eval(expr.e1) + eval(expr.e2) 		
		Expr.NotANumber -> Double.NaN		// 不再需要 `else` 子句,因为我们已经覆盖了所有的情况	}

