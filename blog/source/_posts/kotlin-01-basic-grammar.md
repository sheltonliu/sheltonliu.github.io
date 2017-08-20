---
title: kotlin基础(基本类型、控制流)
date: 2017-08-17 23:22:46
categories: "kotlin语言文档"
tags:
	- "kotlin"
---

# 基础

<font size=4>

## 基本类型

### 数字
Kotlin 处理数字在某种程度上接近 Java,但是并不完全相同。例如,对于数字没有隐式拓宽转换(如 Java 中 int 可以隐式转换为 long ⸺译者注),另 外有些情况的字面值略有不同。

Kotlin 提供了如下的内置类型来表示数字(与 Java 很相近):	Type Bit width	Double	64 
	Float	32 
	Long	64 
	Int		32 
	Short	16 
	Byte	8

注意在 Kotlin 中字符不是数字

#### 字面常量

数值常量字面值有以下几种: 

* — 十进制: 123* — Long 类型用大写 L 标记: 123L 
* — 十六进制: 0x0F* — 二进制: 0b00001011注意: 不支持八进制Kotlin 同样支持浮点数的常规表示方法:* — 默认 double:123.5 、123.5e10 
* — Float 用 f 或者 F 标记: 123.5f

#### 数字字面值中的下划线(自 1.1 起)
你可以使用下划线使数字常量更易读:

	val oneMillion = 1_000_000	val creditCardNumber = 1234_5678_9012_3456L	val socialSecurityNumber = 999_99_9999L	val hexBytes = 0xFF_EC_DE_5E	val bytes = 0b11010010_01101001_10010100_10010010


#### 表示方式
在 Java 平台数字是物理存储为 JVM 的原生类型,除非我们需要一个可空的引用(如 Int? )或泛型。后者情况下会把数字装箱。

注意数字装箱不必保留同一性:
	
	val a: Int = 10000	print(a === a) // 输出“true”	val boxedA: Int? = a	val anotherBoxedA: Int? = a	print(boxedA === anotherBoxedA) // !!!输出“false”!!!

另一方面,它保留了相等性:

	val a: Int = 10000	print(a == a) // 输出“true”	val boxedA: Int? = a	val anotherBoxedA: Int? = a	print(boxedA == anotherBoxedA) // 输出“true”

#### 显式转换
由于不同的表示方式,较小类型并不是较大类型的子类型。如果它们是的话,就会出现下述问题:
	// 假想的代码,实际上并不能编译:	val a: Int? = 1 // 一个装箱的 Int (java.lang.Integer)	val b: Long? = a // 隐式转换产生一个装箱的 Long (java.lang.Long)	print(a == b) // 惊!这将打印 "false" 鉴于 Long 的 equals() 检测其他部分也是 Long	
所以同一性还有相等性都会在所有地方悄无声息地失去。

因此较小的类型不能隐式转换为较大的类型。这意味着在不进行显式转换的情况下我们不能把 Byte 型值赋给一个 Int 变量。

	val b: Byte = 1 // OK, 字面值是静态检测的 
	val i: Int = b // 错

我们可以显式转换来拓宽数字

	val i: Int = b.toInt() // OK: 显式拓宽


每个数字类型支持如下的转换:

	— toByte(): Byte	— toShort(): Short	— toInt(): Int	— toLong(): Long	— toFloat(): Float	— toDouble(): Double — toChar(): Char

缺乏隐式类型转换并不显著,因为类型会从上下文推断出来,而算术运算会有重载做适当转换,例如:

	val l = 1L + 3 // Long + Int => Long

#### 运算

Kotlin支持数字运算的标准集,运算被定义为相应的类成员(但编译器会将函数调用优化为相应的指令)。参⻅运算符重载。 

对于位运算,没有特殊字符来表示,而只可用中缀方式调用命名函数,例如:

	val x = (1 shl 2) and 0x000FF000

这是完整的位运算列表(只用于 Int 和 Long ):

	— shl(bits) ‒ 有符号左移 (Java 的 << ) 
	— shr(bits) ‒ 有符号右移 (Java 的 >> ) 
	— ushr(bits) ‒ 无符号右移 (Java 的 >>> ) 
	— and(bits) ‒位与	— or(bits) ‒位或	— xor(bits) ‒位异或 
	— inv() ‒位非	

### 字符

