---
title: kotlin-06-对象表达式/对象声明/委托
date: 2017-09-01 12:29:54
categories: "kotlin语言文档"
tags:
	- "kotlin"
---

参考kotlin中文网站：

[kotlin中文网站](https://www.kotlincn.net/docs/reference/)


# 1.对象表达式和对象声明
<font size=4>
## 1.1对象表达式
要创建一个继承自某个(或某些)类型的匿名类的对象,我们会这么写:

	window.addMouseListener(object : MouseAdapter() { 
		override fun mouseClicked(e: MouseEvent) {
			// ......
	})


如果超类型有一个构造函数,则必须传递适当的构造函数参数给它。多个超类型可以由跟在冒号后面的逗号分隔的列表指定:
	 
	 	override val y = 15

任何时候,如果我们只需要“一个对象而已”,并不需要特殊超类型,那么我们可以简单地写:
		   }
	   }

请注意,匿名对象可以用作只在本地和私有作用域中声明的类型。如果你使用匿名对象作为公有函数的 返回类型或者用作公有属性的类型,那么该函数或 属性的实际类型 会是匿名对象声明的超类型,如果你没有声明任何超类型,就会是 Any 。在匿名对象 中添加的成员将无法访问。


	 class C {
		private fun foo() = object {
		}
		
		fun publicFoo() = object {
		}
	 }

表达式中的代码可以访问来自包含它的作用域的变量

	 fun countClicks(window: JComponent) { 
	 	var clickCount = 0
			override fun mouseClicked(e: MouseEvent) {
        	
				enterCount++
		})


	 



## 1.2对象声明

