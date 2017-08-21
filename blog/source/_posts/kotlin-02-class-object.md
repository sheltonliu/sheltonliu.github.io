---
title: kotlin-02类和对象(类和继承)
date: 2017-08-20 19:02:53
categories: "kotlin语言文档"
tags:
	- "kotlin"
---

# 类
<font size=4>

Kotlin 中使用关键字 class 声明类

	class Invoice {
	}

类声明由类名、类头(指定其类型参数、主 构造函数等)和由大括号包围的类体构成。类头和类体都是可选的;如果一个类没有类体,可以省略花括号。	class Empty

	

## 1.构造函数
在 Kotlin 中的一个类可以有一个主构造函数和一个或多个次构造函数。主 构造函数是类头的一部分:它跟在类名(和可选的类型参数)后。	￼class Person constructor(firstName: String) { 
	}如果主构造函数没有任何注解或者可⻅性修饰符,可以省略这个 constructor 关键字。

	￼class Person(firstName: String) {
	}	

主构造函数不能包含任何的代码。初始化的代码可以放 到以 init 关键字作为前缀的初始化块(initializer blocks)中:

	class Customer(name: String) { 
		init {			logger.info("Customer initialized with value ${name}") 
		}	}

注意,主构造的参数可以在初始化块中使用。它们也可以在 类体内声明的属性初始化器中使用:

	class Customer(name: String) {		val customerKey = name.toUpperCase()	}