字符用 Char 类型表示。它们不能直接当作数字

	fun check(c: Char) {		if (c == 1) { // 错误:类型不兼容		// ......	} }
	
字符字面值用单引号括起来: '1' 。特殊字符可以用反斜杠转义。支持这几个转义序列:\t 、\b 、\n 、\r 、\' 、\" 、\\ 和 \$\$。编码其他字符要用 Unicode 转义序列语法:'\uFF00' 。	

我们可以显式把字符转换为 Int 数字:

	fun decimalDigitValue(c: Char): Int {
	 	if (c !in '0'..'9') //字符
	 		throw IllegalArgumentException("Out of range") 
	 	return c.toInt() - '0'.toInt() // 显式转换为数字	}
当需要可空引用时,像数字、字符会被装箱。装箱操作不会保留同一性。

### 布尔

布尔用 Boolean 类型表示,它有两个值:true 和 false。

若需要可空引用布尔会被装箱。

内置的布尔运算有:

	— || ‒短路逻辑或 
	— && ‒短路逻辑与 
	— ! -逻辑非	

### 数组

数组在 Kotlin 中使用 Array 类来表示,它定义了 get 和 set 函数(按照运算符重载约定这会转变为 [] )和 size 属性,以及一些其他有用的成员 函数:

	class Array<T> private constructor() { val size: Int		operator fun get(index: Int): T		operator fun set(index: Int, value: T): Unit		operator fun iterator(): Iterator<T>		// ......	}

我们可以使用库函数 arrayOf() 来创建一个数组并传递元素值给它,这样 arrayOf(1, 2, 3) 创建了 array [1, 2, 3]。或者,库函数 arrayOfNulls() 可以用于创建一个指定大小、元素都为空的数组。另一个选项是用接受数组大小和一个函数参数的工厂函数,用作参数的函数能够返回 给定索引的每个元素初始值:

	//创建一个 Array<String> 初始化为 ["0", "1", "4", "9", "16"]
	val asc = Array(5, { i -> (i * i).toString() })

如上所述,[] 运算符代表调用成员函数 get() 和 set() 。

注意: 与 Java 不同的是,Kotlin 中数组是不型变的(invariant)。这意味着 Kotlin 不让我们把 Array<String> 赋值给 Array<Any> ,以防止可能的运行时失败(但是你可以使用 Array<out Any> , 参⻅类型投影)。

Kotlin 也有无装箱开销的专⻔的类来表示原生类型数组: ByteArray 、ShortArray 、IntArray 等等。这些类和 Array 并没有继承关系,但是 它们有同样的方法属性集。它们也都有相应的工厂方法:
	
	val x: IntArray = intArrayOf(1, 2, 3)
	
	x[0] = x[1] + x[2]


### 字符串
字符串用 String 类型表示。字符串是不可变的。字符串的元素⸺字符可以使用索引运算符访问: s[i] 。可以用 for 循环迭代字符串:	for (c in str) {
		 println(c)	}

#### 字符串字面值　
Kotlin 有两种类型的字符串字面值: 转义字符串可以有转义字符,以及原生字符串可以包含换行和任意文本。转义字符串很像 Java 字符串:	val s = "Hello, world!\n"
	
转义采用传统的反斜杠方式。参⻅上面的 字符 查看支持的转义序列。原生字符串 使用三个引号( """ )分界符括起来,内部没有转义并且可以包含换行和任何其他字符:

	val text = """	for (c in "foo")	print(c) """	你可以通过 trimMargin() 函数去除前导空格:	val text = """		|Tell me and I forget. 
		|Teach me and I remember. 
		|Involve me and I learn. 
		|(Benjamin Franklin) """.trimMargin()默认 | 用作边界前缀,但你可以选择其他字符并作为参数传入,比如 trimMargin(">") 。 


#### 字符串模板

	val i = 10
	val s = "i = $i" // 求值结果为 i = 10

或者用花括号扩起来的任意表达式:

	val s = "abc"
	val str = "$s.length is ${s.length}" // 求值结果为 "abc.length is 3"
	
原生字符串和转义字符串内部都支持模板。如果你需要在原生字符串中表示字面值 "$"字符(它不支持反斜杠转义),你可以用下列语法:￼
	val price = """ 
	${'$'}9.99	"""	


## 控制流

### 1.if表达式

在 Kotlin 中,if是一个表达式,即它会返回一个值。因此就不需要三元运算符(条件 ? 然后 : 否则),因为普通的 if 就能胜任这个⻆色。

	// 传统用法	var max = a	if (a < b) max = b	
	// With else	var max: Int
	if (a > b) {		max = a 
	} else {		max = b 
	}	
	// 作为表达式	val max = if (a > b) a else b
	

if的分支可以是代码块,最后的表达式作为该块的值:

	val max = if (a > b){
		print("Choose a") 
		a	} else { 
		print("Choose b") 
		b	}

如果你使用if作为表达式而不是语句(例如:返回它的值或者把它赋给变量),该表达式需要有 else 分支	
	


### 2.when表达式

when 取代了类 C 语言的 switch 操作符。其最简单的形式如下:

	when (x) {		1 -> print("x == 1") 
		2 -> print("x == 2") 
		else -> { // 注意这个块			print("x is neither 1 nor 2") 
		}	}

when 将它的参数和所有的分支条件顺序比较,直到某个分支满足条件。when 既可以被当做表达式使用也可以被当做语句使用。如果它被当做表达式,符 合条件的分支的值就是整个表达式的值,如果当做语句使用,则忽略个别分支的值。(像 if 一样,每一个分支可以是一个代码块,它的值 是块中最后的表 达式的值。)如果其他分支都不满足条件将会求值 else 分支。如果 when 作为一个表达式使用,则必须有 else 分支,除非编译器能够检测出所有的可能情况都已经 覆盖了。

如果很多分支需要用相同的方式处理,则可以把多个分支条件放在一起,用逗号分隔:	when (x) {		0, 1 -> print("x == 0 or x == 1") 
		else -> print("otherwise")	}
	

我们可以用任意表达式(而不只是常量)作为分支条件

	when (x) {		parseInt(s) -> print("s encodes x") 
		else -> print("s does not encode x")	}

我们也可以检测一个值在(in)或者不在(!in)一个区间或者集合中:

	when (x) {		in 1..10 -> print("x is in the range")		in validNumbers -> print("x is valid")		!in 10..20 -> print("x is outside the range") 		else -> print("none of the above")	}
	
另一种可能性是检测一个值是(is)或者不是(!is)一个特定类型的值。注意:由于智能转换,你可以访问该类型的方法和属性而无需 任何额外的检测。	fun hasPrefix(x: Any) = when(x) {		is String -> x.startsWith("prefix") 
		else -> false	}
	
when 也可以用来取代 if-else if链。如果不提供参数,所有的分支条件都是简单的布尔表达式,而当一个分支的条件为真时则执行该分支:	when {		x.isOdd() -> print("x is odd") 
		x.isEven() -> print("x is even") 
		else -> print("x is funny")	}
	

### 3.for循环

for 循环可以对任何提供迭代器(iterator)的对象进行遍历,语法如下:

	for (item in collection) print(item)
	
循环体可以是一个代码块。

	for (item: Int in ints) { 
		// ......
	}			

如上所述,for 可以循环遍历任何提供了迭代器的对象。即:* — 有一个成员函数或者扩展函数 iterator() ,它的返回类型* — 有一个成员函数或者扩展函数 next() ,并且* — 有一个成员函数或者扩展函数 hasNext() 返回 Boolean 。这三个函数都需要标记为 operator 。对数组的 for 循环会被编译为并不创建迭代器的基于索引的循环。 

如果你想要通过索引遍历一个数组或者一个 list,你可以这么做:

	for (i in array.indices) { 
		print(array[i])	}

注意这种“在区间上遍历”会编译成优化的实现而不会创建额外对象。 或者你可以用库函数 withIndex :

	for ((index, value) in array.withIndex()){  
		println("the element at (美刀符)index is (美刀符)value") 	}		


### 4.While循环

while 和 do..while 照常使用

	while (x > 0) { 
		x--
	}
	
		do {		val y = retrieveData()	} while (y != null) // y 在此处可⻅				
## 返回和跳转

Kotlin 有三种结构化跳转表达式:

* — return。默认从最直接包围它的函数或者匿名函数返回。 
* — break。终止最直接包围它的循环。* — continue。继续下一次最直接包围它的循环。所有这些表达式都可以用作更大表达式的一部分:

	val s = person.name ?: return
	
这些表达式的类型是 Nothing 类型。

	

### 1.Break和Continue标签

在 Kotlin 中任何表达式都可以用标签(label)来标记。标签的格式为标识符后跟 @ 符号,例如:abc@ 、fooBar@ 都是有效的标签(参⻅语法)。要为一个表达式加标签,我们只要在其前加标签即可。

	loop@ for (i in 1..100) { // ......	
	}

现在,我们可以用标签限制 break 或者continue:

	loop@ for (i in 1..100) { 
		for (j in 1..100) {			if (......) break@loop 
		}	}


标签限制的 break 跳转到刚好位于该标签指定的循环后面的执行点。continue 继续标签指定的循环的下一次迭代。


### 2.标签处返回 

Kotlin 有函数字面量、局部函数和对象表达式。因此 Kotlin 的函数可以被嵌套。标签限制的 return 允许我们从外层函数返回。最重要的一个用途就是从 lambda 表达式中返回。回想一下我们这么写的时候:

	fun foo() { 
		ints.forEach {			if (it == 0) return			print(it) 
		}	}	


这个return表达式从最直接包围它的函数即 foo 中返回。(注意,这种非局部的返回只支持传给内联函数的lambda表达式。)如果我们需要从 lambda 表达式中返回,我们必须给它加标签并用以限制 return。

	fun foo() { 
		ints.forEach lit@ {			if (it == 0) return@lit			print(it) 
		}	}

现在,它只会从 lambda 表达式中返回。通常情况下使用隐式标签更方便。该标签与接受该 lambda 的函数同名。

	fun foo() { 
		ints.forEach {			if (it == 0) return@forEach			print(it) 
		}	}

或者,我们用一个匿名函数替代 lambda 表达式。匿名函数内部的 return 语句将从该匿名函数自身返回

	fun foo() { 
		ints.forEach(fun(value: Int) {			if (value == 0) return        	print(value)    	})	}

当要返一个回值的时候,解析器优先选用标签限制的 return,即

	return@a 1

意为“从标签 @a 返回 1”,而不是“返回一个标签标注的表达式 (@a 1) ”。

	
			

		
	
	
		
	
	





