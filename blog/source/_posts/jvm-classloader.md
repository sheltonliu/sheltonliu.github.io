---
title: 虚拟机类加载机制
date: 2017-08-06 18:13:06
categories: "java虚拟机"
tags:
	- "深入理解java虚拟机"
	- "jvm"
---
<font size=4 >

# 类加载的时机

类从被加载到虚拟机内存开始，到卸载出内存开始，生命周期包括七个阶段

![类加载工程](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/07/14/android_classloader_process.png)

其中”验证“，”准备“，”解析“三个部分统称为连接(Linking)

对于初始化，虚拟机是严格规定了有且只有四种情况必须立即对类进行”初始化“

* 1.遇到 new, getstatic, putstatic, 或invokestatic 这4条字节码指令，如果没有初始化则先初始化。这4条指令对应的java代码场景是：
  * 使用 new 关键字实例化对象的时候
  * 读取或设置一个类的静态字段(被final修饰，已在编译期把结果放入常量池的静态字段除外)
  * 调用一个类的静态方法的时候

* 2.使用 java.long.reflect包的方法对类进行反射调用，如果类没有初始化则先初始化 

* 3.当初始化类时，如发现父类没有初始化，则先初始化父类

* 4.当虚拟机启动，用户需要指定一个执行的主类(包含main()方法的那个类)，虚拟机会先初始化这个主类。

这四种场景称为对一个类进行主动引用。除此之外所有引用称为被动引用，都不会触发初始化。
下面的代码是被动引用的例子

	package org.fenixsoft.classloading;

	/**
 	* 被动使用类字段演示一：
	* 通过子类引用父类的静态字段，不会导致子类初始化
 	**/
	public class SuperClass {

		static {
			System.out.println("SuperClass init!");//会输出
		}

		public static int value = 123;
	}

	public class SubClass extends SuperClass {

		static {
			System.out.println("SubClass init!");//不会输出
		}
	}

	/**
 	* 非主动使用类字段演示
 	**/
	public class NotInitialization {

		public static void main(String[] args) {
			System.out.println(SubClass.value);
		}

	}

上面代码只触发父类初始化，子类不会初始化。对于静态字段，只有直接定义这个字段的类才会被初始化。



	package org.fenixsoft.classloading;

	/**
 	* 被动使用类字段演示二：
	* 通过数组定义来引用类，不会触发此类的初始化
 	**/
	public class NotInitialization {

		public static void main(String[] args) {
			SuperClass[] sca = new SuperClass[10];
		}

	}