单例模式是一种非常有用的模式,而 Kotlin(继 Scala 之后)使单例声明变得很容易:

	 object DataProviderManager {
			get() = // ......

这称为对象声明。并且它总是在 object 关键字后跟一个名称。就像变量声明一样,对象声明不是一个表达式,不能用在赋值语句的右边。


要引用该对象,我们直接使用其名称即可:

	  DataProviderManager.registerDataProvider(......)
	  
	  
这些对象可以有超类型:

	  object DefaultListener : MouseAdapter() { 
	  		override fun mouseClicked(e: MouseEvent) {
			
				// ......
	  }


注意:对象声明不能在局部作用域(即直接嵌套在函数内部),但是它们可以嵌套到其他对象声明或非内部类中。



### 1.2.1伴生对象
类内部的对象声明可以用 companion 关键字标记:

	  class MyClass {
	  	  }


该伴生对象的成员可通过只使用类名作为限定符来调用:

	  val instance = MyClass.create()


可以省略伴生对象的名称,在这种情况下将使用名称 Companion :

	  class MyClass { 
	  		companion object { }
	  


请注意,即使伴生对象的成员看起来像其他语言的静态成员,在运行时他们 仍然是真实对象的实例成员,而且,例如还可以实现接口:

	  interface Factory<T> { 
	  	  fun create(): T
	  
		  }


当然,在 JVM 平台,如果使用 @JvmStatic 注解,你可以将伴生对象的成员生成为真正的 静态方法和字段。更详细信息请参⻅Java 互操作性一节 。



### 1.2.2对象表达式和声明之间的差异






# 2.委托
## 2.1类委托

委托模式已经证明是实现继承的一个很好的替代方式,而 Kotlin 可以零样板代码地原生支持它。类 Derived 可以继承一个接口 Base ,并将其所有共 有的方法委托给一个指定的对象:

	  interface Base { 
	  	  fun print()
	  
	  	  override fun print() { print(x) }
	  
	  	  val b = BaseImpl(10) Derived(b).print() // 输出 10
	
Derived的超类型列表中的 by-子句表示 b 将会在 Derived 中内部存储。并且编译器将生成转发给 b 的所有 Base 的方法。



	

# 3.委托属性

有一些常⻅的属性类型,虽然我们可以在每次需要的时候手动实现它们,但是如果能够为大家把他们只实现一次并放入一个库会更好。例如包括

	  class Example {
		  	  return "$thisRef, thank you for delegating '${property.name}' to me!" 
		  }
		  
		  	  println("$value has been assigned to '${property.name} in $thisRef.'")
	  }

	  ￼val e = Example() 
	  println(e.p)
	  //输出结果：Example@33a17727, thank you for delegating ‘p’ to me!
	  //输出结果：NEW has been assigned to ‘p’ in Example@33a17727.
## 3.1标准委托

Kotlin 标准库为几种有用的委托提供了工厂方法。

### 3.1.1延迟属性Lazy

* lazy() 是接受一个 lambda 并返回一个 Lazy <T> 实例的函数,返回的实例可以作为实现延迟属性的委托:第一次调用 get() 会执行已传递给



		  val lazyValue: String by lazy { 
	  		  println("computed!") 
	  		  "Hello"
			  println(lazyValue) 
			  println(lazyValue)
		
		  
		这个例子输出:




如果初始化委托的同步锁不是必需的,这样多个线程 可以同时执行,那么将 LazyThreadSafetyMode.PUBLICATION 作为参数传递给 lazy() 函数。

而如果你确定初始化将总是发生在单个线程,那么你可以使用 LazyThreadSafetyMode.NONE 模式,它不会有任何线程安全的保证和相关的开销。

Delegates.observable() 接受两个参数:初始值和修改时处理程序(handler)。每当我们给属性赋值时会调用该处理程序(在赋值后执行)。它有三

	  import kotlin.properties.Delegates
		  		prop, old, new ->
	  
		  val user = User() 
		  user.name = "first" 
		  user.name = "second"
	  
	  这个例子输出:


如果你想能够截获一个赋值并“否决”它,就使用 vetoable() 取代 observable() 。在属性被赋新值生效之前会调用传递给 vetoable 的处理程 序。





## 3.2把属性存储在映射中

一个常⻅的用例是在一个映射(map)里存储属性的值。这经常出现在像解析 JSON 或者做其他“动态”事情的应用中。在这种情况下,你可以使用映射实 例自身作为委托来实现委托属性。


	  class User(val map: Map<String, Any?>) { 
	  	  val name: String by map

在这个例子中,构造函数接受一个映射参数:

	  val user = User(mapOf( 
	  	  "name" to "John Doe", 
	  	  "age" to 25


委托属性会从这个映射中取值(通过字符串键⸺属性的名称):

	￼println(user.name) // Prints "John Doe" 	
	println(user.age) // Prints 25


这也适用于var属性,如果把只读的 Map 换成 MutableMap 的话:
	  	  var name: String by map



## 3.3局部委托属性（自1.1起）

你可以将局部变量声明为委托属性。例如,你可以使一个局部变量惰性初始化:
	  	  val memoizedFoo by lazy(computeFoo)
		  	  memoizedFoo.doSomething()
	  }

memoizedFoo 变量只会在第一次访问时计算。如果 someCondition 失败,那么该变量根本不会计算。
## 3.4属性委托要求

这里我们总结了委托对象的要求。
	* — property ⸺ 必须是类型 KProperty<*> 或其超类型,



* 对于一个可变属性(即var声明的),委托必须额外提供一个名为 setValue 的函数,该函数接受以下参数:
	  
		  operator fun setValue(thisRef: R, property: KProperty<*>, value: T)






### 3.4.1翻译规则

在每个委托属性的实现的背后,Kotlin 编译器都会生成辅助属性并委托给它。例如,对于属性 prop ,生成隐藏属性 prop$delegate ,而访问器的代码 只是简单地委托给这个附加属性:

	  class C {
	  
	  class C {
		  var prop: Type
			  set(value: Type) = prop$delegate.setValue(this, this::prop, value)	


Kotlin 编译器在参数中提供了关于 prop 的所有必要信息:第一个参数 this 引用到外部类 C 的实例而 this::prop 是 KProperty 类型的反射 对象,该对象描述 prop 自身。



### 3.4.2提供委托(自1.1起)
通过定义 provideDelegate 操作符,可以扩展创建属性实现所委托对象的逻辑。如果 by 右侧所使用的对象将 provideDelegate 定义为成员或 扩展函数,那么会调用该函数来 创建属性委托实例。

例如,如果要在绑定之前检查属性名称,可以这样写:

	  class ResourceLoader<T>(id: ResourceID<T>) { 
	  	  operator fun provideDelegate(
		  }
	  }
	  
	  
		  val text by bindResource(ResourceID.text_id)


￼￼provideDelegate 的参数与 getValue 相同:



如果没有这种拦截属性与其委托之间的绑定的能力,为了实现相同的功能,你必须显式传递属性名,这不是很方便:


	  // 检查属性名称而不使用“provideDelegate”功能 
	  class MyUI {
	  }
	  
	  	  id: ResourceID<T>,
	  ): ReadOnlyProperty<MyUI, T> {
	  }



在生成的代码中,会调用 provideDelegate 方法来初始化辅助的 prop$delegate 属性。比较对于属性声明 val prop: Type by MyDelegate() 生成的代码与 上面(当 provideDelegate 方法不存在时)生成的代码:

	  class C {
	  
	  // 由编译器生成的代码:
		  val prop: Type


请注意,provideDelegate 方法只影响辅助属性的创建,并不会影响为 getter 或 setter 生成的代码。