事实上,声明属性以及从主构造函数初始化属性,Kotlin 有简洁的语法:

	class Person(val firstName: String, val lastName: 		String, var age: Int) { // ......	}

与普通属性一样,主构造函数中声明的属性可以是 可变的(var)或只读的(val)。

如果构造函数有注解或可⻅性修饰符,这个 constructor 关键字是必需的,并且 这些修饰符在它前面:	class Customer public @Inject constructor(name: String) { ...... }

次构造函数

类也可以声明前缀有 constructor** 的次构造函数**:

	class Person { 
		constructor(parent: Person) {			parent.children.add(this) 
		}
	}

如果类有一个主构造函数,每个次构造函数需要委托给主构造函数,可以直接委托或者通过别的次构造函数间接委托。委托到同一个类的另一个构造函数 用 this 关键字即可:

	class Person(val name: String) {		constructor(name: String, parent: Person) : this(name) {			parent.children.add(this) 
		}	}

如果一个非抽象类没有声明任何(主或次)构造函数,它会有一个生成的 不带参数的主构造函数。构造函数的可⻅性是 public。如果你不希望你的类 有一 个公有构造函数,你需要声明一个带有非默认可⻅性的空的主构造函数:

	￼class DontCreateMe private constructor () { 
	}

注意:在 JVM 上,如果主构造函数的所有的参数都有默认值,编译器会生成 一个额外的无参构造函数,它将使用默认值。这使得 Kotlin 更易于使用 像 Jackson 或者 JPA 这样的通过无参构造函数创建类的实例的库。

	class Customer(val customerName: String = "")


构造函数的例子：

	class Customer constructor(val name:String){
		//类体内声明的属性
		val upName = name.toUpperCase();
		//初始化块
		init {
    		println("初始化块...upName..."+upName)
    	}
    	//次构造函数,通过this关键字,委托给主构造函数
    	constructor(name:String, age:Int) : this(name){
			println("次构造函数..."+name+"   "+age)
		}
		
		fun testOther(){
    		println("函数体...")
    	}
    }



## 2.创建类的实例
要创建一个类的实例,我们就像普通函数一样调用构造函数:

	val invoice = Invoice()	val customer = Customer("Joe Smith")

注意 Kotlin 并没有 new 关键字。

创建嵌套类、内部类和匿名内部类的类实例在嵌套类中有述。



## 3.类成员

类可以包含* — 构造函数和初始化块 
* — 函数* — 属性* — 嵌套类和内部类* — 对象声明





# 继承
在 Kotlin 中所有类都有一个共同的超类 Any ,这对于没有超类型声明的类是默认超类:

	class Example // 从 Any 隐式继承

Any 不是 java.lang.Object ;尤其是,它除了 equals() 、hashCode() 和 toString() 外没有任何成员。更多细节请查阅Java互操作性部分。 

要声明一个显式的超类型,我们把类型放到类头的冒号之后:

	open class Base(p: Int)	
	class Derived(p: Int) : Base(p) //冒号继承

如果该类有一个主构造函数,其基类型可以(并且必须)用(基类型的)主构造函数参数就地初始化。

如果类没有主构造函数,那么每个次构造函数必须 使用 super 关键字初始化其基类型,或委托给另一个构造函数做到这一点。注意,在这种情况下,不同 的次构造函数可以调用基类型的不同的构造函数:

	class MyView : View {		constructor(ctx: Context) : super(ctx)		constructor(ctx: Context, attrs: AttributeSet) : super(ctx, attrs) 
	}

## open关键字

类上的 open 标注与 Java 中 final 相反,它允许其他类 从这个类继承。默认情况下,在 Kotlin 中所有的类都是 final,对应于 Effective Java书中的 第 17 条:要么为继承而设计,并提供文档说明,要么就禁止继承。


## 覆盖方法

	open class Base { 
		open fun v() {}		fun nv() {} 
	}	class Derived() : Base() { 
		override fun v() {}	}

Derived.v()函数上必须加上override标注。如果没写,编译器将会报错。如果函数没有标注open如 Base.nv(),则子类中不允许定义相同签名的函 数,不论加不加 override。在一个 final 类中(没有用 open 标注的类),开放成员是禁止的。标记为 override 的成员本身是开放的,也就是说,它可以在子类中覆盖。如果你想禁止再次覆盖,使用 final 关键字:	open class AnotherDerived() : Base() { 
		final override fun v() {} //禁止再次覆盖	}

## 覆盖属性

属性覆盖与方法覆盖类似;在超类中声明然后在派生类中重新声明的属性必须以 override 开头,并且它们必须具有兼容的类型。每个声明的属性可以由 具有初始化器的属性或者具有 getter 方法的属性覆盖。	open class Foo {		open val x: Int get { ...... }	}	class Bar1 : Foo() { 
		override val x: Int = ......	}你也可以用一个 var 属性覆盖一个 val 属性,但反之则不行。这是允许的,因为一个 val 属性本质上声明了一个 getter 方法,而将其覆盖为 var 只 是在子类中额外声明一个 setter 方法。


在主构造函数中使用 override 关键字作为属性声明的一部分。

	interface Foo { 
		val count: Int	}	//在主构造函数中使用 override	class Bar1(override val count: Int) : Foo	
	class Bar2 : Foo {		override var count: Int = 0	}		


## 覆盖规则
### super关键字
	
	open class A {		open fun f() { print("A") } 
		fun a() { print("a") }	}	interface B {		fun f() { print("B") } // 接口成员默认就是“open”的 
		fun b() { print("b") }	}	class C() : A(), B {		// 编译器要求覆盖 f(): 
		override fun f() {			super<A>.f() // 调用 A.f()			super<B>.f() // 调用 B.f() 
		}	}


# 抽象类
类和其中的某些成员可以声明为 abstract。抽象成员在本类中可以不用实现。需要注意的是,我们并不需要用 open 标注一个抽象类或者函数⸺因 为这不言而喻。

我们可以用一个抽象成员覆盖一个非抽象的开放成员

	open class Base { 
		open fun f() {}	}	abstract class Derived : Base() { 
		override abstract fun f()	}



# 伴生对象

与 Java 或 C# 不同,在 Kotlin 中类没有静态方法。在大多数情况下,它建议简单地使用 包级函数。 

如果你需要写一个可以无需用一个类的实例来调用、但需要访问类内部的 函数(例如,工厂方法),你可以把它写成该类内对象声明 中的一员。 

更具体地讲,如果在你的类内声明了一个伴生对象,你就可以使用像在 Java/C# 中调用静态方法相同的语法来调用其成员,只使用类名 作为限定符。