上述代码没有输出"SuperClass init!".这段代码触发了名为”[Lorg.fenixsoft.classloading.SuperClass“的类的初始化，这不是合法的类名称，是由java虚拟机自动生成的，直接继承与Object的子类，创建动作由字节码指令newarray触发。

这个类代表了一个元素类型为”org.fenixsoft.classloading.SuperClass"的一维数组。
	
	
	package org.fenixsoft.classloading;

	/**
 	* 被动使用类字段演示三：
	* 常量在编译阶段会存入调用类的常量池中，本质上没有直接引用到定义常量的类，因此不会触发定	义常量的类的初始化。
 	**/
	public class ConstClass {

		static {
			System.out.println("ConstClass init!");//不会输出
		}

		public static final String HELLOWORLD = "hello world";
	}

	/**
 	* 非主动使用类字段演示
 	**/
	public class NotInitialization {

		public static void main(String[] args) {
			System.out.println(ConstClass.HELLOWORLD);
		}
	}

上述代码没有输出”ConstClass init!“因为在编译阶段将此常量的值"hello world"存储到NotInitialization类的常量池中了。对常量ConstClass.HELLOWORLD的引用转化成NotInitialization类对自身常量池的引用了。也就是说NotInitialization的class文件之中没有ConstClass类的符号引用入口，这两个类在编译成class之后就不存在任何联系了。

## 接口与类加载的过程有不同，主要区别在于：
* 当一个类在初始化时，要求其父类进行初始化。但是当一个接口在初始化时，并不要求其父接口全部都完成初始化，只有在真正使用到父接口的时候(如引用接口中定义的常量)才会初始化。


# 类加载器
## 类与类加载器
比较两个类是否"相等"，只有在这两个类是由同一个类加载器加载的前提下才有意义。否则即使两个类是来源同一个class文件，只要加载它们的类加载器不同，那两个类就必定不相等。

这里的”相等“包括代表类的class对象的equals()方法，isAssignableFrom(), isInstance()方法的返回结果，也包括了使用instanceof关键字做对象所属关系判定等情况。
		
	/**
 	* 类加载器与instanceof关键字演示
 	* 
 	* @author zzm
 	*/
	public class ClassLoaderTest {

    	public static void main(String[] args) throws Exception {

        	ClassLoader myLoader = new ClassLoader() {
            	@Override
            	public Class<?> loadClass(String name) throws 	ClassNotFoundException {
                try {
                    String fileName = name.substring(name.lastIndexOf(".") + 1) + ".class";
                    InputStream is = getClass().getResourceAsStream(fileName);
                    if (is == null) {
                        return super.loadClass(name);
                    }
                    byte[] b = new byte[is.available()];
                    is.read(b);
                    return defineClass(name, b, 0, b.length);
                } catch (IOException e) {
                    throw new ClassNotFoundException(name);
                }
            }
        };

        Object obj = myLoader.loadClass("org.fenixsoft.classloading.ClassLoaderTest").newInstance();

        System.out.println(obj.getClass());
        System.out.println(obj instanceof org.fenixsoft.classloading.ClassLoaderTest);
    }
	}
	
运行结果：
class org.fenixsoft.classloading.ClassLoaderTest
false

虚拟机中存在两个ClassLoaderTest类，类加载器不同。


# 双亲委派模型

* 启动类加载器(Bootstrap ClassLoader): C++语言实现，是虚拟机自身的一部分。负责将存放在(JAVA_HOME\lib)目录中的类库加载到虚拟机内存中。启动器类加载器无法被java程序直接引用。

* 扩展类加载器(Extension ClassLoader): 负责加载(JAVA_HOME\lib\ext)目录中的，或者被java.ext.dirs系统变量所指定的路径中的所有类库，开发者可直接使用扩展类加载器。

* 应用程序类加载器(Application ClassLoader)：由于这个类加载器是ClassLoader中的getSystemClassLoader()方法的返回值，所以一般也称为系统类加载器。它负责加载用户类路径(ClassPath)上所指定的类库，开发者可直接使用这个类加载器。如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。

* 自定义类加载器


![模型](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/07/14/android_classloader.png)

* 流程是当一个classlaoder尝试加载一个类时，它会先抛给父加载器，一层层的往上抛，抛到最上层启动类加载器。如果启动类加载器无法加载，这时会自上而下的抛给其他类加载器，直到可以加载为止


双亲委派模型的实现

	protected synchronized Class<?> loadClass(String name, boolean resolve) 	throws ClassNotFoundException {
		//首先，检查请求的类是否已经被加载过了
		Class c = findLoadedClass(name);
		if (c == null){
			try{
				if (parent != null){
					c = parent.loadClass(name,false);
				}else {
					c = findBootstrapClassOrNull(name);
				}
			}catch(ClassNotFoundException e){
				//如果父类加载器抛出异常
				//说明父类加载器无法完成加载请求
			}
			if(c == null){
				//在父类加载器无法加载的时候
				//再调用本身的findClass方法来进行类加载
				c = findClass(name);
			}
		}
		if (resolve){
			resolveClass(c);
		}
		return c ; 
	}
 

采用双亲委派模式的优点：

* 避免重复加载，如果已经加载过一次class,就不需要再次加载，而是直接读取已加载的class
* 更加安全。
	* 如果不使用双亲委托模式，就可以自定义一个String的类来替换系统String类，显然造成安全隐患
	* 当采用双亲委托模式，会使得系统String类在java虚拟机启动的时候就被加载，这样就无法通过自定义的类来替换系统类了。
	* 只有两个类名一致，并且被同一个类加载器加载的类，java虚拟机才会认为它们是同一个类。
	
	

# Android中的类加载器

### 系统类加载器
* BootClassLoader(java代码实现)
	* 在Zygote进程的Zygote入口方法中被创建，用于加载preloaded-classes文件中的预加载类

* DexClassLoader
	* 继承BaseDexClassLoader. 可以加载dex文件，以及包含dex的压缩文件(apk和jar文件)

* PathClassLoader
	* 继承BaseDexClassLoader. PathClassLoader的构造方法中，没有optimizedDirectory参数，因为已经默认值为/data/dalvik-cache. 所以PathClassLoader无法定义解压dex的存储路径。因此PathClassLoader通常加载已经安装的apk的dex文件(安装的APK的dex文件会存储在/data/dalvik-cache中)
	


### 自定义类加载器	  


# Java和Android的类加载器的差异
* java的引导类加载器是由C++编写的，Android的引导类加载器是由java编写的
* Android的继承关系要比java的继承关系更复杂些，提供的功能更多
* Android加载的不再是class文件，所以没有ExtClassLoader和AppClassLoader. 替换它们的是PatchClassLoader和DexClassLoader


参考：

《深入理解java虚拟机》周志明 著
《Android进阶解密》

	




	
	








