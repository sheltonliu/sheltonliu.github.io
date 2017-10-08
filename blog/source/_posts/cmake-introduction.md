---
title: cmake 简介
date: 2017-10-08 12:46:13
categories: "cmake简介"
tags:
	- "cmake"
---
# cmake简介
<font size=4>

以下是CMakeLists.txt的内容

	cmake_minimum_required(VERSION 2.6)

	#myapp:main.o 定义工程名称为myapp 
	PROJECT(myapp)

	#导入头文件
	INCLUDE_DIRECTORIES(
	include
	)

	#指定源文件的目录
	AUX_SOURCE_DIRECTORY(src DIR_SRCS)

	#把可执行的文件放大哪个目录下  SET  方法设置变量
	SET(TEST_MATH ${DIR_SRCS})
	
	#定义了这个工程会生成一个文件名为 myapp 的可执行文件,相关的源文件是 TEST_MATH 中定义的源文件列表
	ADD_EXECUTABLE(${PROJECT_NAME} ${TEST_MATH})

测试的目录解构：

build文件夹(自己新建的)

CMakeLists.txt

include文件夹----->add.h

src文件夹--------->add.c  main.c

1.然后进入build目录(cd build) 然后执行cmake .. 其实就是执行上层目录下CMakeLists.txt中的命令

2 在build目录中会看到 Makefile文件， 然后执行make命令，会生成myapp的可执行文件

3 执行./myapp展现出程序的结果


# 语法

* 1: #注释

* 2:  变量：使用set命令显式定义及赋值，在非if语句中，使用${}引用，if中直接使用变量名引用；后续的set命令会清理变量原来的值；* 3: command (args ...)  #命令不分大小写，参数使用空格分隔，使用双引号引起参数中空格* 4: set(var a;b;c) <=> set(var a b c)  #定义变量var并赋值为a;b;c这样一个string list* 5:	
		Add_executable(${var}) <=> Add_executable(a b c)   #变量使用${xxx}引用* 6: 条件语句：

		if(var) #var 非empty 0 N No OFF FALSE... #非运算使用NOT		       	…		else()/elseif() … endif(var)
	* 7: 循环语句		Set(VAR a b c)		Foreach(f ${VAR})       …Endforeach(f)* 8:循环语句		WHILE() … ENDWHILE()# 内部变量

	CMAKE_C_COMPILER：指定C编译器	CMAKE_CXX_COMPILER：	CMAKE_C_FLAGS：编译C文件时的选项，如-g；也可以通过add_definitions添加编译选项	EXECUTABLE_OUTPUT_PATH：可执行文件的存放路径	LIBRARY_OUTPUT_PATH：库文件路径	CMAKE_BUILD_TYPE:：build 类型(Debug, Release, ...)，
	CMAKE_BUILD_TYPE=Debug	BUILD_SHARED_LIBS：Switch between shared and static libraries内置变量的使用：# 命令(1)

	project (HELLO)   #指定项目名称，生成的VC项目的名称；		>>使用${HELLO_SOURCE_DIR}表示项目根目录	include_directories：指定头文件的搜索路径，相当于指定gcc的-I参数		>> include_directories (${HELLO_SOURCE_DIR}/Hello)  #增加Hello为include目录	link_directories：动态链接库或静态链接库的搜索路径，相当于gcc的-L参数       >> link_directories (${HELLO_BINARY_DIR}/Hello)     #增加Hello为link目录	add_subdirectory：包含子目录       >> add_subdirectory (Hello)	add_executable：编译可执行程序，指定编译，好像也可以添加.o文件       >> add_executable (helloDemo demo.cxx demo_b.cxx)   #将cxx编译成可执行文件——# 命令(2)	add_definitions：添加编译参数	>> add_definitions(-DDEBUG)将在gcc命令行添加DEBUG宏定义；	>> add_definitions( “-Wall -ansi –pedantic –g”)	target_link_libraries：添加链接库,相同于指定-l参数	>> target_link_libraries(demo Hello) #将可执行文件与Hello连接成最终文件demo	add_library:	>> add_library(Hello hello.cxx)  #将hello.cxx编译成静态库如libHello.a	add_custom_target:	message( status|fatal_error, “message”):	
	#在指定目录下搜索一个库, 保存在变量MY_LIB中	find_ibrary(MY_LIB libmylib.a ./)	
	set_target_properties( ... ): lots of properties... OUTPUT_NAME, VERSION, ....	link_libraries( lib1 lib2 ...): All targets link with the same set of libs


最后：相关目录解构可以参考：
[目录结构参考](https://github.com/sheltonliu/Unix_Code_Project/tree/master/cmake/cmake)

关于cmake更详细的内容可以参见[cmake实战](https://github.com/sheltonliu/Unix_Code_Project/tree/master/cmake)



